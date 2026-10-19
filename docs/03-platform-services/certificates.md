# Certificates and TLS

> **Part 3 · Platform Services** — [Part overview](./README.md) · Previous: [DNS and Name Resolution](./dns.md) · Next: [Identity and Access](./identity.md)

Once services have [names](./dns.md), they need certificates so that clients can connect with TLS: traffic is encrypted, and the client can verify it is talking to the intended service rather than an impostor. Without a deliberate certificate design, a homelab ends up with self-signed warnings everywhere—and people learn to click through warnings, which defeats the purpose of TLS.

This chapter covers the trust models available to a homelab, how certificates are issued and renewed automatically, what publicly trusted certificates reveal, and how to protect private keys. Names come from the [DNS chapter](./dns.md); keys are secrets managed as described in [Secrets Management](./secrets.md).

---

## Table of Contents

- [Purpose and Requirements](#purpose-and-requirements)
- [Trust Models](#trust-models)
- [ACME Validation Methods](#acme-validation-methods)
- [Design Options](#design-options)
- [Certificate Transparency and Internal Names](#certificate-transparency-and-internal-names)
- [Renewal, Expiry and Keys](#renewal-expiry-and-keys)
- [Implementation and Verification](#implementation-and-verification)

---

## Purpose and Requirements

| Requirement | Why it matters | Design consequence |
|---|---|---|
| Clients trust service certificates without warnings | Warnings train users to ignore real problems | Use a CA that household clients trust, or distribute an internal CA deliberately |
| Certificates renew automatically | Short certificate lifetimes make manual renewal an outage waiting to happen | ACME or another automated issuance workflow, with expiry monitoring |
| Internal names are not exposed unnecessarily | Publicly trusted certificates are published in transparency logs | Choose naming and certificate scope with [certificate transparency](#certificate-transparency-and-internal-names) in mind |
| Private keys stay private | A stolen key lets an attacker impersonate the service | Keys stored and delivered as [secrets](./secrets.md); wildcard keys limited to few places |
| Management interfaces use TLS too | Hypervisor, controller and gateway administration carries credentials | Replace or accept vendor certificates deliberately; never disable verification in automation by default |

---

## Trust Models

There are three basic ways for a client to trust a certificate:

1. **A publicly trusted CA** issues the certificate. Every common browser and operating system already trusts it, so nothing has to be distributed to clients. The CA must verify control of the name, which requires either public reachability or control of public DNS for that name.
2. **An internal (private) CA** issues the certificate. It can issue for any internal name, including `home.arpa`, without public validation—but every client that should trust it must have its root certificate installed, and the root key becomes a highly sensitive secret.
3. **Self-signed or vendor-default certificates** are trusted individually or not at all. They encrypt traffic but provide no real identity verification unless each certificate is pinned or manually trusted.

Most homelabs combine these: publicly trusted certificates for services used by household devices, an internal CA or accepted vendor certificates for management interfaces, and self-signed certificates only for short-lived experiments.

---

## ACME Validation Methods

The [ACME protocol](https://www.rfc-editor.org/rfc/rfc8555) automates issuance and renewal with CAs such as [Let's Encrypt](https://letsencrypt.org/docs/). The CA verifies control of the name through a [challenge](https://letsencrypt.org/docs/challenge-types/):

| Challenge | How control is proven | Fit for a homelab |
|---|---|---|
| HTTP-01 | A token is served over HTTP on port 80 of the name being validated | Requires the service to be reachable from the internet on port 80; unsuitable for internal-only services and cannot issue wildcard certificates |
| DNS-01 | A TXT record is created in the public DNS zone of the name | Works for internal-only services and wildcard certificates, without opening inbound ports; requires an owned domain and a DNS provider API token that can modify the zone |
| TLS-ALPN-01 | A special certificate is presented on port 443 during validation | Requires public reachability on port 443; supported by fewer clients |

DNS-01 is usually the best fit for a homelab with an owned domain, because nothing has to be exposed to the internet. The trade-off is the DNS API token: it can modify the zone, so scope it as narrowly as the provider allows and protect it as a high-impact [secret](./secrets.md).

---

## Design Options

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Publicly trusted ACME certificates, e.g. [Let's Encrypt](https://letsencrypt.org/docs/) | Free, automated certificates from a public CA through ACME | Trusted by household devices without distributing anything; automated renewal | Needs an owned domain and a validation path; names are publicly logged; rate limits apply. **Maintenance:** Monitor renewals and expiry, protect the DNS API token or exposed validation path, and follow the CA's policy changes | Services used from ordinary household devices and browsers |
| [cert-manager](https://cert-manager.io/docs/) | Kubernetes controller that requests, renews and stores certificates as Secrets | Certificates follow Ingress/Gateway resources declaratively; supports ACME and internal issuers | Another cluster component with credentials to a DNS provider or CA; certificates live in Kubernetes Secrets. **Maintenance:** Upgrade with Kubernetes compatibility in mind, monitor failed orders and expiry, and back up issuer configuration | Platform-hosted services on Kubernetes |
| Reverse proxy with built-in ACME, e.g. [Caddy](https://caddyserver.com/docs/automatic-https) or [Traefik](https://doc.traefik.io/traefik/https/acme/) | A proxy that obtains and renews certificates for the names it serves | Very little configuration; TLS handled at one entry point | The proxy holds all keys for the names it serves and becomes a central dependency. **Maintenance:** Update the proxy, persist and back up its certificate storage, and monitor renewal | Single-host or Compose-based services behind one entry point |
| Internal CA, e.g. [step-ca](https://smallstep.com/docs/step-ca/) | A self-hosted certificate authority, optionally with an ACME interface | Certificates for any internal name, including `home.arpa`; nothing published externally | Root must be installed on every client; the root key is a crown-jewel secret; the CA itself must be available for renewals. **Maintenance:** Protect the root key offline, operate and back up the CA, rotate intermediates, and manage client trust stores | Internal-only names, management interfaces and learning PKI |
| Vendor or platform certificates | Certificates generated by appliances and platforms, e.g. hypervisor or controller defaults | Work out of the box | Usually self-signed; clients warn or need manual trust. **Maintenance:** Track regeneration after updates or reinstallation; replace with CA-issued certificates where supported if warnings become routine | Low-exposure management interfaces where the trade-off is accepted and documented |
| Self-signed per service | A certificate signed by its own key | Instant, no dependencies | No meaningful identity verification without pinning; trains users to accept warnings. **Maintenance:** Manual renewal and trust management for each service | Short-lived experiments only |

---

## Certificate Transparency and Internal Names

Publicly trusted certificates are recorded in public [Certificate Transparency](https://certificate.transparency.dev/) logs. Anyone can search those logs for every name a certificate was issued for. For a homelab, this means a publicly trusted certificate for an internal service name publishes that name—even if the service is never reachable from the internet.

That matters under this book's [publication boundary](../repository-guidelines.md#publication-boundary): hostnames such as those of management interfaces can reveal the structure of the network. Ways to limit what is logged:

- Use a **wildcard certificate** for an internal subdomain, so only the wildcard name is logged rather than each service name. The trade-off is that one key now covers every name under it; keep it in as few places as possible.
- Use an **internal CA** for names that should never appear publicly, such as management interfaces.
- Choose neutral service names that do not describe the device, model or role in detail.

---

## Renewal, Expiry and Keys

Public certificates are short-lived, and maximum lifetimes are being reduced further over time; check the CA's current policy rather than assuming a fixed number. Treat renewal as an automated process with monitoring, not a calendar reminder: expired certificates are one of the most common causes of self-inflicted homelab outages. Add certificate-expiry checks to [observability](../06-operations/observability.md), alerting well before expiry so that a failed renewal can be fixed calmly.

Private keys are [secrets](./secrets.md). Store them encrypted, limit which systems hold them, avoid copying a wildcard key to many hosts, and rotate them after any suspected exposure. If an internal CA is used, keep its root key offline and use an intermediate for day-to-day issuance; installing a root certificate on household devices gives that CA the power to issue certificates those devices will trust for *any* name, so protect it accordingly.

---

## Implementation and Verification

Decide the naming scheme and trust model per group of services, then configure issuance where certificates are consumed—an ingress controller, a reverse proxy or the service itself—and avoid several tools issuing certificates for the same name. Verify that:

- Clients on each relevant segment connect without warnings and see the expected certificate chain.
- Renewal works before it is needed: trigger or simulate a renewal, and confirm the service reloads the new certificate.
- Expiry alerts fire when a test certificate approaches expiry.
- Keys and DNS API tokens are stored as protected secrets and not visible in logs, manifests or repositories.
- Automation verifies TLS by default; any exception for a vendor certificate is explicit and documented.

The actual domain, certificate scope, CA configuration and issued names belong in the private record.
