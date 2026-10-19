# DNS and Name Resolution

> **Part 3 · Platform Services** — [Part overview](./README.md) · Next: [Certificates and TLS](./certificates.md)

Every other platform service depends on names. Users reach applications by name, certificates are issued for names, automation and monitoring address hosts by name, and Kubernetes discovers services by name. DNS is also one of the easiest ways to break a household network by accident: if the only resolver runs on a lab VM, turning off the lab turns off "the internet" for everyone, even though routing still works.

This chapter applies [DevHub's DHCP and DNS reference](https://github.com/CollinPoetoehena/DevHub/blob/main/reference/network/DHCP_and_DNS.md) to a home-and-lab environment: choosing a naming scheme, deciding which resolvers exist and where they run, keeping household resolution independent of experiments, and automating records without creating competing owners. Network segmentation, gateways and DHCP placement are covered in the [network chapter](../02-architecture/network.md); this chapter builds on them.

---

## Table of Contents

- [Purpose and Requirements](#purpose-and-requirements)
- [Choosing a Naming Scheme](#choosing-a-naming-scheme)
- [Resolver and Zone Roles](#resolver-and-zone-roles)
- [Design Options](#design-options)
- [Placement and Availability](#placement-and-availability)
- [Split-Horizon and Public DNS](#split-horizon-and-public-dns)
- [DNS Inside Kubernetes](#dns-inside-kubernetes)
- [Implementation and Verification](#implementation-and-verification)

---

## Purpose and Requirements

A homelab's DNS design typically has to satisfy these requirements:

| Requirement | Why it matters | Design consequence |
|---|---|---|
| Household resolution survives lab outages | Name resolution is part of household connectivity—the [foundation tier](../01-foundations/requirements.md#service-tiers-how-much-does-it-matter) | The resolver handed out to household clients does not depend on disposable lab compute |
| Lab services have stable names | Users and automation should not depend on addresses that change | An internal zone with records for selected services, managed by one owner |
| Names work with certificates | Browsers and clients expect valid TLS for named services | The naming scheme must allow the chosen [certificate approach](./certificates.md) |
| Internal names are not exposed unnecessarily | Hostnames can reveal the structure of the network | Internal records stay in internal DNS; public DNS contains only deliberately public names |
| Records stay consistent with reality | Stale records point to removed or reassigned systems | Automated or reviewed record management, with record removal part of [retiring a service](../05-services/workloads.md#retiring-a-service) |

DNS does not grant access. A record pointing to a lab application does not open the firewall; the [network policy](../02-architecture/network.md#reaching-applications-from-home) still decides who can connect.

---

## Choosing a Naming Scheme

The domain used for internal names affects certificates, client behavior and future flexibility. Choose it before creating many records, because renaming later touches every client, certificate and configuration.

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| [`home.arpa`](https://www.rfc-editor.org/rfc/rfc8375) | The special-use domain reserved for residential home networks | Designed for this purpose; never delegated publicly, so it cannot collide with a real domain | Public certificate authorities cannot issue certificates for it; trusted TLS requires an internal CA. **Maintenance:** Little naming upkeep; maintain the internal CA and client trust if TLS is used | Internal-only names where an internal CA or plain HTTP inside the lab is acceptable |
| Subdomain of a domain you own, e.g. `lab.example.com` | Internal names under a registered domain, served internally and optionally partly in public DNS | Enables publicly trusted certificates through the DNS-01 challenge; consistent names inside and outside | Requires owning and renewing the domain and protecting the DNS provider account; issued certificates are publicly logged. **Maintenance:** Renew the domain, protect provider credentials and API tokens, review public records and certificate-transparency exposure | Trusted TLS without distributing an internal CA, or services that may later become public |
| `.local` | The multicast DNS (mDNS) namespace defined in [RFC 6762](https://www.rfc-editor.org/rfc/rfc6762) | Zero-configuration discovery for devices that support mDNS | Reserved for mDNS; using it for unicast DNS zones causes inconsistent resolution across clients. **Maintenance:** Do not build a unicast zone on it; leave it to device discovery | Device discovery only, not an internal DNS zone |
| Invented top-level domain | A made-up suffix such as `.lab` or `.internal-something` | Short and convenient | May collide with a real or future public TLD; some real TLDs enforce HTTPS in browsers; no public certificates. **Maintenance:** Risk of breakage when the suffix becomes public or client behavior changes | Avoid for anything long-lived |

Whatever scheme is chosen, keep the actual domain and its records private. Public examples in this book use `example.com` and `home.arpa`, as required by the [publication boundary](../repository-guidelines.md#publication-boundary).

---

## Resolver and Zone Roles

DNS in a homelab is several roles that are often combined in one device. Separating them conceptually makes placement and failure behavior easier to reason about:

| Role | Responsibility | Typical location |
|---|---|---|
| Client resolver handed out by DHCP | The address clients query for every name | Gateway or a dedicated, always-on resolver |
| Recursive resolution | Resolves public names by walking the DNS hierarchy or forwarding to an upstream | Gateway, a local recursive resolver or an upstream provider |
| Filtering | Blocks advertising, tracking or malware domains for selected segments | A filtering resolver in front of recursion |
| Internal authoritative zone | Answers for internal names such as lab services | Gateway local records, a dedicated DNS server or the filtering resolver |
| Public authoritative zone | Answers for deliberately public names on the internet | A DNS provider, not the homelab |
| Cluster DNS | Service discovery inside Kubernetes | CoreDNS inside the cluster |
| Record automation | Creates and removes records when services change | ExternalDNS or similar, writing to an internal or public zone |

---

## Design Options

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Gateway built-in DNS | The router/gateway's own resolver, local records and DHCP integration | No extra device; lives in the foundation tier with routing and DHCP | Feature set varies by product; filtering and zone management may be limited. **Maintenance:** Covered by gateway updates and configuration backups; recheck local records after firmware changes or replacement | Small networks where household resolution and a few internal names are enough |
| [Pi-hole](https://docs.pi-hole.net/) | Filtering DNS resolver with a web interface and optional DHCP | Network-wide filtering and visibility of DNS queries; local records | A single instance becomes a household dependency; query logs are privacy-sensitive. **Maintenance:** Update the application and blocklists, back up configuration, review logging retention and keep a fallback resolver path | Network-wide filtering on an always-on, small dedicated device or foundation-tier host |
| [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome/wiki) | Filtering DNS resolver with encrypted-DNS support and a web interface | Filtering plus DNS-over-HTTPS/TLS upstreams and per-client settings | Same dependency and privacy considerations as other filtering resolvers. **Maintenance:** Update, back up configuration, review filters and logs, and keep a fallback | Filtering with encrypted upstream DNS |
| [Unbound](https://unbound.docs.nlnetlabs.nl/en/latest/) | Validating, recursive and caching resolver | Full recursion without relying on an upstream resolver; DNSSEC validation | Configuration is file-based and more technical; recursion adds outbound DNS traffic and troubleshooting. **Maintenance:** Update, monitor resolution failures and keep configuration versioned | Learning recursive resolution or reducing reliance on upstream resolvers, often behind a filtering resolver |
| [dnsmasq](https://thekelleys.org.uk/dnsmasq/doc.html) | Lightweight DNS forwarder and DHCP server | Small footprint; simple local records and DHCP integration | Limited to forwarding/caching and simple local data; configuration through files. **Maintenance:** Low; keep configuration versioned and updated with the host | Small embedded or router-based setups and lab exercises |
| [Technitium DNS Server](https://technitium.com/dns/) | Self-hosted authoritative and recursive DNS server with a web interface | Full zone management, recursion and filtering in one application | Another stateful service; must be placed in the right tier if household clients use it. **Maintenance:** Update, back up zones and settings, monitor resolution | A self-hosted DNS server with real zone management |
| [CoreDNS](https://coredns.io/manual/toc/) | Plugin-based DNS server, the default cluster DNS in Kubernetes | Native service discovery inside the cluster | Cluster-internal role; not normally the household resolver. **Maintenance:** Upgraded with the cluster; monitor and size it like other system workloads | Kubernetes service discovery; see [DNS inside Kubernetes](#dns-inside-kubernetes) |
| [ExternalDNS](https://kubernetes-sigs.github.io/external-dns/) | Controller that creates DNS records from Kubernetes Services and Ingresses | Records follow the workloads automatically; fewer stale or missing records | Needs credentials for the target DNS provider/server and becomes the owner of those records. **Maintenance:** Scope its credentials to the intended zone, restrict which records it may own and monitor failed updates | Automated internal or public records for platform-hosted services |
| Hosted public DNS provider | A provider serves the public zone of an owned domain | Reliable public authoritative DNS; APIs for automation and DNS-01 certificates | Account security is critical; public records are visible to everyone. **Maintenance:** Protect the account with MFA, scope API tokens, review records regularly and keep registrar renewal current | Owned domains, public services and DNS-01 certificate validation |

These options combine. A common, low-maintenance arrangement is gateway DNS for household clients with local records for a few lab names; a more elaborate one adds a filtering resolver on a small always-on device, an internal zone and ExternalDNS for platform services.

---

## Placement and Availability

Treat the resolver that DHCP hands to household clients as **foundation infrastructure**, regardless of which software provides it:

- Do not make a lab VM or Kubernetes workload the only resolver for household segments. If it is down, clients cannot resolve anything and the network appears broken.
- If a self-hosted resolver serves household clients, place it on an always-on device outside disposable lab compute, keep its configuration backed up, and decide on a fallback—a second resolver or the gateway—knowing that some clients use a secondary resolver unpredictably, which can bypass filtering.
- Lab segments can use a lab resolver; an outage then affects only the lab.
- Clients and browsers may bypass the local resolver through their own encrypted DNS settings. Decide whether that is acceptable instead of assuming all queries pass through the network's resolver.

Check placement with the [scheduling](./scheduling.md) design: a resolver on a host that is shut down at night breaks name resolution at night.

---

## Split-Horizon and Public DNS

Split-horizon DNS returns different answers for the same name depending on where the query comes from—for example, an internal address for clients at home and a public address, or nothing, for the internet. It lets one name work everywhere, but it adds a second zone to keep consistent, and troubleshooting depends on which resolver a client actually used.

Keep public DNS minimal. Every public record, and every publicly trusted certificate (see [certificate transparency](./certificates.md#certificate-transparency-and-internal-names)), can reveal something about the network behind it. Publish only names that must be reachable from the internet; keep internal hostnames in internal DNS. The actual domain, records and any internet-facing endpoints are sensitive values that belong in protected storage under the [repository guidelines](../repository-guidelines.md#secrets-and-sensitive-artifacts).

---

## DNS Inside Kubernetes

Kubernetes runs its own [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/), normally provided by CoreDNS. It resolves cluster-internal names and forwards everything else to the resolvers configured for the nodes. This is a different layer from household DNS: cluster names such as `service.namespace.svc.cluster.local` are not meant to be resolved by household clients.

To make a platform service reachable by name from home, publish a record for its stable external endpoint—the ingress or LoadBalancer address described in [reaching applications from home](../02-architecture/network.md#reaching-applications-from-home)—in the internal zone, either manually or with ExternalDNS. Never publish Pod or cluster-internal Service addresses. Make sure node resolvers point to something that is available whenever the cluster runs, or cluster workloads will fail to resolve external names.

---

## Implementation and Verification

Choose the naming scheme and resolver roles first, then configure the resolvers, the DHCP options that hand them out per segment, and the owner of each zone. Avoid two owners for the same records: if ExternalDNS manages a zone, do not also edit those records by hand.

Verify from each segment, not just from an administrator's workstation:

- Household clients resolve public and required internal names, **with lab compute turned off**.
- Lab clients resolve what they need without gaining unintended access to other segments' resolvers or management interfaces.
- Internal names are not resolvable from the internet unless deliberately published.
- Records disappear when services are retired, and TTLs are short enough for planned changes.
- Resolver configuration is backed up and can be restored, including local records and filtering settings.

The actual domain, zones, records and resolver placement belong in the private record.
