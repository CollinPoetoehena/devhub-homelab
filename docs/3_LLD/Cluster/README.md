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

| Platform | Strength | Cost / best fit |
|---|---|---|
| Compose on one host | Explicit service relationships with little platform overhead | Host lifecycle and failover are yours; good for a few bounded services |
| Kubernetes | Declarative APIs, scheduling, controllers and a large integration ecosystem | More control-plane, networking and upgrade work; choose for ecosystem/learning needs |
| Nomad | Scheduler model spanning several workload types | Evaluate its networking, service-discovery and secret integrations as a complete system |
| Swarm | Relatively small multi-host container model | Different ecosystem/features from Kubernetes; verify required integrations and support |

Kubernetes can be worthwhile for learning even when the workloads would fit Compose, but distinguish that learning goal from a claim that it is operationally necessary. Separate control-plane consensus from worker capacity. HA control planes need independent failure placement and recovery planning; scaling workers does not repair lost control-plane quorum.

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

**Talos** removes general-purpose SSH/package administration and exposes a declarative node API. This can reduce drift for dedicated Kubernetes nodes, but requires learning its diagnostics, machine configuration, image/extensions and recovery model. It is not a general utility VM, and immutable does not mean maintenance-free.

**K3s/k0s** package much installation work for a smaller starting footprint. Review bundled components, datastore/HA modes, networking and update paths rather than assuming the default combination fits every lab. The underlying conventional OS still needs patching and access controls.

**kubeadm** exposes upstream bootstrap responsibilities directly: OS, runtime, networking and upgrades are deliberately yours. **Kubespray** automates a conventional-host lifecycle using inventories/playbooks, useful when repeatable host-based deployment is part of the goal. Their flexibility adds integration ownership compared with a purpose-built node OS.

**MicroK8s/RKE2** provide other packaged workflows; **OpenShift/OKD** broaden the platform with operator/application-management concepts and corresponding resource requirements. Managed cloud Kubernetes offloads parts of the platform but adds provider/network/cost constraints. Select by the lifecycle you want to operate, supported hardware and integrations, not merely installation convenience.

---

## Networking and Persistence

Choose pod/service/node ranges without household/VPN overlap. Cilium provides eBPF/policy/optional flow visibility; Calico offers mature networking/policy modes; Flannel supplies connectivity but does not itself enforce NetworkPolicy. Use a compatible enforcing solution if isolation is required.

Namespaces alone do not isolate traffic. Decide policy, cluster DNS, ingress, external address allocation and CSI/persistence separately. Cluster and gateway policies cover different paths. Application/data recovery is part of [storage and backups](../Storage_Backups/README.md).

A CNI supplies pod networking and may enforce policy; a service implementation provides service reachability; ingress/gateway components route application traffic; an external address allocator solves yet another problem. Choose supported combinations before enabling advanced BGP, proxy replacement or overlays. Each adds prerequisites and new failure paths.

Cilium can combine policy and eBPF-based visibility/features; Calico supports several routing/policy approaches. Their operational differences matter more than a feature-count contest. Flannel can be a simpler connectivity choice when paired with an appropriate policy solution if isolation is required. Verify enforcement with real allowed/denied flows, not just successful pod startup.

Storage classes/CSI expose volume lifecycle to workloads, but the backend determines durability and failure handling. Plan what happens when a node disappears, whether volumes can attach elsewhere and how application-consistent backups are restored. etcd recovery and application-data recovery are separate jobs.

---

## Implementation and Verification

After the design, pin compatible versions, prepare node images/configuration, bootstrap through the chosen supported workflow, and validate node readiness, DNS, service paths and actual policy enforcement. Do not enable advanced routing or proxy replacement just because available.

Declare VM provisioning, node lifecycle and [GitOps workload delivery](../Services/README.md#gitops-and-continuous-delivery) ownership explicitly. Test staged upgrades and recovery; declarative manifests do not restore application data. Exact commands, node inputs and execution results belong in private runbooks or official product guides.

Validate node health, DNS, pod-to-pod/service connectivity, external access, policy, volume attachment and controlled replacement separately. Drain/reboot experiments should account for disruption budgets, spare capacity and workloads that cannot move. A successful drain is not evidence of physical-host resilience.

Keep a compatible version matrix and review release notes for the OS, Kubernetes, CNI and storage controllers. Test recovery credentials and supported control-plane backup procedures outside ordinary Git. The public guide explains the lifecycle; it should not prescribe the owner's exact node count, addresses or bootstrap commands.
