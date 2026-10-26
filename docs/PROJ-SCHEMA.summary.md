# Project Schema — Summary

**No persistence layer** — no DB/SQL schema; dashboards are read-only over live
k8s. Artifacts covered: `infra-config.yaml` sections consumed (`telemetry:` block
with 6 keys: environment, host_type, otelcol_version, resource_detectors,
memory/spike limits), host-generated OTel Collector + Fluent Bit configs
(`/etc/signoz-otel-collector/config.yaml`, `/etc/fluent-bit/*`, buffer
`/var/lib/fluent-bit`), the committed `.meta/pointers.yaml` doc-pointer store,
and env-var contracts (`K8_CONFIG`, `K8_LIB_DIR`, `K8_STATUS_*`,
`K8_NODEPOOL_LABELS`, `K8_NAMESPACE`, `K8_MANTICORE_*`, `K8_TELEMETRY_*`,
`OTELCOL_VERSION`, `PG_MONITOR_*` / `MYSQL_MONITOR_*`, `FORCE_REINSTALL`).
Full reference: [PROJ-SCHEMA.md](PROJ-SCHEMA.md).

```
config precedence:  --config flag → env var → infra-config.yaml → built-in default
writes:             only cluster-setup-telemetry, on target host (root/sudo)
state files:        none
```
