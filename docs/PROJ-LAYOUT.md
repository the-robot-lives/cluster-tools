# Project Layout — cluster-utils

Terminal utility package: Kubernetes cluster inspection dashboards (`cluster-*`)
and a remote-host telemetry installer. Bash scripts use the current `kubectl`
context and optionally read monorepo `infra-config.yaml` via shared **k8-lib**.
Installs to `~/.local/bin` via `make install` (or monorepo
`make install-utilities`). Dual-path:
`Portfolio/Utilities/source/cluster-utils` ↔ `utilities/k8/cluster-utils`.

Plain tree: [PROJ-LAYOUT.summary.md](PROJ-LAYOUT.summary.md).
Arch: [PROJ-ARCH.md](PROJ-ARCH.md). How-to: [PROJ-HOWTO.md](PROJ-HOWTO.md).

```
cluster-utils/
├── bin/                            # Bash tools → ~/.local/bin (install -m 755)
│   ├── cluster-status              #   Tiered pod dashboard; --watch
│   ├── cluster-nodes               #   Node capacity/reservations; --pods
│   ├── cluster-resources           #   Pod usage vs requests (metrics-server)
│   ├── cluster-helm                #   Helm release status + failure highlight
│   ├── cluster-layout              #   Node/pod/PVC/PV markdown (glow)
│   ├── cluster-manticore           #   Manticore: readers, indexes, S3, jobs
│   └── cluster-setup-telemetry     #   OTel Collector + Fluent Bit on VM/EC2
├── docs/
│   ├── PROJ-ARCH.md(+.summary)     #   Architecture + quick reference
│   ├── PROJ-LAYOUT.md(+.summary)   #   This file + tree-only companion
│   ├── PROJ-HOWTO.md(+.summary)    #   Task guides index + companion
│   ├── PROJ-FAQ.md(+.summary)      #   FAQ + companion
│   ├── arch/
│   │   ├── installation.md         #     Install flow / k8-lib dependency
│   │   └── shared-library.md       #     k8-lib modules sourced at runtime
│   └── howto/
│       └── setup-telemetry.md      #     Remote telemetry install walkthrough
├── CHANGELOG.md                    # Package changelog / milestones
├── .gitignore                      # .DS_Store, editor swap, .env, .envrc.local
├── Makefile                        # compile/test no-ops; install → INSTALL_DIR
└── README.md                       # Start here — prereqs, tools table, telemetry
```

## Install mapping (`make install`)

| Source | Install path | Method |
|--------|--------------|--------|
| `bin/cluster-*` (all seven) | `~/.local/bin/<basename>` | copy (`install -m 755`) |

Override destination: `INSTALL_DIR=/other/path make install`. Globs
`bin/cluster-*` only — no completions package.

## Notes

- **No `lib/`** — shared logic is monorepo **k8-lib** at
  `~/.local/share/k8-lib` (`K8_LIB_DIR` override). Dashboards require it;
  `cluster-setup-telemetry` treats it as optional (remote VM fallbacks).
- **Config** (not in this package): `infra-config.yaml` / `.infra-config.yaml`
  at cwd or monorepo root. Tiers, status patterns, `telemetry:` block.
  Every tool accepts `--config <path>` (pre-parsed into `K8_CONFIG` before
  k8-lib source). Missing config → built-in defaults.
- **Prerequisites**: `kubectl`; `helm` for `cluster-helm`; metrics-server for
  `cluster-resources`; optional `glow` for `cluster-layout`.
- **`cluster-setup-telemetry`**: run on target host with root/sudo (not typical
  dev machine). Values overridable via `K8_TELEMETRY_*` and service monitor
  env vars — see README / [howto/setup-telemetry.md](howto/setup-telemetry.md).
- **Makefile**: `compile` / `test` are no-ops; only `install` does work.

## Key Files Requiring Setup

| File / artifact | Action |
|-----------------|--------|
| `infra-config.yaml` (optional) | Tier groupings, status patterns, `telemetry:` for setup-telemetry |
| `~/.local/share/k8-lib/` | Install via monorepo `make install-utilities` (required for dashboards) |
| `kubectl` context | Point at target cluster before running dashboards |
| `K8_TELEMETRY_*` / monitor creds | Optional overrides when installing telemetry on a host |
