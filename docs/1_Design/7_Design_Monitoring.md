# Monitoring Design

Use metrics for trends/thresholds, logs for event context, dashboards for correlation, and alerts for actionable interruptions. Add tracing when distributed-application diagnosis needs it, not just to complete a tool collection. See [tool options](./2_Design_Software_Prerequisites.md#observability-storage-and-backup-tools).

## Placement options

| Placement | Benefits | Drawbacks | Choose when |
|---|---|---|---|
| Inside the workload cluster | One deployment model, Kubernetes discovery and shared capacity | Cluster failure also removes monitoring | Learning/small workloads can accept temporary blindness |
| External VM or appliance | Visibility can survive a workload-cluster outage | Another OS/service to maintain; host/storage may still be shared | Diagnosing cluster outages matters |
| Hybrid / dedicated observability platform | Central correlation across clusters, independent storage | More components, capacity and operational complexity | Multiple environments or explicit availability requirements |
| Hosted observability | Offloads some storage/operations | Cost, internet and privacy dependency | Those trade-offs are acceptable |

## Scope and safeguards

Monitor infrastructure, node/workload health, resource pressure, storage, backups and selected user-facing checks. Discovery/exporter configuration is still required; it is not automatic merely because Prometheus is installed. Restrict scrape permissions and endpoints.

Namespaces and dashboard permissions do not supply network isolation. Use enforcing policies, RBAC, appropriate ingress authentication and narrowly permitted cross-segment scrapes. Metrics/log labels can reveal addresses, identities and usage patterns; retention, access controls and publication review apply to them too.

Set requests/limits and retention against measured ingestion and available disk. Avoid unbounded labels/full-text indexing without a budget. Alert on conditions with a useful action, group correlated failures, and test delivery/silencing rather than adding many noisy thresholds.

If monitoring lives inside the cluster, consider limited independent host/network notifications or external reachability checks. They do not provide full central visibility, but can help when the main stack is down. Do not make household routing depend on monitoring.

Use [services and operations setup](../2_Setup/Services_Operations.md) to validate ingestion, queries, alert delivery and recovery.
