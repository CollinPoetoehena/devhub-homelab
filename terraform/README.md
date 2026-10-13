# Terraform in a Homelab

Terraform/OpenTofu can provision VM resources from reviewed desired-state definitions. Declare what the provider owns, which existing templates/bridges/storage it consumes, and which separate mechanism configures guests or bootstraps Kubernetes.

The public repository documents capabilities and safe workflows, not actual root configuration, endpoints, node maps, provider output or state. Reusable modules should be independently maintained with generic inputs, validation, mocked/offline behavior tests and a deliberate release process.

A module cloning cloud-init Linux guests is not automatically suitable for Talos. OS image, guest-agent behavior, network/bootstrap and secret lifecycle require a compatible explicit implementation.

Protect state, plans and generated output even in the private repo; runtime credentials do not cease to be sensitive when present in state. Review a real plan locally before an explicitly approved apply. Validation/mocked tests do not prove live cloning or connectivity.

See [software options](../docs/1_Design/2_Design_Software_Prerequisites.md#provisioning-and-configuration-management), [compute setup](../docs/2_Setup/Compute_Cluster.md), [repository guidelines](../docs/Repository_Guidelines.md#secrets-and-sensitive-artifacts), and [DevHub Terraform publishing guidance](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/Terraform.md).
