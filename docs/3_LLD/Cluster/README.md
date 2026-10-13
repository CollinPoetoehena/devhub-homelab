# Container Orchestration and Cluster Lifecycle

Start with workload/learning needs, not a predetermined Kubernetes installation.

---

## Table of Contents

- [Design Options](#design-options)
- [Kubernetes Node Lifecycle](#kubernetes-node-lifecycle)
- [Networking and Persistence](#networking-and-persistence)
- [Implementation and Verification](#implementation-and-verification)

---

## Design Options

Compose suits a handful of single-host services. Kubernetes adds scheduling, reconciliation and a broad ecosystem with more lifecycle work. Nomad offers another scheduler/workload model; Swarm offers simpler multi-node containers with a different ecosystem. A hypervisor cluster is not an orchestration cluster.

Choose single-node/outage-tolerant operation versus an HA control plane deliberately. More virtual nodes on one physical host teach distributed behavior without physical resilience. See [quorum and failure domains](../../2_HLD/Hosts/README.md#quorum-and-failure-domains).

---

## Kubernetes Node Lifecycle

| Approach | Benefits | Costs / when to choose |
|---|---|---|
| Talos Linux | Immutable API-managed node OS; less general-purpose drift | Different debugging/extensions model; choose dedicated Kubernetes nodes and still own compatibility/recovery |
| K3s / k0s | Convenient lightweight distributions | Conventional host OS still maintained; useful for constrained or simpler installations |
| kubeadm on Linux | Direct upstream bootstrap practice and host control | Own OS/runtime/kubelet integration; choose for that learning/flexibility |
| Kubespray | Ansible-driven deployment/lifecycle | Inventory/playbooks and Linux maintenance remain; choose automated conventional hosts |
| MicroK8s / RKE2 | Other packaged lifecycle/security approaches | Evaluate host/update model and required integrations |
| OpenShift / OKD | Broader operator-managed application platform | More resource/scope/operations; choose explicit platform learning |
| Managed cloud Kubernetes | Provider owns parts of control-plane/lifecycle | Cost/provider dependency; choose cloud learning or offloading local infrastructure |

Talos is a node OS, kubeadm is bootstrap tooling, Kubespray is automation, and K3s is a distribution. Compare the whole system you would operate. Temporary kubeadm experiments can coexist with a lower-maintenance long-lived platform.

---

## Networking and Persistence

Choose pod/service/node ranges without household/VPN overlap. Cilium provides eBPF/policy/optional flow visibility; Calico offers mature networking/policy modes; Flannel supplies connectivity but does not itself enforce NetworkPolicy. Use a compatible enforcing solution if isolation is required.

Namespaces alone do not isolate traffic. Decide policy, cluster DNS, ingress, external address allocation and CSI/persistence separately. Cluster and gateway policies cover different paths. Application/data recovery is part of [storage and backups](../Storage_Backups/README.md).

---

## Implementation and Verification

After the design, pin compatible versions, prepare node images/configuration, bootstrap through the chosen supported workflow, and validate node readiness, DNS, service paths and actual policy enforcement. Do not enable advanced routing or proxy replacement just because available.

Declare VM provisioning, node lifecycle and [GitOps workload delivery](../Services/README.md#gitops-and-continuous-delivery) ownership explicitly. Test staged upgrades and recovery; declarative manifests do not restore application data. Exact commands, node inputs and execution results belong in private runbooks or official product guides.
