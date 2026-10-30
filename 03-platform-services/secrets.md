# Secrets Management

> **Part 3 · Platform Services** — [Part overview](./README.md) · Previous: [Identity and Access](./identity.md) · Next: [Scheduling and Power](./scheduling.md)

Almost every component of a homelab needs a secret: an administrator password, an SSH key, a hypervisor API token, a Wi-Fi passphrase, a database password, a backup encryption key. Secrets management is the platform capability that decides **where those secrets live, how they reach the systems that use them, who can read them, and how they are rotated and recovered**. Done casually, it becomes the weakest point of an otherwise careful design; done deliberately, it lets configuration be versioned and automated without exposing credentials.

This chapter covers the architecture and tool options for secrets in a homelab. The binding repository *policy*—what may never be committed, how sensitive endpoints are treated and what to do after an exposure—is owned by the [repository guidelines](../GUIDELINES.md#secrets-and-sensitive-artifacts); this chapter applies that policy to the design of the lab and links back to it instead of repeating it.

---

## Table of Contents

- [Purpose and Requirements](#purpose-and-requirements)
- [The Secret Lifecycle](#the-secret-lifecycle)
- [Design Options](#design-options)
- [Where Decryption Happens](#where-decryption-happens)
- [Human Secrets and Break-Glass Access](#human-secrets-and-break-glass-access)
- [Rotation and Exposure](#rotation-and-exposure)
- [Implementation and Verification](#implementation-and-verification)
- [Further Reading](#further-reading)

---

## Purpose and Requirements

Start by listing the kinds of secrets the homelab has, rather than by choosing a tool. Different kinds have different consumers, risks and recovery needs, and one tool rarely fits all of them well.

| Kind of secret | Examples | Who or what uses it | Typical home | Recovery consideration |
|---|---|---|---|---|
| Human administrator credentials | Gateway, hypervisor and controller logins, MFA recovery codes | The owner, interactively | A password manager, with recovery codes stored offline | Must be available when the lab, the internet or a cloud account is down |
| Automation credentials | SSH private keys, hypervisor/API tokens, cloud tokens | Ansible, Terraform, CI jobs | Outside Git: a local protected key store, encrypted variables or a secret manager | Losing them blocks automation; leaking them grants broad access, so scope them narrowly |
| Encryption keys | Ansible Vault password, age private key, backup repository keys | Decryption steps in automation and restore procedures | Separately protected from the data they decrypt | Losing them can make good backups and encrypted configuration unusable |
| Workload secrets | Database passwords, application API keys, OAuth client secrets | Applications at runtime | Encrypted in Git (for example SOPS) or synchronized from a secret store | Rotation must reach the consumer; rendered Kubernetes Secrets are not encrypted by default |
| Device and network secrets | Wi-Fi passphrases, VPN keys, switch/AP credentials | Network equipment and clients | Device configuration plus protected exports | Device exports often contain these secrets; protect the [backups](../06-operations/backups.md) accordingly |
| Sensitive identifying values | Actual public IPs, global IPv6 prefixes, identifying domains and endpoints | Automation and documentation | Encrypted or protected storage, referenced by name in prose | Not passwords, but treated like secrets under the [repository guidelines](../GUIDELINES.md#secrets-and-sensitive-artifacts) |

Typical requirements are: no plaintext secrets in Git (public **or** private), automation can run without a person typing every credential, secrets can be rotated without rebuilding everything, and the material needed for [disaster recovery](../06-operations/disaster-recovery.md) is reachable when the lab itself is broken.

---

## The Secret Lifecycle

Every secret goes through the same stages. Designing each stage explicitly prevents the common failures—a token that never expires, a key that exists only on one laptop, a password that was rotated in one place but not in its consumer.

1. **Create** with sufficient strength and the narrowest scope that works, for example a hypervisor token limited to the resources Terraform manages instead of a full administrator password.
2. **Store** in one authoritative place per secret, encrypted at rest and with access limited to the people and identities that need it.
3. **Deliver** to the consumer at the moment it is needed, without writing plaintext to logs, diffs, rendered artifacts or command history.
4. **Use** with least privilege; avoid sharing one credential between several systems so that revoking it does not break unrelated components.
5. **Rotate** periodically for high-impact credentials, and immediately after a suspected exposure or when access should end.
6. **Revoke and remove** credentials of retired systems, finished experiments and lost devices.
7. **Recover**: keep the material needed to regain access—recovery codes, decryption keys, break-glass credentials—available independently of the systems it protects.

---

## Design Options

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| [SOPS](https://getsops.io/docs/) | Encrypts selected structured-file values using supported key mechanisms | Git-native review/versioning of encrypted configuration | Protect keys outside Git; configure explicit decryption and prevent plaintext diffs/logs/artifacts. **Maintenance:** update recipient/access rules, rotate compromised material and test decryption/recovery separately from controller health | Encrypted configuration in an owned delivery workflow |
| [age](https://github.com/FiloSottile/age#readme) | Encryption tool/key option, including use with SOPS | Simple key-based encryption workflow | Key custody, rotation and recovery; not a delivery controller. **Maintenance:** Protect and rotate keys/recipients when access changes; test recovery with separately stored keys and do not confuse encryption with secret rotation. | A supported encryption mechanism for protected values |
| [Vault](https://developer.hashicorp.com/vault/docs) | External secret-management service | Centralized secret lifecycle and supported dynamic credentials | Store availability, identity, policy, keys and tested recovery. **Maintenance:** Substantial service/identity/policy lifecycle; monitor availability, renew credentials and test storage/unseal/recovery procedures for the chosen mode. | Centralized/runtime secret needs justify another service |
| [External Secrets Operator](https://external-secrets.io/latest/) | Synchronizes supported external stores into Kubernetes | Connects workload secret delivery to a store | Not itself a store; protect store credentials and resulting Secrets. **Maintenance:** Update controllers/store integrations and access identities; monitor failed synchronization and verify rotation reaches consumers safely. | A supported external manager owns secret lifecycle |
| [Ansible Vault](https://docs.ansible.com/ansible/latest/vault_guide/index.html) | Encryption for host-automation files/variables | Protects Ansible inputs in tracked encrypted form | Vault-key protection and plaintext handling during runs. **Maintenance:** Protect vault passwords, review authorized decryptors and rotate exposed secrets; prevent plaintext in run logs or temporary outputs. | Host configuration consumes encrypted variables |
| [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) | Kubernetes objects consumed by workloads | Native application integration and access controls | Base64 is not encryption; configure RBAC, storage encryption and safe delivery. **Maintenance:** Maintain RBAC, supported storage encryption and consumer rotation/reload behavior; review access and avoid plaintext rendered artifacts. | Runtime consumption, not a substitute for protected source/key storage |

Protect identifying endpoints as well as credentials in private records under the [shared guideline](../GUIDELINES.md#secrets-and-sensitive-artifacts). Choosing an encryption tool does not automatically wire decryption into a delivery controller.

Two additional options cover needs the table above does not: secrets that people use interactively, and Kubernetes-native encryption without SOPS.

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Password manager, e.g. [Bitwarden](https://bitwarden.com/help/) or [KeePassXC](https://keepassxc.org/docs/) | Encrypted vault for credentials people use interactively, hosted or as a local database file | Strong unique passwords, MFA seeds/recovery codes and notes in one protected place; usable during a lab outage when hosted or stored locally | Becomes a high-value target and a recovery dependency; a self-hosted vault inside the lab can be unavailable exactly when it is needed. **Maintenance:** Protect the master secret and MFA, keep offline/exported recovery copies according to the product's guidance, review shared entries and test access from a second device | Human administrator credentials, recovery codes and break-glass material |
| [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets#readme) | Kubernetes controller that decrypts encrypted resources committed to Git into ordinary Secrets inside one cluster | Git-friendly workload secrets without an external store; encryption is tied to the cluster's controller key | Encrypted values are bound to that controller key; rebuilding the cluster requires backing up and restoring the key. **Maintenance:** Back up and protect the controller key, re-encrypt when keys rotate or clusters are replaced, and keep the controller compatible with Kubernetes upgrades | Single-cluster GitOps where SOPS or an external store is not preferred |

Choose where decryption occurs and who can access the resulting plaintext. Protect decryption keys outside Git, constrain controller/runner identity and avoid plaintext in rendered diffs, logs or artifacts. A private repository or a base64-encoded Kubernetes Secret is not a substitute for that design.

---

## Where Decryption Happens

Choosing an encryption tool answers *how secrets are stored*; the more important design question is **where plaintext appears**. Each delivery pattern below moves that point to a different identity, with different exposure and recovery consequences.

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Decrypt at automation run time | Ansible decrypts Vault-encrypted variables on the control node during a run | Simple, no extra runtime service; secrets reach hosts only as configuration | The control node and its operator see plaintext; task output and diffs can leak values. **Maintenance:** Protect the vault password outside Git, mark sensitive tasks with output suppression where supported, and review logs/artifacts after changes | Host and device configuration managed by [configuration management](../04-automation/configuration-management.md) |
| Decrypt in the GitOps controller | The cluster's delivery controller decrypts SOPS-encrypted manifests, for example Flux's [SOPS integration](https://fluxcd.io/flux/guides/mozilla-sops/) or an Argo CD [secret-management approach](https://argo-cd.readthedocs.io/en/stable/operator-manual/secret-management/) | Encrypted values stay in Git; the cluster reconciles them like other resources | The controller identity holds the decryption key; supported integration differs per controller and must be configured explicitly. **Maintenance:** Rotate recipients and keys deliberately, back up the key outside the cluster and verify decryption after controller upgrades | Workload secrets delivered by [GitOps](../04-automation/deployment.md#gitops-and-continuous-delivery) |
| Inject in CI/CD | The pipeline platform supplies scoped secrets to trusted jobs at run time | No secret files in the repository; platform-managed access controls and audit | Runner trust is decisive; untrusted pull requests must never reach deployment secrets. **Maintenance:** Scope secrets per environment, require approvals, review workflow permissions and rotate after runner or workflow changes | Credentials needed only by build or deployment jobs; see [CI](../04-automation/deployment.md#continuous-integration) |
| Synchronize from an external store | A store such as Vault is the source of truth; an operator such as External Secrets Operator creates Kubernetes Secrets from it | Central rotation, audit and optionally dynamic credentials | The store becomes a critical dependency that must be available, backed up and recoverable before the workloads that need it. **Maintenance:** Operate, upgrade and back up the store; monitor synchronization failures and test recovery order | Several consumers share secrets, or dynamic credentials are an explicit learning goal |

Keep the number of patterns small. A homelab that uses Ansible Vault for host configuration and SOPS with one controller-specific integration for workloads already has two keys to protect and recover; adding a secret-store service is justified when its centralized lifecycle provides more value than the extra platform it introduces.

---

## Human Secrets and Break-Glass Access

Automation secrets are only half of the picture. The owner also needs credentials to log in, approve changes and—most importantly—recover after something breaks. Plan that access as part of the [disaster-recovery](../06-operations/disaster-recovery.md) design:

- Keep administrator credentials, MFA recovery codes and decryption keys in a password manager or equivalent protected store that does **not** depend on the homelab being up. A vault hosted only inside the lab cannot help restore the lab.
- Keep at least one **break-glass** path per critical system: a local administrator account or console access that does not depend on a central identity provider, a VPN or a cloud sign-in. Protect it strongly and use it only for recovery. See [Identity and Access](./identity.md#break-glass-and-recovery-access).
- Store an encrypted, offline copy of the recovery material in a separate place from the equipment, with a documented way to decrypt it.
- Test the recovery path periodically: can the gateway, hypervisor and backup system be reached and the backups decrypted using only the recovery material?

---

## Rotation and Exposure

Rotate credentials when there is a reason—suspected exposure, a lost device, a person or system no longer needing access, a scheduled interval for high-impact credentials—and make sure the consumer receives the new value. Short-lived, automatically issued credentials remove much of this work where a platform supports them, at the cost of a more complex issuing service.

If a secret or sensitive value is exposed, follow the repository's [exposure procedure](../GUIDELINES.md#if-information-is-exposed): stop further publication, revoke or rotate first, and only then clean up. Deleting a file or rewriting history does not undo disclosure.

---

## Implementation and Verification

Decide the secret inventory, the authoritative store for each kind and the decryption point for each consumer before writing automation. Then:

- Verify that encrypted files really are encrypted before committing them—for example, check the vault header or SOPS metadata—and that ignore rules cover decrypted copies, key files, state and plans. Ignore rules are a guardrail, not protection for files that are already tracked.
- Run automation and deployments once with attention to output: no secret values in task output, diffs, CI logs, rendered manifests or error messages.
- Test rotation for at least one credential of each kind end to end, including reload or restart of the consumer.
- Test recovery from scratch: with only the documented recovery material, decrypt a backup, regain administrator access and run the automation that redeploys secrets.
- Review access periodically: who and which identities can decrypt what, and whether retired systems still hold valid credentials.

The public book describes these patterns with generic examples. The actual inventory of secrets, their locations, recipients and rotation records belong in the private record and in protected storage—never in either repository in plaintext.

## Further Reading

For cryptographic key lifecycle and protection, see NIST's [Recommendation for Key Management (SP 800-57 Part 1 Rev. 5)](https://csrc.nist.gov/pubs/sp/800/57/pt1/r5/final). For practical application-secret handling, see OWASP's [Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html), alongside the official tool documentation linked in the options table. This chapter keeps the scope to homelab workflows; see [Goals and Scope](../01-foundations/goals-and-scope.md#define-what-is-out-of-scope).
