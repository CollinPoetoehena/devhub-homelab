# Software Options for a Homelab

Evaluate maintenance, hardware compatibility, licensing/support terms, recovery, automation interfaces and learning value. A choice is fit for purpose, not a popularity ranking. Verify current upstream requirements and lifecycle support before deployment.

## Routing and network management

| Option | Benefits | Costs / limits | Choose when |
|---|---|---|---|
| ISP / consumer router firmware | Few components, provider support | VLAN, SSID and firewall controls may be limited | Simple household needs are met by the exact model |
| UniFi OS + Network | Cohesive gateway/switch/AP management; local console on supported gateways | Vendor firmware/ecosystem dependence, model-specific features, not fully declarative | Reduced integration effort is more important than router-OS freedom |
| TP-Link Omada | Integrated network management and competitive options | Controller placement and feature support vary | The selected devices and management model fit requirements |
| MikroTik RouterOS | Extensive routing controls and scripting | Configuration depth and operational learning curve | Routing flexibility is a deliberate goal |
| OPNsense / pfSense | Purpose-built firewall OS, detailed control and extensions | Appliance OS/plugin lifecycle and separate AP/switch integration | Open firewall administration or learning justifies the work |
| OpenWrt | Flexible lightweight firmware on supported hardware | Hardware/driver support and updates need checking | Customization on compatible devices |
| VyOS | Routing-focused configuration model and automation potential | Image/support availability and routing expertise | Routing labs or deliberate configuration-as-code needs |
| General Linux + routing services | Maximum control and fundamental learning | You own DHCP/DNS/firewall/kernel integration and recovery | Isolated experiments, not an accidental household dependency |

Controller location and traffic forwarding are separate concerns. Confirm which functions continue during controller/cloud/lab outages. See [network architecture](./3_Design_Network.md) and [Wi-Fi](./WiFi.md).

## Hypervisor and host OS

| Option | Benefits | Costs / limits | Fit |
|---|---|---|---|
| Proxmox VE | KVM/LXC, UI/API, templates, backups and clustering in one platform | Host updates, storage and quorum remain your responsibility | Integrated virtualization on compatible home hardware |
| XCP-ng + Xen Orchestra | Xen-based platform with management and backup tooling | Different ecosystem and build/support choices | Xen learning or preferred management workflow |
| VMware ESXi / vSphere | Relevant enterprise virtualization skills | Current licensing, HCL and API/management access must be checked | Explicit VMware learning with suitable access/hardware |
| KVM/libvirt | Flexible Linux-native VMs | More integration and management assembly | Linux virtualization itself is the learning goal |
| Hyper-V | Windows-native virtualization | Edition, management and guest-support requirements | A Windows-focused lab |
| Bare-metal services/containers | Few layers and less overhead | Less isolation/rebuild flexibility; no hypervisor features | Small workloads that do not need VM separation |

Debian and Ubuntu LTS provide conventional Linux administration; RHEL/Rocky/AlmaLinux suit that ecosystem. Alpine's small footprint comes with compatibility considerations. Rolling-release distributions suit experimentation but need a deliberate change policy. Let the hypervisor's supported base OS and workload compatibility guide the choice rather than replacing its base arbitrarily.

## Container orchestration

Docker Compose suits a handful of services on one host and supports restart policies, but is not a multi-node scheduler. Kubernetes offers API-driven reconciliation, scheduling and a large ecosystem at greater operational cost. Nomad supports several workload types with a different ecosystem. Docker Swarm offers a simpler multi-node container model; assess current ecosystem and learning goals. LXC is a system-container approach, not an orchestration platform.

### Kubernetes node OS and lifecycle

These are not interchangeable layers: **Talos is a node OS**, kubeadm is bootstrap tooling, Kubespray automates deployment, and K3s is a Kubernetes distribution. Compare the total system you would maintain.

