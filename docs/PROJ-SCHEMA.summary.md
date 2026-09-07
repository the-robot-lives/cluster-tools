# Project Schema — Summary

**No persistence layer** — no DB/SQL schema; dashboards are read-only over live
k8s. Artifacts covered: `infra-config.yaml` sections consumed (`telemetry:` block
with 6 keys: environment, host_type, otelcol_version, resource_detectors,
memory/spike limits), generated-on-target-host OTel Collector + Fluent Bit
configs, and env-var contracts (`K8_CONFIG`, `K8_LIB_DIR`, `K8_TELEMETRY_*`,
`PG_MONITOR_*` / `MYSQL_MONITOR_*`, `FORCE_REINSTALL`).
Full reference: [PROJ-SCHEMA.md](PROJ-SCHEMA.md).

```
config precedence:  --config flag → env var → infra-config.yaml → built-in default
writes:             only cluster-setup-telemetry, on target host (root/sudo)
state files:        none
```
