# Cluster Design

Choose orchestration for the workload and learning goal. A small service collection may need only Compose; Kubernetes teaches scheduling, reconciliation, policy and distributed lifecycle at additional cost. Compare [orchestrators, Talos, K3s, kubeadm and Kubespray](./2_Design_Software_Prerequisites.md#container-orchestration) before choosing the actual platform.

## Control plane and workers

A single control plane is reasonable for an outage-tolerant learning cluster, but is not HA. A quorum-based HA control plane commonly uses three etcd members across suitable failure domains, an API endpoint strategy and surviving capacity. Multiple virtual nodes on the same physical host teach node-level behavior without host-failure resilience.

Declare ownership of VM creation, node OS/configuration, bootstrap, upgrades and workloads. Talos lifecycle APIs, conventional Linux configuration and GitOps reconciliation manage different layers. Bootstrap must not casually recreate cluster identity/secrets on every automation run.

## Cluster networking

Keep node, pod, service and externally allocated address ranges non-overlapping with household/VPN routes. Choose CNI/policy, service exposure, DNS, ingress and load-balancer strategy deliberately. Cilium, Calico and Flannel have different policy and observability capabilities; see [CNI options](./2_Design_Software_Prerequisites.md#cluster-networking-cni).

Namespaces organize resources and RBAC; **they do not isolate traffic by themselves**. Use an enforcing NetworkPolicy implementation and explicit policies. Cluster policy and the home gateway firewall cover different paths.

Distinguish:

- Ingress/application routing from Service load-balancer address allocation.
- Node access from pod/service reachability.
- Cluster-internal DNS from household DNS dependencies.
- Private access from deliberate internet exposure with TLS, authentication and narrow firewall permissions.

## Lifecycle and recovery

Pin compatible node OS/Kubernetes/CNI/CSI versions, stage upgrades, and test connectivity and policy after changes. Budget persistent storage and backup for application data plus etcd and cluster credentials. Recreating manifests does not restore databases or PV contents.

Use [compute and cluster setup](../2_Setup/Compute_Cluster.md) for dependency order. Keep actual node identities, addresses, API endpoints, credentials and validation results private under the [guidelines](../Repository_Guidelines.md).
