# Platform: Container Orchestration and Cluster Lifecycle

> **Part 2 · Architecture** — [Part overview](./README.md) · Previous: [Storage](./storage.md) · Next: [Part 3 · Platform Services](../03-platform-services/README.md)

The platform is the layer between the infrastructure and the services: it decides how workloads are packaged, scheduled, networked and given persistent storage. In this book, "platform" means the **workload platform**—Docker Compose, Kubernetes or an alternative orchestrator—and **not** the hypervisor host cluster, which belongs to [compute](./compute.md). A Kubernetes cluster made of VMs consumes compute capacity and inherits its failure domains; it does not replace them.

This is the last chapter of the architecture part because it builds on all the others: nodes run on [compute](./compute.md), node and Pod networking sit inside the [network design](./network.md#networks-inside-the-lab), and persistent volumes depend on the [storage design](./storage.md#design-options). The [platform services](../03-platform-services/README.md) and [services](../05-services/README.md) that follow then run on it.

Start with workload/learning needs, not a predetermined Kubernetes installation.

---

## Table of Contents

- [Design Options](#design-options)
- [Environment Strategy](#environment-strategy)
- [Kubernetes Node Lifecycle](#kubernetes-node-lifecycle)
- [Networking and Persistence](#networking-and-persistence)
- [Implementation and Verification](#implementation-and-verification)
- [Further Reading](#further-reading)

---

## Design Options

A hypervisor cluster is not an orchestration cluster. Compare workload models and lifecycle ownership before choosing a platform.

Choose single-node/outage-tolerant operation versus an HA control plane deliberately. More virtual nodes on one physical host teach distributed behavior without physical resilience. See [quorum and failure domains](./compute.md#quorum-and-failure-domains).

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| [Docker Compose](https://docs.docker.com/compose/) | Declarative multi-container application on a host | Explicit service relationships and little platform overhead | Host lifecycle/failover are yours; no multi-node scheduler. **Maintenance:** comparatively small platform scope, but still update host/runtime/images and test service/data restoration | A few bounded services or local application development |
| [Kubernetes](https://kubernetes.io/docs/home/) | Orchestration platform with APIs, scheduling and reconciliation controllers | Large integration ecosystem, declarative workloads and distributed-platform learning | Control-plane, networking, storage and upgrade work. **Maintenance:** higher integration scope; track supported component versions, staged node upgrades and separate control-plane/application recovery | Ecosystem/learning or scheduling needs justify platform overhead |
| [Nomad](https://developer.hashicorp.com/nomad/docs) | Scheduler spanning several workload types | Alternative scheduling model including non-container tasks | Evaluate service discovery, networking and secret integrations as a complete system. **Maintenance:** Coordinate server/client releases and scheduler state recovery; maintain networking, discovery and secret integrations alongside workloads. | Mixed workload scheduling and its ecosystem fit the goal |
| [Docker Swarm](https://docs.docker.com/engine/swarm/) | Docker-integrated multi-host container orchestration | Relatively small multi-node model | Different feature/integration ecosystem from Kubernetes; verify required support. **Maintenance:** Update managers/workers and container runtime, preserve manager recovery material and test node replacement and application-data restoration. | Simpler multi-host containers without a Kubernetes-specific requirement |
| Single-node / outage-tolerant cluster | Platform runs without a highly available control plane | Lower capacity and operational overhead; useful lifecycle practice | Node/host maintenance can interrupt control-plane access; data still needs recovery. **Maintenance:** Simpler member coordination, but schedule outage windows and keep tested rebuild/control-plane/data recovery outside the failed node. | Learning or services that tolerate outages |
| HA Kubernetes control plane | Supported consensus members across independent failure domains | Can tolerate supported member failures while retaining control-plane quorum | More resources, compatible placement, backups and recovery planning; workers do not vote in etcd. **Maintenance:** Stage member upgrades while preserving quorum; monitor certificates/etcd health and test member loss and recovery across real failure domains. | Availability requirement justifies independent placement and operations |

Kubernetes can be worthwhile for learning even when the workloads would fit Compose, but distinguish that learning goal from a claim that it is operationally necessary. Separate control-plane consensus from worker capacity. HA control planes need independent failure placement and recovery planning; scaling workers does not repair lost control-plane quorum.

---

## Environment Strategy

Environment labels describe the purpose and change policy of workloads, not necessarily separate physical infrastructure. **Dev** is for active changes, **Tst (Test)** for repeatable checks, **Acc (Acceptance)** for review against expected behavior, and **PRD (Production)** for stable services. Decide which distinctions are useful before building four copies of everything.

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| One environment (PRD-only) | A single environment runs stable workloads; experiments use disposable local or temporary resources where possible | Lowest ongoing compute and administration cost; fewest copies of stateful services | No separate pre-release environment, so changes can affect the stable workloads. **Maintenance:** Maintain one platform and its backups, and use careful change windows and tested recovery because there is no separate environment to catch mistakes. | A small homelab, limited hardware, or services where a separate test stack adds little value |
| Separate Dev, Tst, Acc and PRD | Four environments with their own workload configuration and, depending on design, separate clusters or virtual machines | Clear progression and room to validate changes before release; failures can be scoped more deliberately | Duplicates resource use, configuration, credentials, upgrades and data handling; extra environments do not help if they are not kept representative. **Maintenance:** Keep four configurations and their dependencies compatible, patch each, and maintain separate access, data and recovery procedures. | A specific need for staged release practice and enough spare capacity and time to operate every stage |
| One DTA plus one PRD | A shared non-production environment covers Dev/Test/Acc, alongside a separate production workload environment | Practical middle ground: production changes can be checked in DTA without maintaining four full stacks | DTA and PRD can still diverge; shared physical hosts remain a common failure domain. **Maintenance:** Keep DTA close enough to PRD to make tests useful, label/protect data and credentials, and patch, back up and verify both environments. | A homelab that wants release/testing practice while keeping the environment count and resource cost modest |

For a resource-limited homelab, start with **one environment** and make workloads reproducible, backed up and easy to restore. Add a small, preferably on-demand **DTA** and a separate **PRD** workload boundary only when there is enough capacity and a real testing benefit. Give DTA only the resources needed for tests; size PRD for its actual always-on services and recovery needs rather than duplicating every service at full scale. “Production” in a homelab describes the stability expectation, not business-grade availability.

The environments can share the same physical equipment. For example, one hypervisor host can run separate workload VMs, or one hypervisor cluster can host VMs for multiple environments. A separate physical stack or hypervisor cluster per environment is usually unnecessary and expensive for a homelab. Virtualization does not create independent failure domains: environments on one host share its outage, power, storage and maintenance. A single host can be operated standalone; a multi-node hypervisor cluster may have quorum or node-count requirements that should be checked for the chosen product.

Kubernetes is one example of how the environment choices might be implemented, not a prescribed setup. A single small Kubernetes node can combine control-plane and workload roles; this is the simplest footprint but a node outage takes down the cluster. One control-plane node plus one worker separates those roles and is useful for learning, but it is still **not highly available**: loss or maintenance of the sole control plane interrupts cluster management. Separate Kubernetes clusters for DTA and PRD offer a stronger workload boundary but consume more resources and add lifecycle work. Namespaces in one cluster are cheaper and useful for organization, quotas and some policy, but are not equivalent to separate clusters or a strong security boundary: cluster administrators, cluster-scoped resources, node capacity and underlying storage/network remain shared.

The exact environment layout, runtime, node count and resource allocation depend on the specific homelab: available hardware and capacity, workloads, failure tolerance, and the hypervisor, orchestrator, automation and other tools selected. The examples here are starting points for comparison, not requirements; adapt them to the capabilities and operating model of the chosen tools. Keep production-like data and credentials out of experiments; use synthetic or sanitized test data. Verify that the chosen backup and restore process covers each environment and its state, not just its VM or Kubernetes manifests.

---

## Kubernetes Node Lifecycle

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| [Talos Linux](https://docs.siderolabs.com/) | Immutable Kubernetes-specific node OS with declarative API management | Removes general-purpose SSH/package administration; repeatable images/configuration reduce drift and integration work | Learn API diagnostics, extensions and recovery; own compatible OS/Kubernetes upgrades, credentials and backups; not a utility VM or managed service. **Maintenance:** fewer mutable-host tasks, but retain tested machine configuration, supported upgrade paths and accessible recovery credentials | Dedicated Kubernetes nodes where reduced host maintenance matters |
| [Flatcar Container Linux with kubeadm](https://www.flatcar.org/docs/latest/orchestrate/kubernetes/getting-started-with-kubernetes/) | Container-focused Linux OS with an immutable base, provisioned through Ignition (commonly authored with Butane), with Kubernetes bootstrapped separately using kubeadm | Repeatable node provisioning and image-based OS updates reduce package drift; retains familiar Linux/systemd administration and flexibility beyond Kubernetes | Not an integrated Kubernetes lifecycle like Talos; own bootstrap, Kubernetes binaries/system extensions, networking and storage compatibility. **Maintenance:** Coordinate OS update channels and reboot scheduling with node drains and control-plane quorum; separately manage supported Kubernetes upgrades, provisioning configuration, etcd backups and tested node replacement. Automatic OS updates do not automatically make Kubernetes upgrades safe | Immutable container hosts with conventional Linux access, or learning kubeadm while keeping the base OS repeatable; choose when the separate OS and cluster lifecycle responsibilities are acceptable |
| [K3s](https://docs.k3s.io/) | Lightweight packaged Kubernetes distribution | Simplifies installation and bundles common components | Conventional OS patching remains; evaluate bundled networking, datastore/HA and upgrades. **Maintenance:** Update conventional hosts and distribution releases; verify bundled component/datastore compatibility and supported backup/restore before upgrades. | Constrained hardware or simpler supported installations |
| [k0s](https://docs.k0sproject.io/) | Packaged Kubernetes distribution with a compact deployment model | Reduces bootstrap assembly and offers supported lifecycle tooling | Host OS, datastore/HA, networking and version compatibility still need design. **Maintenance:** Maintain host patching, distribution configuration and datastore backups; test supported upgrades and node replacement with chosen integrations. | A lightweight distribution fitting the supported lifecycle/hardware |
| [kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/) on Linux | Upstream Kubernetes bootstrap tooling on conventional hosts | Direct fundamentals and control over host/runtime choices | Own OS, runtime, kubelet, networking and coordinated upgrades. **Maintenance:** higher custom integration responsibility; keep package/runtime/bootstrap automation compatible and test drain, upgrade and rebuild behavior | Bootstrap learning or host flexibility; disposable exercises can suffice |
| [Kubespray](https://kubespray.io/) | Ansible-driven conventional-host deployment/lifecycle | Repeatable inventory/playbook-based cluster operations | Linux, inventories, playbooks and compatibility still need maintenance. **Maintenance:** Keep inventories, Ansible dependencies and supported release combinations aligned; test playbook changes and coordinated host/cluster upgrades. | Automated conventional hosts are part of the learning/operational goal |
| [MicroK8s](https://canonical.com/microk8s/docs) | Packaged Kubernetes with add-on workflows | Convenient local/small-cluster start | Review host/package updates, HA, add-ons and integrations. **Maintenance:** Review supported package/channel updates and add-on compatibility; test cluster/datastore recovery and avoid assuming every update is operationally transparent. | A supported packaged workflow suits development or small deployments |
| [RKE2](https://docs.rke2.io/) | Kubernetes distribution with packaged security/lifecycle choices | Integrated distribution approach with a security-oriented scope | Host/update model, resources and integrations still need validation. **Maintenance:** Coordinate host and distribution releases, configuration and datastore backups; validate networking/security integrations after upgrades. | Its supported operational/security model matches the goal |
| [OpenShift](https://docs.redhat.com/en/documentation/openshift_container_platform) | Broader operator-managed enterprise application platform | Platform-management, operator and enterprise-workflow learning | More resource, release, support and operational scope than a small Kubernetes stack. **Maintenance:** Operate platform operators, supported upgrade paths, certificates and backups; broader platform scope requires more routine health and compatibility review. | Explicit OpenShift/platform learning |
| [OKD](https://docs.okd.io/) | Community Kubernetes application platform related to OpenShift | OpenShift-style platform concepts without an OCP subscription | Free to obtain is not lightweight or maintenance-free; release/support/resource requirements remain. **Maintenance:** Track community release paths and operator compatibility; budget similar platform-health and recovery work despite no OCP subscription. | Community platform learning justifies the broader scope |
| Managed cloud Kubernetes, e.g. [AKS](https://learn.microsoft.com/en-us/azure/aks/) / [EKS](https://docs.aws.amazon.com/eks/) | Provider operates defined control-plane/lifecycle responsibilities | Offloads parts of local infrastructure and teaches cloud integration | Cost, provider/network dependencies and shared responsibility; workloads/data remain yours. **Maintenance:** Review provider upgrade/deprecation windows, node images, permissions and costs; maintain add-ons, workloads and data recovery under shared responsibility. | Cloud learning or deliberate infrastructure offloading |

Talos is a node OS, kubeadm is bootstrap tooling, Kubespray is automation, and K3s is a distribution. Flatcar is a container-focused node OS rather than a Kubernetes distribution: its immutable base does not replace the bootstrap and cluster lifecycle tooling. Unlike Talos's Kubernetes-specific API-managed approach, Flatcar retains conventional Linux administration while you assemble and maintain the Kubernetes integration. Compare the whole system you would operate. Temporary kubeadm experiments can coexist with a lower-maintenance long-lived platform.

Select by the complete lifecycle you want to operate, supported hardware and integrations, not merely installation convenience. Immutable nodes, packaged distributions and managed control planes reduce different work; none removes workload/data recovery obligations.

---

## Networking and Persistence

Choose pod/service/node ranges without household/VPN overlap. Use a compatible enforcing solution if isolation is required.

See [Networks Inside the Lab](./network.md#networks-inside-the-lab) for the node/Pod/Service distinction, household application access and DMZ limitations. Kubernetes nodes can share the Lab VLAN; Pod and Service address spaces do not normally require separate physical VLANs.

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| [Cilium](https://docs.cilium.io/) | eBPF-based networking/policy with optional Hubble visibility and service features | Policy and flow visibility in an integrated ecosystem | Kernel/platform/version prerequisites; extra resource/debugging costs; advanced routing/proxy replacement adds complexity. **Maintenance:** Track kernel/Kubernetes compatibility, controller/agent health and policy behavior; retest allowed/denied flows and any advanced routing after upgrades. | Policy/observability learning with supported compatible nodes |
| [Calico](https://docs.tigera.io/calico/latest/about/) | Pod networking and policy platform with several routing/data-plane modes | Mature policy and routing choices | Choose supported modes and routing prerequisites; own integration and upgrades. **Maintenance:** Coordinate supported data-plane/routing modes with node updates; monitor agents/routes and test policy enforcement after configuration changes. | Networking/policy requirements fit its operational model |
| [Flannel](https://github.com/flannel-io/flannel#readme) | Pod-connectivity networking component | Relatively focused connectivity role | Does not itself enforce NetworkPolicy; pair with a compatible enforcing solution if needed. **Maintenance:** Keep pod routes/overlay configuration compatible with nodes; separately maintain and test any added policy-enforcement component. | Simple connectivity with explicit separate isolation planning |
| CSI-backed volumes | Kubernetes volume lifecycle through a compatible driver/backend | Workload attachment, supported expansion/snapshots and declarative storage use | Durability belongs to the backend; check access modes, node loss and backup integration. **Maintenance:** Coordinate driver/backend upgrades and credentials; monitor attachment failures and test node-loss, snapshot and application restore behavior. | Persistent workloads with a designed [storage/recovery backend](./storage.md#design-options) |

Namespaces alone do not isolate traffic. Decide policy, cluster DNS, ingress, external address allocation and CSI/persistence separately. Cluster and gateway policies cover different paths. Application/data recovery is part of [storage and backups](../06-operations/backups.md).

A CNI supplies pod networking and may enforce policy; a service implementation provides service reachability; ingress/gateway components route application traffic; an external address allocator solves yet another problem. Choose supported combinations before enabling advanced BGP, proxy replacement or overlays. Each adds prerequisites and new failure paths.

Operational differences matter more than a feature-count contest. Verify enforcement with real allowed/denied flows, not just successful pod startup.

Storage classes/CSI expose volume lifecycle to workloads, but the backend determines durability and failure handling. Plan what happens when a node disappears, whether volumes can attach elsewhere and how application-consistent backups are restored. etcd recovery and application-data recovery are separate jobs.

---

## Implementation and Verification

After the design, pin compatible versions, prepare node images/configuration, bootstrap through the chosen supported workflow, and validate node readiness, DNS, service paths and actual policy enforcement. Do not enable advanced routing or proxy replacement just because available.

Declare VM provisioning, node lifecycle and [GitOps workload delivery](../04-automation/deployment.md#gitops-and-continuous-delivery) ownership explicitly. Test staged upgrades and recovery; declarative manifests do not restore application data. Exact commands, node inputs and execution results belong in private runbooks or official product guides.

Validate node health, DNS, pod-to-pod/service connectivity, external access, policy, volume attachment and controlled replacement separately. Drain/reboot experiments should account for disruption budgets, spare capacity and workloads that cannot move. A successful drain is not evidence of physical-host resilience.

Keep a compatible version matrix and review release notes for the OS, Kubernetes, CNI and storage controllers. Test recovery credentials and supported control-plane backup procedures outside ordinary Git. The public guide explains the lifecycle; it should not prescribe the owner's exact node count, addresses or bootstrap commands.

## Further Reading

Use the [Kubernetes documentation](https://kubernetes.io/docs/home/) for API concepts, [cluster administration](https://kubernetes.io/docs/tasks/administer-cluster/) and [version-skew policy](https://kubernetes.io/releases/version-skew-policy/). Distribution and add-on behavior is documented by each project's official sources, linked in the option tables above. This chapter explains the homelab trade-offs rather than reproducing Kubernetes theory; see [Goals and Scope](../01-foundations/goals-and-scope.md#define-what-is-out-of-scope).
