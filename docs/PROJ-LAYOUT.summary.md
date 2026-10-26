# Project Layout — Summary

K8s cluster inspection + telemetry install utilities (`cluster-*`).
Submodule `Projects/cluster-tools`, aliased in the monorepo at
`Portfolio/Utilities/cluster` (tracked symlink).
Full annotated tree: [PROJ-LAYOUT.md](PROJ-LAYOUT.md).

```
cluster-tools/
├── bin/                        # status · nodes · resources · helm · layout · manticore · setup-telemetry
├── docs/
│   ├── PROJ-ARCH.md · PROJ-ARCH.summary.md
│   ├── PROJ-LAYOUT.md · PROJ-LAYOUT.summary.md
│   ├── PROJ-SCHEMA.md · PROJ-SCHEMA.summary.md
│   ├── PROJ-HOWTO.md · PROJ-HOWTO.summary.md
│   ├── PROJ-FAQ.md · PROJ-FAQ.summary.md
│   ├── THREAT-MODEL.md · THREAT-MODEL.summary.md
│   ├── arch/                   # installation, shared-library
│   └── howto/                  # setup-telemetry
├── .meta/pointers.yaml         # doc-pointer store
├── AGENT.md · AGENTS.md · CLAUDE.md
├── CHANGELOG.md
├── merge-notes.md                  # branch-sweep audit trail
├── Makefile                    # make install → ~/.local/bin
└── README.md
```