| Approach | Benefits | Costs / limits | Choose when |
|---|---|---|---|
| [Talos Linux](https://docs.siderolabs.com/) | Minimal immutable API-managed OS, declarative node lifecycle, no normal SSH/shell administration | Different debugging model; extension, storage and version compatibility; API credentials and recovery still yours | Dedicated Kubernetes nodes with reduced general-purpose OS drift |
| [K3s](https://docs.k3s.io/) / [k0s](https://docs.k0sproject.io/) | Convenient lightweight distributions | Host OS still needs maintaining; packaged defaults/integrations need review | Limited resources or simpler conventional-Linux deployment |
| [kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/) on Linux | Direct upstream bootstrap experience, host flexibility | OS/runtime/kubelet integration, upgrades and recovery require explicit processes | Learning assembly or needing conventional OS control |
| [Kubespray](https://kubespray.io/) | Ansible-driven deployment/lifecycle across supported hosts | Inventory, playbooks, compatibility and Linux host maintenance | Automated conventional-Linux clusters |
| [MicroK8s](https://microk8s.io/) | Convenient packaged Kubernetes and add-ons | Host and snap lifecycle/updates remain | Its platform and update model suit the lab |
| [RKE2](https://docs.rke2.io/) | Security-focused distribution and packaged lifecycle | Underlying Linux OS and integration still maintained | Its security/operational model is required |
| [OpenShift](https://docs.redhat.com/en/documentation/openshift_container_platform/) / [OKD](https://okd.io/) | Integrated operator-managed application platform | Larger resource and platform scope; distinct upgrade/support requirements | Explicit OpenShift-platform learning, not merely small Kubernetes |
| Managed cloud Kubernetes (AKS/EKS/GKE) | Provider manages parts of lifecycle/control plane | Cost and provider dependency; workloads, identity, networking and data still yours | Cloud learning or offloading local infrastructure |

Talos is not a managed cloud service; declarative nodes still need staged updates, compatible CNI/CSI, etcd backups and restore testing. Disposable kubeadm experiments can coexist with a lower-maintenance long-lived cluster.

### Cluster networking (CNI)

| Option | Benefits | Costs / limits | Choose when |
|---|---|---|---|
| Cilium | eBPF networking, policy, optional Hubble visibility and advanced features | Resource use, kernel/integration compatibility and troubleshooting depth | Networking observability or eBPF capabilities matter |
| Calico | Mature networking and policy, multiple dataplane choices | Configure the selected mode and routing carefully | Its policy/routing ecosystem fits |
| Flannel | Simple pod connectivity | Does not itself enforce Kubernetes NetworkPolicy | Connectivity is enough or a separate policy solution is deliberately added |

Do not enable kube-proxy replacement, BGP, or advanced load balancing solely because a CNI supports them. Start with a compatible basic configuration; verify policies before adding features.

## Provisioning and configuration management

Terraform and OpenTofu offer declarative provider-based provisioning with state and plans; choose based on supported providers, licensing, compatibility and organizational practice. Pulumi provides programming-language infrastructure definitions, useful when that model suits the team. All require review and protected state. Ansible can provision too, but its state/reconciliation model differs; a check-mode result is not equivalent to a Terraform plan.

Use Ansible for agentless desired-state configuration on supported Linux hosts. Puppet, Chef and Salt offer different continuous-enforcement/agent models with extra infrastructure or learning costs. Scripts can be appropriate for small explicit tasks but require their own idempotency and error handling. Packer/golden images complement configuration management rather than automatically replacing it.

Declare ownership: a provisioner creates VM resources, an image process prepares templates, host configuration manages supported machines, and Kubernetes/GitOps manages workloads. Do not have two tools fight over the same resource. See [Ansible guidance](../../ansible/README.md) and [Terraform guidance](../../terraform/README.md).

## Delivery and secrets

| Capability | Options and trade-offs |
|---|---|
| GitOps | Argo CD offers visual sync/diff workflows; Flux offers a controller/CLI-oriented model. Both reconcile desired state. A manual Helm install is packaging without ongoing reconciliation. |
| Push-based delivery | CI-driven deployment is straightforward, but needs deployment credentials and an explicit drift-management approach. |
| CI | GitHub Actions/GitLab CI provide hosted options; Jenkins, Woodpecker or Gitea Actions can be self-hosted with additional maintenance. Tekton/Argo Workflows suit deliberate Kubernetes-native workflow needs. Check runner trust, billing and permissions. |
| Git-native secrets | SOPS + age can encrypt files without a dedicated server; keys and the controller's decryption/delivery integration need protection and explicit configuration. |
| External secrets | Vault or an external manager can provide dynamic credentials and auditing, at the cost of availability, identity, backup and recovery responsibilities. External Secrets Operator integrates supported external stores; it is not a store itself. |
| Ansible secrets | Ansible Vault suits encrypted Ansible data; its password remains outside Git. |
| Kubernetes Secrets | A runtime delivery object, not a safe plaintext/base64 Git storage format; configure access and encryption at rest appropriately. |

Generic [DevHub workflows](https://github.com/CollinPoetoehena/DevHub/tree/main/.github/workflows) and [AI skills](https://github.com/CollinPoetoehena/DevHub/tree/main/.github/skills) remain shared. Homelab-specific guidance adds context, not another copy of those workflows.

## Observability, storage and backup tools

Prometheus integrates time-series metrics with Kubernetes discovery; InfluxDB suits other telemetry models; Zabbix/Checkmk/Nagios can suit traditional infrastructure monitoring. Loki provides label-indexed log aggregation; Elasticsearch/OpenSearch support full-text search with a different resource/operational budget. Grafana can query multiple sources; Alertmanager groups/routes Prometheus alerts, while Grafana Alerting supports another centralized alerting workflow. Local vendor UIs remain useful when central monitoring is unavailable. See [monitoring design](./7_Design_Monitoring.md).

Local disks/ZFS, a NAS over NFS, and distributed storage such as Ceph solve different needs; none automatically supplies backups. Proxmox Backup Server can provide deduplicated VM/container backups; `vzdump`, file backup tools and commercial products offer other workflows. Compare consistency, encryption, retention, restore testing and off-site copies in [storage and backups](./8_Design_Storage_Backups.md), not only backup file size.
