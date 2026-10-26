# Project Schema — cluster-utils

<!-- 〚📁:fecaf08c-c679-5cfd-8f25-a46dace9e656〛 docs/PROJ-SCHEMA.md :: Data-artifact schema: no persistence layer; infra-config.yaml telemetry block, env-var contract, host-generated configs. -->

> **No persistence layer.** This package has **no database, no SQL schema, and no
> Liquibase changelogs**. All `cluster-*` dashboards are read-only views over a
> live Kubernetes cluster (via `kubectl`/`helm`); nothing is written to disk on
> the dev machine. This document instead covers the **configuration and data
> artifacts** the tools read and, in one case, generate.

Plain tree: [PROJ-LAYOUT.md](PROJ-LAYOUT.md). Arch: [PROJ-ARCH.md](PROJ-ARCH.md).

## Artifact inventory

| Artifact | Kind | Created by | Location |
|----------|------|-----------|----------|
| `infra-config.yaml` / `.infra-config.yaml` | YAML config (read-only input) | monorepo / user | cwd, monorepo root, or `--config` path |
| `~/.local/share/k8-lib/bin/config.sh` | runtime config resolver (read-only) | monorepo `make install-utilities` | installed lib dir (`K8_LIB_DIR` override) |
| `.meta/pointers.yaml` | YAML doc-pointer store (committed) | `doc-pointers build --write` | repo root |
| `/opt/signoz-otel-collector/bin/signoz-otel-collector` | collector binary | `cluster-setup-telemetry` | target VM/EC2 host |
| `/etc/signoz-otel-collector/config.yaml` | generated collector config | `cluster-setup-telemetry` | target VM/EC2 host |
| `/etc/fluent-bit/fluent-bit.conf` + `parsers.conf` | generated log-agent config | `cluster-setup-telemetry` | target VM/EC2 host |
| `/var/lib/fluent-bit/` | buffer + per-input position DBs (`syslog.db`, `auth.db`, `journal.db`, …) | Fluent Bit at runtime | target VM/EC2 host |
| `signoz-otel-collector.service`, `fluent-bit.service` | systemd units (package unit, else `/etc/systemd/system/signoz-otel-collector.service`) | `cluster-setup-telemetry` | target VM/EC2 host |

No state files, caches, or databases are written by this package on the dev
machine. The only write side-effect in the entire suite is
`cluster-setup-telemetry` generating configs and services on the **target host**
(requires root/sudo); it backs up any existing `config.yaml` / `fluent-bit.conf`
to `<file>.bak.<epoch>` before overwriting.


## `infra-config.yaml` — sections consumed

Consumed indirectly via k8-lib `config.sh` (dashboards) or `_telem_cfg` YAML
lookups (`cluster-setup-telemetry`). Missing file → built-in defaults.

### `telemetry:` block (cluster-setup-telemetry)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `environment` | string | `production` | `deployment.environment` resource attribute |
| `host_type` | enum: `ec2` \| `vm` \| `bare-metal` | `ec2` | `host.type` resource attribute |
| `otelcol_version` | string | `0.129.12` | signoz-otel-collector release version |
| `resource_detectors` | string | `env, system, ec2` | detector chain |
| `otelcol_memory_limit_mib` | int | `512` | collector memory_limiter limit |
| `otelcol_spike_limit_mib` | int | `128` | collector memory_limiter spike limit |

Other sections consumed by dashboards via k8-lib: **tier groupings**,
**namespaces** — shapes owned by k8-lib's `config-resolver.sh`, not this
package. Status patterns, node-pool labels, and Manticore selectors are read
directly by this package's scripts (see the env table below).

### Service monitor keys (telemetry installer receivers)

`PG_MONITOR_HOST` (default `localhost`), `PG_MONITOR_PORT` (5432),
`PG_MONITOR_USER` (default `otel_monitor`), `PG_MONITOR_PASSWORD`;
`MYSQL_MONITOR_*` mirrored (port 3306). There is no functional password
default: when unset, the literal `TODO_SET_PASSWORD` is written into the
generated config and the run ends with a TODO warning — the receiver block is
still emitted, so the collector starts with a placeholder credential.

