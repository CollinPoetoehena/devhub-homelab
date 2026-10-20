# Observability and Monitoring

> **Part 6 · Operations** — [Part overview](./README.md) · Next: [Backups](./backups.md)

Decide which failures and questions matter before choosing dashboards or collecting every signal.

Observability is how the homelab provides *ongoing evidence* that it works—the operational form of the principle [evidence over assumption](../01-foundations/design-principles.md#evidence-over-assumption). This chapter covers both observability (the ability to understand the system from its metrics, logs and traces) and monitoring (the checks and alerts that tell you when to act). They are kept in one chapter on purpose: monitoring is one use of the same signals, and splitting them would describe the same collectors, placement and failure domains twice.

---

## Table of Contents

- [Design Signals and Placement](#design-signals-and-placement)
- [Tool Choices](#tool-choices)
- [Implementation and Verification](#implementation-and-verification)

---

## Design Signals and Placement

Metrics support trends/thresholds; logs explain events; dashboards correlate; alerts request action. Add tracing when distributed-application diagnosis needs it.

| Option | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Inside workload cluster | Monitoring deployed with the workloads it observes | Shared deployment/resources and cluster discovery | Cluster failure removes visibility; budget shared resources. **Maintenance:** Shares cluster upgrades and resource pressure; maintain collectors/rules and accept that a full cluster outage also removes diagnostics. | Outage-tolerant learning with accepted blindness |
| External VM/appliance | Monitoring outside the workload cluster | Can observe cluster failures and support separate diagnostics | Another lifecycle; may still share host/storage/power. **Maintenance:** Another OS/platform to patch and back up; test reachability and confirm it can observe the failure domains it is meant to cover. | Cluster-outage visibility matters |
| Hybrid / dedicated platform | Split internal detail and independent critical checks | Visibility across environments/failure domains | More components, capacity and signal ownership. **Maintenance:** Coordinate multiple pipelines and rule ownership, credentials and capacity; test independent checks when the main stack is unavailable. | Scale/availability justify independent observation |
| Hosted service | Provider operates parts of monitoring/storage | Offloads some operations and may check public reachability | Cost, internet/privacy dependencies; private paths may remain invisible. **Maintenance:** Maintain agents/API keys, billing and retention settings; review privacy, provider changes and fallback diagnostics during connectivity loss. | External observation or reduced local operations fits the requirements |

Choose signals from questions: is the household connection working, is a host under memory pressure, did a backup finish and can a service serve requests? A dashboard full of numbers does not answer those automatically. Start with a small set of actionable checks and add detail when an investigation needs it.

Apply the boundary that [notifications are not automatic interruptions](../01-foundations/goals-and-scope.md#notifications-are-not-automatic-interruptions). Classify alerts by impact and required response: routine experiment failures can wait for a lab session; credible security, data-loss or essential-household incidents need a deliberately chosen response path. Group duplicate symptoms, silence expected maintenance downtime and keep nonurgent notifications out of work and rest. Test routing and silence expiry as well as delivery; reducing noise must not hide a genuinely urgent condition.

An external VM is only independent of Kubernetes, not necessarily of the host, power or storage running it. Hosted checks may observe public reachability but cannot always diagnose private paths. Match placement to the failure you need to see, and accept blind spots explicitly when the lab is allowed to go dark.

---

## Tool Choices

Compare tools within their signal responsibility; a dashboard, collector and alert router are not substitutes for each other.

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| [Prometheus](https://prometheus.io/docs/introduction/overview/) | Time-series metrics collection/querying with exporters and discovery | Kubernetes-aware integrations, PromQL and rule evaluation | Scrape/exporter setup, retention and label cardinality; unique IDs/URLs can inflate memory/storage. **Maintenance:** update exporters/rules with target changes, watch disk/cardinality growth and test missing-data alerts | Cloud-native metrics and quantitative trends/thresholds |
| [InfluxDB](https://docs.influxdata.com/) | Time-series database/ecosystem | Another metrics/time-series storage/query model | Version-specific ingestion/query/integration and lifecycle choices. **Maintenance:** Monitor ingestion, retention and disk growth; test supported database/query upgrades and backup restoration for the chosen version. | Its integrations and operational familiarity fit the data |
| [Zabbix](https://www.zabbix.com/documentation/current/en/manual) | Infrastructure/service monitoring platform | Broad host/device checking and alert workflows | Agent/check/discovery configuration and platform maintenance. **Maintenance:** Update server/proxies/agents and templates; check discovery, database growth and alert delivery after target changes. | Infrastructure-oriented monitoring |
| [Checkmk](https://docs.checkmk.com/latest/en/) | Host/service monitoring with discovery and integrations | Structured infrastructure checks | Edition/integration choices and another platform lifecycle. **Maintenance:** Keep sites, agents and integrations compatible; review discovered services and failed checks after host/platform changes. | Host/device/service-check requirements |
| [Nagios Core](https://www.nagios.org/documentation/) | Plugin-oriented availability/service checking | Explicit checks and established plugin approach | Plugin/configuration upkeep and integration effort. **Maintenance:** Maintain plugins and check definitions, tune stale/noisy checks and verify notification paths rather than relying on dashboard status alone. | Bounded service checks with suitable expertise |
| [Loki](https://grafana.com/docs/loki/latest/) | Log system indexing labels and searching selected streams | Correlates label-based logs with metrics without full-content indexing | Label/volume/retention budgets; less full-text indexing; logs can contain sensitive data. **Maintenance:** review collector compatibility, rejected/missing streams, storage retention and access to sensitive logs | Stream-oriented logs and Grafana correlation |
| [Elasticsearch](https://www.elastic.co/docs) | Indexed search/analytics engine used for logs | Rich indexed content search | Resource, retention, licensing/edition and cluster lifecycle choices. **Maintenance:** Manage index lifecycle, capacity, cluster health and supported upgrades; test snapshots and access controls before incidents. | Full-text/search requirements justify the cost |
| [OpenSearch](https://docs.opensearch.org/latest/) | Search/analytics platform used for indexed logs | Rich search and analytics ecosystem | Resource/cluster/index lifecycle and access protection. **Maintenance:** Coordinate node/plugin versions, index retention and capacity; monitor cluster health and validate snapshot recovery. | Indexed-search needs with suitable ecosystem fit |
| [Grafana](https://grafana.com/docs/grafana/latest/) | Dashboards/query interface over data sources | Correlates metrics/logs and reusable dashboards | Does not collect automatically; source permissions and dashboard lifecycle. **Maintenance:** update plugins/queries with source changes, protect access and keep dashboards/rules reproducible rather than UI-only | Unified visualization across chosen sources |
| [Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/) | Notification grouping, routing, inhibition and silencing for alerts | Separates Prometheus rule evaluation from usable delivery | Notification configuration/testing; duplicate rule ownership can page twice. **Maintenance:** Maintain routes/contact credentials, grouping and inhibition rules; periodically test delivery, silences and recovery notifications. | Prometheus rules with controlled notification behavior |
| [Grafana Alerting](https://grafana.com/docs/grafana/latest/alerting/) | Grafana-managed alert evaluation and notification workflows | Alternative rule workflow across supported sources | Source/evaluation compatibility and overlap with other alert owners. **Maintenance:** Review data-source credentials, evaluation errors and rule ownership; test routing/silences after source or plugin upgrades. | Alert ownership is deliberately centered on Grafana |
| [OpenTelemetry](https://opentelemetry.io/docs/) pipeline | Instrumentation and collection/export of supported telemetry | Request-path tracing and vendor-neutral integration | Requires a compatible storage/query backend, sampling and additional components/data. **Maintenance:** Maintain instrumentation/collector/backend compatibility and sampling budgets; detect dropped data and reassess privacy and cardinality. | Distributed diagnosis warrants tracing beyond metrics/logs |
| Vendor/local diagnostics | Device/platform-provided health and troubleshooting interfaces | Available when central monitoring is unavailable | Fragmented scope and no automatic unified alerts. **Maintenance:** Keep recovery access and platform familiarity current; review vendor updates and confirm useful diagnostics remain available without central monitoring. | Recovery fallback and bounded appliance visibility |

Vendor/local diagnostics remain useful when central monitoring is unavailable. Tool selection does not configure discovery/exporters or enforce access automatically.

Choose integrations and operational familiarity rather than expecting one system to cover every signal. Log volume/retention can outweigh metrics storage; collect only justified sources and protect access. Declare rule ownership so the same event does not page twice, and add tracing only when request-path diagnosis justifies it.

---

## Implementation and Verification

Set bounded resource/retention budgets, narrow scrape permissions and authenticated dashboards. Avoid unbounded labels and collection of sensitive identifiers. Configure ingestion/discovery, test queries and controlled alert delivery/grouping/silences.

Monitor storage/backup health and consider whole-system power measurements for [electricity planning](../02-architecture/compute.md#electricity-and-consolidation). If the main stack shares the workload cluster, decide whether limited independent checks are useful. Record actual targets/output privately, not in the public reference.

Alerts should name an impact and a next action, with a sensible delay to avoid transient noise. Test delivery through a controlled failure, then recovery and silence handling. Separate "the monitoring system has no data" from "the service is healthy": missing exporters or a failed pipeline can otherwise produce false reassurance.

Keep dashboards/rules reproducible where supported, with protected inputs and versioned configuration. Budget CPU, storage and retention alongside workloads. Monitor the monitoring pipeline itself and document local diagnostics available during its outage; long retention and every possible exporter are not prerequisites for a useful lab.
