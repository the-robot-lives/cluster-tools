# Project Schema — cluster-utils

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
| `/etc/otelcol/config.yaml` | generated collector config | `cluster-setup-telemetry` | target VM/EC2 host |
| `/etc/fluent-bit/fluent-bit.conf` | generated log-agent config | `cluster-setup-telemetry` | target VM/EC2 host |
| systemd units (otelcol, fluent-bit) | generated service defs | `cluster-setup-telemetry` | target VM/EC2 host |

No state files, caches, or databases are written by this package. The only
write side-effect in the entire suite is `cluster-setup-telemetry` generating
OTel Collector + Fluent Bit configs on the **target host** (requires root/sudo).

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
**status patterns**, **namespaces**, **Manticore S3 bucket/labels** — shapes
owned by k8-lib's `config-resolver.sh`, not this package.

### Service monitor keys (telemetry installer receivers)

`PG_MONITOR_HOST` (default `localhost`), `PG_MONITOR_PORT` (5432),
`PG_MONITOR_USER` (default `otel_monitor`), `PG_MONITOR_PASSWORD`;
`MYSQL_MONITOR_*` mirrored (port 3306). Passwords are never defaulted — the
installer skips the receiver if unset.

## Environment-variable contract

| Variable | Applies to | Purpose |
|----------|-----------|---------|
| `K8_CONFIG` | all tools | config file path (`--config` flag pre-parses into this) |
| `K8_LIB_DIR` | all dashboards | k8-lib location (default `~/.local/share/k8-lib`) |
| `K8_TELEMETRY_ENVIRONMENT`, `_HOST_TYPE`, `_RESOURCE_DETECTORS`, `_OTELCOL_VERSION`, `_OTELCOL_MEMORY_LIMIT_MIB`, `_OTELCOL_SPIKE_LIMIT_MIB` | setup-telemetry | override any `telemetry:` key (env > config > default) |
| `PG_MONITOR_*` / `MYSQL_MONITOR_*` | setup-telemetry | DB receiver credentials (see table above) |
| `FORCE_REINSTALL` | setup-telemetry | `=1` overwrites existing collector/fluent-bit install |

Precedence: CLI flag → env var → config file → built-in default.

## Cluster-sourced data (read-only, not persisted)

Dashboards derive everything at runtime from: pod/node/helm/Manticore state via
`kubectl`/`helm` JSON queries, `kubectl top` metrics (metrics-server), and PVC/PV
manifests (`cluster-layout`). Nothing from the cluster is cached or stored.
