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

Choose signals from questions: is the household connection working, is a host under memory pressure, did a backup finish and can a service serve requests? A dashboard full of numbers does not answer those automatically. Start with a small set of actionable checks and add detail when an investigation needs it.

An external VM is only independent of Kubernetes, not necessarily of the host, power or storage running it. Hosted checks may observe public reachability but cannot always diagnose private paths. Match placement to the failure you need to see, and accept blind spots explicitly when the lab is allowed to go dark.

---

## Tool Choices

Prometheus suits Kubernetes-aware metrics/discovery; InfluxDB and infrastructure-oriented Zabbix/Checkmk/Nagios fit other models. Loki provides label-indexed logs; Elasticsearch/OpenSearch offer full-text search with different resource costs. Grafana can correlate sources; Alertmanager or Grafana Alerting provide different alert-rule/routing workflows.

Vendor/local diagnostics remain useful when central monitoring is unavailable. Tool selection does not configure discovery/exporters or enforce access automatically.

**Prometheus** collects time-series metrics and commonly integrates with exporters and Kubernetes service discovery. Plan scrape scope and label cardinality; arbitrary unique IDs or request URLs can cause large storage/memory growth. **InfluxDB** offers a different time-series ecosystem; **Zabbix/Checkmk/Nagios** suit more infrastructure/service-check-oriented workflows. Choose integrations and operational familiarity rather than expecting one system to cover every signal.

**Loki** indexes labels and searches log content within selected streams; **Elasticsearch/OpenSearch** support richer indexed search at a different resource/lifecycle cost. Log volume and retention can outweigh metrics storage, and logs can contain credentials or personal identifiers. Collect only justified sources and protect access.

**Grafana** presents data from sources rather than creating collection automatically. Prometheus rules plus Alertmanager separate rule evaluation from notification grouping/routing; Grafana Alerting offers another evaluation workflow. Choose ownership so the same event does not page twice. Add tracing, for example through an OpenTelemetry pipeline and compatible backend, only when request-path diagnosis justifies the extra data and components.

---

## Implementation and Verification

Set bounded resource/retention budgets, narrow scrape permissions and authenticated dashboards. Avoid unbounded labels and collection of sensitive identifiers. Configure ingestion/discovery, test queries and controlled alert delivery/grouping/silences.

Monitor storage/backup health and consider whole-system power measurements for [electricity planning](../../2_HLD/Hosts/README.md#electricity-and-consolidation). If the main stack shares the workload cluster, decide whether limited independent checks are useful. Record actual targets/output privately, not in the public reference.

Alerts should name an impact and a next action, with a sensible delay to avoid transient noise. Test delivery through a controlled failure, then recovery and silence handling. Separate "the monitoring system has no data" from "the service is healthy": missing exporters or a failed pipeline can otherwise produce false reassurance.

Keep dashboards/rules reproducible where supported, with protected inputs and versioned configuration. Budget CPU, storage and retention alongside workloads. Monitor the monitoring pipeline itself and document local diagnostics available during its outage; long retention and every possible exporter are not prerequisites for a useful lab.
