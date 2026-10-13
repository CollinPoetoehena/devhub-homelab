# Preparation

1. Define [goals](../1_Design/0_Design_Goals.md), recovery expectations and households/services affected by changes.
2. Inspect acquired equipment: firmware locks, health, virtualization support, RAM/storage, cooling, power, cabling and PoE budget. Keep actual serials, placement and purchase records private.
3. Use official images and supported releases; verify their published integrity information. Record selected versions and compatibility privately.
4. Prepare a management workstation with Git, required CLIs and a supported isolated Python environment if using Ansible. Scope credentials and trust the correct certificate authorities; do not disable TLS checks for convenience.
5. Generate/protect SSH or API identities appropriate to each platform. API-managed immutable nodes need their own credentials, not an assumed SSH key. Keep private keys and secret-store recovery outside Git.
6. If using Ansible, create the environment/inventory and encrypted vault through the selected project workflow, verify encryption and protect its password. If using Terraform, configure protected state and runtime secrets before a real plan.
7. Build a private input record for addressing, port mappings, DNS, host roles and resource allocation. Confirm values rather than converting placeholders into claimed facts.
8. Back up any system being replaced and retain a direct wired recovery path plus rollback procedure. Confirm permission before changing shared infrastructure.

Reusable roles/modules belong in independently versioned repositories, not a copy of the actual lab configuration. See [Ansible](../../ansible/README.md), [Terraform](../../terraform/README.md), and [DevHub packages](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/README.md).
