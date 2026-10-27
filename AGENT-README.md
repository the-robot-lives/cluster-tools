# AGENT-README

> Orientation for AI coding agents working in this repo. Human docs: README.md.

## Type

scripts

## Purpose

Kubernetes cluster dashboards and inspection utilities (`cluster-*`): tiered pod health, node capacity, resource usage vs requests, Helm release status, storage layout, and remote telemetry bootstrap, from a single install.

## Stack

Bash CLIs in `bin/` sharing `k8-lib`; reads `infra-config.yaml` for Noizu deploy-tier groupings and status patterns; Makefile-driven install.

## Dev mode

```bash
make install    # installs cluster-* tools to ~/.local/bin
make test       # placeholder (no-op)
cluster-status --watch
```

## Ports / URLs

- none (library/CLI)

## Tests

```bash
make test      # placeholder (no-op) — thin Bash wrappers around kubectl/helm
```

## Gotchas

- Uses the current `kubectl` context — aim it at the right cluster first; every tool accepts `--config <path>`.
- `cluster-resources` requires metrics-server in the cluster; `cluster-layout` renders via `glow` (optional).
- `cluster-setup-telemetry` needs root and runs on the target VM/EC2 instance, not the dev machine; monitoring creds via `PG_MONITOR_*` / `MYSQL_MONITOR_*`, overrides via `K8_TELEMETRY_*`, `FORCE_REINSTALL=1` to overwrite.
- Coupled to the trl-infra `terraform/` + `.infra-config.yaml`; shares `k8-lib`.
