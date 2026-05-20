# cluster-tools — Cluster Inspection

Kubernetes cluster dashboards and inspection utilities.

## Installation

```bash
make install    # Installs cluster-* tools to ~/.local/bin
```

## Prerequisites

- `kubectl` with cluster access
- `helm` for release inspection
- `glow` for markdown rendering (optional, used by `cluster-layout`)

## Configuration

Uses current `kubectl` context. Optionally reads `k8-util-config.yaml` for tier groupings and status patterns (see [k8-lib README](../k8-lib/README.md)). Every tool accepts `--config <path>` to specify an alternative config file.

## Tools

| Command | Purpose |
|---------|---------|
| `cluster-status` | Tiered pod dashboard showing health across namespaces |
| `cluster-nodes` | Node layout with CPU/RAM reservations |
| `cluster-resources` | Per-pod CPU/RAM usage vs requests (needs metrics-server) |
| `cluster-helm` | Helm release status, color-coded by status |
| `cluster-layout` | Node/PVC/PV layout as rendered markdown |

## Usage

```bash
cluster-status                  # All pods, grouped by tier
cluster-status --watch          # Auto-refresh
cluster-nodes                   # Node summary
cluster-nodes --pods            # Nodes with pod placement
cluster-resources               # Resource usage (requires metrics-server)
cluster-helm                    # Helm releases with failure highlighting
cluster-layout                  # Full cluster layout (requires glow)
```
