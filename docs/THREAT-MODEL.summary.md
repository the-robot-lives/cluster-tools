# Threat Model — Summary

Operator tooling, not a service: six read-only `cluster-*` dashboards on the
ambient kubectl identity, plus the root `cluster-setup-telemetry` installer on a
remote VM/EC2 host. Real attack surface is the installer: unverified root code
fetch, world-readable DB credentials in generated config, cleartext OTLP export,
and three unauthenticated listeners. Scope: `noizu` cluster only.
Full register: [THREAT-MODEL.md](THREAT-MODEL.md).

## Trust boundaries

`operator shell → cluster API` · `operator shell → host root (sudo)` ·
`host → package network (unauthenticated fetch)` · `host → central collector
(OTLP, tls insecure)` · `local network → host listeners (:2020 :13133 :55679)` ·
`collector → docker.sock` · `host → S3 / DBs`

## Register counts

8 open · 2 partial · 5 accepted of 15 (T-001…T-015).

| Severity | Open | Partial | Accepted |
|----------|------|---------|----------|
| High | T-001, T-002, T-003, T-004 | — | — |
| Medium | T-005, T-006, T-008 | T-007, T-015 | T-009 |
| Low | T-010 | — | T-011, T-012, T-013, T-014 |

## Close first (cheap, no acceptance)

- T-003 — 0600/root on `/etc/signoz-otel-collector/config.yaml`
- T-005 / T-006 — bind Fluent Bit HTTP and collector extensions to 127.0.0.1
- T-004 — TLS on the OTLP export and DB receiver legs
- T-001 / T-002 — pin the Fluent Bit install script; checksum the collector tarball

## Accepted

Ambient kubeconfig identity (T-009), ambient AWS creds (T-011), no installer
audit trail (T-012), 30-minute connectivity retry (T-013), unquoted trap path
(T-014) — privileged-operator tooling inside one perimeter.
