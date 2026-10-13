# Monitoring and Observability

Decide which failures and questions matter before choosing dashboards or collecting every signal.

---

## Table of Contents

- [Design Signals and Placement](#design-signals-and-placement)
- [Tool Choices](#tool-choices)
- [Implementation and Verification](#implementation-and-verification)

---

## Design Signals and Placement

Metrics support trends/thresholds; logs explain events; dashboards correlate; alerts request action. Add tracing when distributed-application diagnosis needs it.

| Placement | Benefits | Drawbacks / fit |
|---|---|---|
| Inside workload cluster | Shared deployment/resources and cluster discovery | Cluster failure removes monitoring; acceptable for outage-tolerant learning |
| External VM/appliance | Can observe workload-cluster failure | Another lifecycle; may still share its physical host/storage |
| Hybrid / dedicated platform | Independent visibility across environments | More components and capacity; justified by scale/availability |
| Hosted service | Offloads some operations/storage | Cost, internet and privacy dependency |

---

## Tool Choices

Prometheus suits Kubernetes-aware metrics/discovery; InfluxDB and infrastructure-oriented Zabbix/Checkmk/Nagios fit other models. Loki provides label-indexed logs; Elasticsearch/OpenSearch offer full-text search with different resource costs. Grafana can correlate sources; Alertmanager or Grafana Alerting provide different alert-rule/routing workflows.

Vendor/local diagnostics remain useful when central monitoring is unavailable. Tool selection does not configure discovery/exporters or enforce access automatically.

---

## Implementation and Verification

Set bounded resource/retention budgets, narrow scrape permissions and authenticated dashboards. Avoid unbounded labels and collection of sensitive identifiers. Configure ingestion/discovery, test queries and controlled alert delivery/grouping/silences.

Monitor storage/backup health and consider whole-system power measurements for [electricity planning](../../2_HLD/Hosts/README.md#electricity-and-consolidation). If the main stack shares the workload cluster, decide whether limited independent checks are useful. Record actual targets/output privately, not in the public reference.
