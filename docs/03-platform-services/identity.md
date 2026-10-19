# Identity and Access

> **Part 3 · Platform Services** — [Part overview](./README.md) · Previous: [Certificates and TLS](./certificates.md) · Next: [Secrets Management](./secrets.md)

Network segmentation decides which *devices* can reach a service; identity and access decide which *people and systems* may use it and what they may do. In a homelab, identity starts small—one administrator, a few household users and some automation accounts—but it quietly grows: every hypervisor, controller, cluster, service and pipeline has its own users, tokens and permissions. Without a deliberate design, that becomes shared passwords, forgotten tokens and one administrator account that can do everything everywhere.

This chapter covers who needs access to what, the account types a homelab uses, centralized identity and single sign-on options, privilege boundaries in the platform, and—most importantly for recovery—break-glass access that works when the identity system itself is down. The credentials themselves are stored and delivered as described in [Secrets Management](./secrets.md); network-level access is defined in the [network chapter](../02-architecture/network.md#segmentation-and-access).

---

## Table of Contents

- [Purpose and Requirements](#purpose-and-requirements)
- [Account Types](#account-types)
- [Design Options](#design-options)
- [Privilege Boundaries in the Platform](#privilege-boundaries-in-the-platform)
- [Administrative Access Paths](#administrative-access-paths)
- [Break-Glass and Recovery Access](#break-glass-and-recovery-access)
- [Implementation and Verification](#implementation-and-verification)

---

## Purpose and Requirements

| Requirement | Why it matters | Design consequence |
|---|---|---|
| Each person and system has its own identity | Shared accounts make it impossible to revoke one user or see who did what | Personal administrator accounts and separate automation/service accounts |
| Least privilege | A compromised account or token should not grant control over everything | Roles and scoped tokens per tool and per task, as in [least privilege and explicit boundaries](../01-foundations/design-principles.md#least-privilege-and-explicit-boundaries) |
| Strong authentication for high-impact access | Management interfaces control the whole environment | SSH keys instead of passwords, MFA where supported, management reachable only from approved paths |
| Access can be removed quickly | Lost devices, finished experiments and changed household arrangements happen | An inventory of accounts and tokens; expiry for temporary access |
| Recovery does not depend on the thing being recovered | An identity provider inside the lab cannot help restore the lab | Local break-glass access for critical systems, stored outside the lab |
| Household users are not burdened | Family members should not need to understand the lab to use a service | Simple sign-in for household services, separate from administration |

---

## Account Types

| Account type | Used by | Typical authentication | Scope |
|---|---|---|---|
| Personal administrator | The owner, interactively | SSH keys with passphrases, passwords from a password manager, MFA | Administration of specific systems; ideally not used for day-to-day browsing or household services |
| Automation account | Ansible, Terraform, CI jobs | Dedicated SSH keys or API tokens | Only the actions the tool performs, on the systems it manages; no interactive login where avoidable |
| Service account | Applications and controllers talking to each other | Tokens, client credentials or platform-native identities | One application, one purpose; rotated with the application |
| Household user | Family members using services | Passwords or single sign-on, MFA where practical | Services only; no administration |
| Break-glass account | The owner, only during recovery | Strong local credentials stored offline | Full recovery access to one critical system, independent of central identity |

Separating automation accounts from personal accounts is one of the most valuable small steps: it allows tokens to be scoped and rotated without locking the owner out, and it makes logs show whether a change came from a person or a pipeline. A configuration-management role can create these accounts, their SSH keys and sudo rules consistently on every host; see [Configuration Management](../04-automation/configuration-management.md#what-configuration-management-owns).

---

## Design Options

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Local accounts per system | Each system keeps its own users, managed manually or by configuration management | No central dependency; works during outages; simple to understand | Accounts multiply; offboarding means touching every system. **Maintenance:** Manage accounts declaratively where possible, review them periodically and remove unused ones | Small homelabs, infrastructure and break-glass access |
| SSH keys managed by configuration management | Public keys and sudo rules distributed by automation | Consistent, auditable access across hosts; easy key rotation | Private keys must be protected on clients; key sprawl without review. **Maintenance:** Rotate keys when devices are lost or replaced, remove old keys from inventories, use passphrases or hardware-backed keys | Administrator and automation access to Linux hosts |
| Platform-native users and roles, e.g. [Proxmox VE user management](https://pve.proxmox.com/wiki/User_Management) and [Kubernetes RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) | Built-in authentication realms, roles and API tokens of each platform | Fine-grained permissions and scoped tokens without extra services | Each platform has its own model to learn and review. **Maintenance:** Review roles and tokens after changes, set expiry on tokens where supported and keep at least one local administrator for recovery | Scoping automation tokens and separating duties within a platform |
| Identity provider with SSO, e.g. [Authentik](https://docs.goauthentik.io/) or [Keycloak](https://www.keycloak.org/documentation) | Self-hosted provider offering OIDC/SAML single sign-on, MFA and user management | One account and MFA for many services; central offboarding; strong learning value for enterprise identity concepts | A critical service: if it is down, every dependent login fails; needs a database, backups, upgrades and careful exposure. **Maintenance:** Patch promptly, back up and test restore, monitor availability and keep break-glass access to every dependent system | Several household services or learning SSO/OIDC as an explicit goal |
| Authentication proxy, e.g. [Authelia](https://www.authelia.com/) | Forward-authentication service in front of web applications, often with MFA | Adds authentication to applications that have none or weak built-in login | Protects only traffic that passes through the proxy; misrouting can bypass it. **Maintenance:** Update, back up configuration and user data, and verify that protected applications are not reachable around the proxy | Adding a consistent login and MFA to several web applications |
| Directory service, e.g. [FreeIPA](https://www.freeipa.org/page/Documentation) or [Kanidm](https://kanidm.github.io/kanidm/stable/) | Central directory of users and groups for hosts and services, often with Linux login integration | Central accounts for host logins and services; enterprise-directory practice | Significant operational scope; host logins depend on directory availability unless cached or backed by local accounts. **Maintenance:** Operate replicas or tested backups, manage certificates and upgrades, and keep local fallback accounts | Learning directory services or managing many hosts and users |
| Hosted identity | A cloud identity provider used for SSO | Provider operates availability and security features | Internet and provider dependency; privacy and cost considerations. **Maintenance:** Protect the account with strong MFA, review applications and sessions, and keep local access for when the internet is down | Services already reachable through the internet, or when operating an identity provider is not a goal |

A central identity provider is a classic case of a useful enterprise pattern with an availability cost. It is worth it when there are enough users and services, or when learning identity is the goal—but it must not become the only way to log in to the infrastructure that runs it.

---

## Privilege Boundaries in the Platform

Least privilege applies inside each platform as well as at the network edge:

- **Hypervisor:** give automation a dedicated user or token with only the privileges it needs for the resources it manages, instead of the root administrator account. Keep personal administration separate from automation.
- **Kubernetes:** distinguish cluster administrators from namespace-scoped access, and give controllers and workloads service accounts with minimal [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) permissions. Protect the cluster-admin credentials as break-glass material.
- **CI/CD:** limit which workflows can use deployment credentials, and keep untrusted pull requests away from them, as described in [Deployment](../04-automation/deployment.md#continuous-integration).
- **Applications:** separate household user roles from application administration; do not make every household user an administrator because it is convenient.

---

## Administrative Access Paths

Identity and network access work together. A strong design restricts *where* administration can come from as well as *who* can administer:

- Management interfaces are reachable only from the management segment or approved administrator clients, as defined in the [trust roles](../02-architecture/network.md#trust-roles-and-minimum-access).
- A single, hardened entry point—such as a jump host or a VPN for administrators—can concentrate remote administrative access, logging and authentication in one place instead of exposing many interfaces. It then becomes a high-value component that needs patching, key-only authentication, a minimal package set, a host firewall and audit logging.
- Remote administration from outside the home uses [private/VPN access](../02-architecture/network.md#isp-handoff-nat-and-remote-access), never publicly exposed management interfaces.

---

## Break-Glass and Recovery Access

Every critical system needs a way in that does not depend on the systems around it being healthy:

- A local administrator account or console access on the gateway, hypervisor and backup system that does not require the identity provider, the VPN or a cloud sign-in.
- Credentials and MFA recovery codes for those accounts stored in a password manager and an offline encrypted copy, as described in [human secrets and break-glass access](./secrets.md#human-secrets-and-break-glass-access).
- A documented, periodically tested procedure for using them, included in the [disaster-recovery plan](../06-operations/disaster-recovery.md).

Break-glass accounts are powerful. Use them only for recovery, protect them strongly and review their use afterwards.

---

## Implementation and Verification

Start with an account inventory: people, automation, services and break-glass access per system. Create separate automation accounts, distribute SSH keys and sudo rules through configuration management, scope platform tokens, and only then decide whether a central identity provider adds enough value.

Verify that:

- Each automation tool works with its scoped identity and fails when it attempts something outside its scope.
- Management interfaces reject access from non-administrative segments and accounts.
- Removing a user or token actually removes access everywhere it was used.
- Break-glass access works with the identity provider, VPN and internet unavailable.
- MFA and key-based authentication are enforced where configured, and password authentication is disabled where keys are expected.

The actual accounts, roles, tokens and access paths belong in the private record; their credentials belong in protected secret storage.