## Environment-variable contract

| Variable | Applies to | Purpose |
|----------|-----------|---------|
| `K8_CONFIG` | all tools | config file path (`--config` flag pre-parses into this) |
| `K8_LIB_DIR` | all dashboards | k8-lib location (default `~/.local/share/k8-lib`) |
| `K8_STATUS_DB_PATTERN` / `_SEARCH_` / `_APP_` / `_NET_` / `_INFRA_` / `_JOB_PATTERN` | cluster-status | regex selecting each dashboard section (defaults: DBs, search, apps, networking, infra, jobs) |
| `K8_NODEPOOL_LABELS` | cluster-layout | `raw=label,...` map for node-pool display (default `general=karpenter,indexer=karpenter:indexer,managed=eks`) |
| `K8_NAMESPACE` | cluster-manticore | namespace to inspect (default `default`) |
| `K8_MANTICORE_APP_LABEL`, `K8_MANTICORE_READER_POD`, `K8_MANTICORE_DELTA_CRONJOB`, `K8_MANTICORE_S3_BUCKET` | cluster-manticore | selectors (defaults `manticore`, `manticore-reader-0`, `manticore-delta-indexer`, `manticore-indexes`) |
| `K8_TELEMETRY_ENVIRONMENT`, `_HOST_TYPE`, `_RESOURCE_DETECTORS`, `_OTELCOL_MEMORY_LIMIT_MIB`, `_OTELCOL_SPIKE_LIMIT_MIB` | setup-telemetry | override the matching `telemetry:` keys (env > config > default) |
| `OTELCOL_VERSION` | setup-telemetry | collector release version — **not** a `K8_TELEMETRY_*` key |
| `PG_MONITOR_*` / `MYSQL_MONITOR_*` | setup-telemetry | DB receiver credentials (see table above) |
| `FORCE_REINSTALL` | setup-telemetry | `=1` overwrites existing collector/fluent-bit install non-interactively |

Precedence: CLI flag → env var → config file → built-in default.

## Generated host configs (written shape)

| File | Format | Shape |
|------|--------|-------|
| `/etc/signoz-otel-collector/config.yaml` | OTel Collector YAML | `receivers` (`hostmetrics` + `fluentforward 127.0.0.1:24224` + one per detected service: `postgresql`, `mysql`, `nginx`, `redis`, `docker_stats`), `processors` (`batch`, `memory_limiter`, `resourcedetection`, `resource/host`), `exporters.otlp`, `extensions` (`health_check`, `zpages`), two pipelines (metrics, logs) |
| `/etc/fluent-bit/fluent-bit.conf` | Fluent Bit classic INI | `[SERVICE]` (flush 5, HTTP server `0.0.0.0:2020`, storage `/var/lib/fluent-bit/storage`), tail `[INPUT]` blocks per detected log file, `systemd` input for `ssh`/`cron` units, `grep` + `modify` filters (adds `hostname`, `host.type`, `deployment.environment`), single `forward` output to `127.0.0.1:24224` |
| `/etc/fluent-bit/parsers.conf` | Fluent Bit classic INI | 12 named `[PARSER]` blocks: syslog (rfc3164/plain/iso8601), nginx access/error, postgresql, mysql error/slow, docker json, auth-syslog (+iso8601) |

Values interpolated from `telemetry:` / env are written verbatim into these
files — no validation or escaping (see [THREAT-MODEL.md](THREAT-MODEL.md) T-008).


## Cluster-sourced data (read-only, not persisted)

Dashboards derive everything at runtime from: pod/node/helm/Manticore state via
`kubectl`/`helm` JSON queries, `kubectl top` metrics (metrics-server), and PVC/PV
manifests (`cluster-layout`). Nothing from the cluster is cached or stored.
