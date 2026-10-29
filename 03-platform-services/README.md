# Part 3 · Platform Services

> **Layer: Architecture — what capabilities make the platform usable?** · [Book overview](../README.md) · Previous part: [Part 2 · Architecture](../02-architecture/README.md) · Next part: [Part 4 · Automation](../04-automation/README.md)

The [architecture](../02-architecture/README.md) provides connectivity, compute, storage and a workload platform. That is not yet enough to run useful services well. Services need names, trusted certificates, user and administrator identities, protected secrets and a sensible schedule for when things run. This part answers the storyline's fourth question: **what capabilities make up that architecture?**

These capabilities are shared by every service. Designing them once, deliberately, avoids each service inventing its own DNS records, self-signed certificates, local passwords and plaintext tokens. They still belong to the architecture layer: they describe *what* is provided and *why*; [Part 4 · Automation](../04-automation/README.md) describes how they are realized as code.

---

## Table of Contents

- [Chapters in This Part](#chapters-in-this-part)
- [How the Capabilities Depend on Each Other](#how-the-capabilities-depend-on-each-other)
- [Placement: Foundation, Platform or Hosted?](#placement-foundation-platform-or-hosted)
- [What Stays Private](#what-stays-private)
- [Further Reading](#further-reading)

---

## Chapters in This Part

| # | Chapter | Question it answers | What you take away |
|---|---|---|---|
| 1 | [DNS and Name Resolution](./dns.md) | How are services found by name, without making the household depend on the lab? | Naming schemes, resolver roles, DNS options, placement, split-horizon and Kubernetes DNS |
| 2 | [Certificates and TLS](./certificates.md) | How do clients trust the services they connect to? | Trust models, ACME challenges, certificate tools, certificate transparency and renewal |
| 3 | [Identity and Access](./identity.md) | Who and what may use or administer each system? | Account types, identity providers and SSO, platform privilege boundaries, admin access paths and break-glass access |
| 4 | [Secrets Management](./secrets.md) | Where do credentials and keys live, and how do they reach their consumers? | Secret inventory, lifecycle, tools, decryption points, human secrets, rotation and exposure |
| 5 | [Scheduling and Power](./scheduling.md) | When does each component need to run? | Always-on versus scheduled versus on-demand, safe automatic shutdown/startup and measuring savings |

---

## How the Capabilities Depend on Each Other

The chapters are ordered by dependency:

```text
DNS (names)
  -> Certificates (trust for those names; DNS-01 validation uses DNS)
     -> Identity (logins over TLS; SSO redirects rely on names and certificates)
        -> Secrets (credentials for identities, keys for certificates, tokens for DNS automation)
           -> Scheduling (when all of the above must be available)
```

The dependencies also run in loops that must be broken deliberately during recovery: a secret store may need DNS and TLS to start, while DNS automation and certificate issuance need secrets. The [disaster-recovery chapter](../06-operations/disaster-recovery.md#recovery-order) explains how to order recovery so that no capability depends on something that is not running yet.

---

## Placement: Foundation, Platform or Hosted?

For every capability, decide which [service tier](../01-foundations/requirements.md#service-tiers-how-much-does-it-matter) it belongs to:

- **Foundation tier:** household DNS resolution must keep working when lab compute is off. It runs on the gateway or another always-on device, not on disposable lab compute.
- **Platform tier:** cluster DNS, cert-manager, an identity provider for lab services and secret controllers can live on the lab platform—provided the household does not depend on them and break-glass access exists.
- **Hosted:** a public DNS provider, a hosted password manager or hosted identity can be the more reliable choice for capabilities that must work when the lab is down.

The [scheduling chapter](./scheduling.md) closes the loop: a capability on a host that is shut down at night is unavailable at night.

---

## What Stays Private

This part explains the options and reasoning. The actual domain and records, certificate scope, accounts, roles, secret inventory and schedules are private implementation details recorded in the private repository—and, for credentials and sensitive identifying values, only in protected storage, as required by the [repository guidelines](../GUIDELINES.md#secrets-and-sensitive-artifacts).

## Further Reading

The chapters link the relevant protocol and product documentation where each capability is discussed. Core specifications include IETF [DNS concepts (RFC 1034)](https://www.rfc-editor.org/rfc/rfc1034), [DNS implementation and specification (RFC 1035)](https://www.rfc-editor.org/rfc/rfc1035) and [ACME (RFC 8555)](https://www.rfc-editor.org/rfc/rfc8555). See [Goals and Scope](../01-foundations/goals-and-scope.md#define-what-is-out-of-scope) for why this part summarizes only homelab-relevant choices.
