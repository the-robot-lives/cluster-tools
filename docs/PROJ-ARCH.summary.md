# cluster-utils — Architecture Summary

Pure-Bash package of seven `cluster-*` tools: six read-only Kubernetes dashboards
against the current kubectl context, plus `cluster-setup-telemetry` (root installer on
VM/EC2 for signoz-otel-collector + Fluent Bit). No in-repo lib — shared config/colors/
`--assist` come from monorepo **k8-lib** at `~/.local/share/k8-lib` (`K8_LIB_DIR`).
Config: `infra-config.yaml` via k8-lib (CLI `--config` pre-parsed before source).
Install: `make install` → `~/.local/bin` (or monorepo `make install-utilities`).

## Components

| Tool | Purpose |
|------|---------|
| `cluster-status` | Category sections by pod-name patterns; `--watch` (5s) |
| `cluster-nodes` | Node capacity/manager/CPU·MEM reservations; `--pods` |
| `cluster-resources` | Usage vs requests (`kubectl top`); needs metrics-server; `-n` |
| `cluster-helm` | Helm release status + DB pods/PVCs; needs helm + python3; `-n` |
| `cluster-layout` | Markdown node/pod/PVC/PV layout; assist optional; glow optional |
| `cluster-manticore` | Readers, index versions/docs, S3, indexer jobs (`aws` + exec) |
| `cluster-setup-telemetry` | Remote OTel+Fluent Bit install; k8-lib optional; `K8_TELEMETRY_*` |
| `Makefile` | `install` only; compile/test no-ops |

## Key Decisions

- Standalone Bash; k8-lib by absolute path (path-independent installs)
- `--config` pre-parsed before library init
- Status sections = name regexes (`K8_STATUS_*_PATTERN`), not deploy tiers
- Layout skips config/common; telemetry degrades without k8-lib
- python3 for JSON (helm/layout); Karpenter labels for capacity/pool

## Tech Stack

Bash · kubectl · helm · metrics-server · python3 · aws CLI (manticore) · k8-lib ·
signoz-otel-collector + Fluent Bit (telemetry) · optional glow / assist AI
