# Compute and Cluster Setup Workflow

Use the [hypervisor](../1_Design/4_Design_Hypervisor.md), [node](../1_Design/5_Design_Hosts_Nodes.md), and [cluster](../1_Design/6_Design_Cluster.md) design references. This guide does not apply changes to a particular installation.

## Hosts and guests

1. Install the supported hypervisor/OS image after confirming disk erasure scope and recovery. Configure management networking to match the switch profile.
2. Establish trusted TLS, restricted administrative accounts and a scoped automation identity. Do not use unrestricted root/administrator API credentials merely because an example does.
3. Prepare storage, VLAN-aware guest bridges and image/templates through the chosen host/image workflow. Verify template provenance and cloud-init support where used.
4. Review a provisioner's plan before creating guests. Supply resource placement, disk/network settings and non-secret inputs from the private configuration; obtain credentials through runtime secret storage.
5. Configure conventional Linux guests with cloud-init/Ansible as designed. Do not apply SSH/package-management assumptions unchanged to Talos or another API-managed node OS.
6. Validate guest startup, addressing, management restrictions and application reachability before adding cluster dependencies.

## Kubernetes

1. Pin compatible node OS, Kubernetes, CNI and storage-driver versions. Select node placement and an API endpoint suited to the intended failure model.
2. For Talos, use the [official documentation](https://docs.siderolabs.com/) for image preparation, machine configuration, API access and bootstrap. Protect generated machine/client secrets and any state containing them.
3. Bootstrap once under the selected lifecycle process; do not regenerate cluster identity on every run. For kubeadm/K3s/Kubespray, use their corresponding supported install and upgrade workflows instead.
4. Install/configure the chosen CNI. With Talos/Cilium, follow the [integration guide](https://docs.siderolabs.com/kubernetes-guides/cni/deploying-cilium); disable conflicting defaults as directed and add advanced features only after the basic path works.
5. Verify node readiness, DNS, pod/service connectivity and **enforced** allowed/denied policies. Namespaces alone are not policy.
6. Add supported CSI/storage and deliberate service exposure. Decide ingress, load-balancer address allocation and TLS/authentication separately; do not expose administration by default.
7. Back up cluster state and application data, test isolated restores, and document upgrade/rollback behavior. A ready node is not evidence that failover or recovery works.

Terraform VM provisioning, node configuration/bootstrap and GitOps workload reconciliation need explicit ownership boundaries. See [Terraform guidance](../../terraform/README.md) and [services setup](./Services_Operations.md).
