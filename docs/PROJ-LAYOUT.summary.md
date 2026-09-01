# Project Layout — Summary

K8s cluster inspection + telemetry install utilities (`cluster-*`).
Full annotated tree: [PROJ-LAYOUT.md](PROJ-LAYOUT.md).

```
cluster-utils/
├── bin/                        # status · nodes · resources · helm · layout · manticore · setup-telemetry
├── docs/
│   ├── PROJ-ARCH.md · PROJ-ARCH.summary.md
│   ├── PROJ-LAYOUT.md · PROJ-LAYOUT.summary.md
│   ├── PROJ-HOWTO.md · PROJ-HOWTO.summary.md
│   ├── PROJ-FAQ.md · PROJ-FAQ.summary.md
│   ├── arch/                   # installation, shared-library
│   └── howto/                  # setup-telemetry
├── CHANGELOG.md
├── merge-notes.md                  # branch-sweep audit trail
├── Makefile                    # make install → ~/.local/bin
└── README.md
```
