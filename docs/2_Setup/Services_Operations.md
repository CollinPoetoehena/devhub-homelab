# Services and Operations

## Delivery and workloads

Choose [delivery/secrets tools](../1_Design/2_Design_Software_Prerequisites.md#delivery-and-secrets) and explicitly wire secret delivery before deploying workloads. Argo CD and Flux have different integrations; selecting encrypted files does not make them consumable automatically.

Keep application code in its own project when it has an independent lifecycle. The private homelab consumes pinned artifacts and records deployment values. The public homelab describes generic patterns, not actual Helm values, inventory or rendered manifests.

Define resource requests/limits, health checks, persistent-data ownership, least-privilege RBAC and enforcing network policies. Decide private ingress/VPN versus deliberate internet exposure with authentication/TLS and narrow permissions. A Service load balancer alone is neither ingress nor access control.

CI validates/builds; CD deploys/reconciles. Restrict runner trust and credentials, avoid untrusted jobs reaching internal services, and add explicit approvals for disruptive operations. Link to [DevHub workflows](https://github.com/CollinPoetoehena/DevHub/tree/main/.github/workflows) rather than cloning generic workflow guidance.

## Monitoring

Implement the chosen [placement](../1_Design/7_Design_Monitoring.md). Configure exporters/discovery, narrow scrape access, authenticated dashboards and bounded retention. Verify metrics/log ingestion and query results; send a controlled alert and confirm delivery, grouping and silence behavior.

Retain a useful independent diagnostic path when central monitoring is unavailable. Dashboard success does not prove firewall isolation, backup integrity or household independence.

## Backup and recovery

Apply the [storage/backup design](../1_Design/8_Design_Storage_Backups.md): configure consistent backups, encryption, retention, protected keys and independent/off-site copies where required. Include device configuration separately from VM/application data.

Restore into an isolated target and check the application/data, not only the archive. Record actual backup/restore times, failures, versions and required identities privately. Ensure the recovery instructions and keys remain accessible if the lab or primary device is offline.

## Experiments

Use disposable VMs/namespaces or genuinely isolated networks. Do not introduce competing DHCP/RA services onto household segments or bypass segmentation to make an experiment work. Record the hypothesis, result and cleanup privately; publish only the reusable, independently generalized lesson.
