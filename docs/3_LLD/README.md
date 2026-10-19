# Low-Level Design (LLD): Platform and Services

Choose what runs on the [infrastructure foundation](../2_HLD/README.md). Each topic combines design, relevant tools and broad implementation/verification considerations without reproducing a specific deployment.

Apply the [repository relationship](../../README.md#how-devhub-and-the-homelab-repositories-fit-together) here too: link shared concepts and general workflows in DevHub, explain their use and trade-offs for a homelab in these categories, and keep actual configuration and results private.

---

## Table of Contents

- [Platform](#platform)
- [Automation](#automation)
- [Workloads and Delivery](#workloads-and-delivery)
- [Service Scheduling](#service-scheduling)
- [Visibility and Data](#visibility-and-data)

---

## Platform

[Container orchestration and cluster lifecycle](./Cluster/README.md) covers Kubernetes and alternatives, node lifecycle and cluster networking. This is the workload cluster, **not a hypervisor host cluster**.

Decide whether workloads need single-host containers, a scheduler or a broader application platform. Then choose node OS/bootstrap, network/policy and persistence integrations as one compatible lifecycle. Kubernetes objects alone do not install a host OS, replace a gateway or make data durable.

---

## Automation

[Automation and configuration management](./Automation.md) covers the low-level choices for provisioning, host/guest configuration, workload delivery boundaries and validation. Automation is a useful cross-cutting principle, but the specific tools, ownership boundaries and workflows belong here alongside the platforms they operate.

---

## Workloads and Delivery

[Services](./Services/README.md) covers workload selection, exposure, GitOps/CD, CI and secret delivery. Tools are choices within those responsibilities, not independent top-level projects in this documentation.

Define how code becomes an artifact, how approved intent reaches the platform and how credentials are delivered. Keep application repositories distinct from infrastructure integration where their release cycles differ. Reconciliation, packaging and infrastructure provisioning may cooperate, but must not fight over the same resource.

---

## Service Scheduling

[Service scheduling](./Services/Scheduling.md) helps decide which components need to be always on, scheduled, or started on demand. Automatic graceful lifecycle management can reduce electricity and avoid risky manual shutdowns; keep the network foundation available when household connectivity depends on it.

---

## Visibility and Data

[Monitoring](./Monitoring/README.md) covers signals, placement and alerting. [Storage and backups](./Storage_Backups/README.md) covers persistence, availability and recoverability. Their resource/failure requirements may require revisiting the HLD.

Decide what remains observable and recoverable when the workload platform is down. A dashboard stored only in that platform cannot report its complete outage; a replicated volume is not an independent backup. Document these accepted limits or add justified independent components.

LLD should produce clear component responsibilities, compatibility constraints, update/recovery workflows and validation criteria. Actual releases, resource allocations, endpoints and results stay in the matching private category; public documentation shows the reasoning and generic implementation approach.
