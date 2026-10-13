# Low-Level Design (LLD): Platform and Services

Choose what runs on the [infrastructure foundation](../2_HLD/README.md). Each topic combines design, relevant tools and broad implementation/verification considerations without reproducing a specific deployment.

---

## Table of Contents

- [Platform](#platform)
- [Workloads and Delivery](#workloads-and-delivery)
- [Visibility and Data](#visibility-and-data)

---

## Platform

[Container orchestration and cluster lifecycle](./Cluster/README.md) covers Kubernetes and alternatives, node lifecycle and cluster networking. This is the workload cluster, **not a hypervisor host cluster**.

---

## Workloads and Delivery

[Services](./Services/README.md) covers workload selection, exposure, GitOps/CD, CI and secret delivery. Tools are choices within those responsibilities, not independent top-level projects in this documentation.

---

## Visibility and Data

[Monitoring](./Monitoring/README.md) covers signals, placement and alerting. [Storage and backups](./Storage_Backups/README.md) covers persistence, availability and recoverability. Their resource/failure requirements may require revisiting the HLD.
