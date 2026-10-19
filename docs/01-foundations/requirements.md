# Requirements

> **Part 1 · Foundations** — [Part overview](./README.md) · Previous: [Goals and Scope](./goals-and-scope.md) · Next: [Design Principles](./design-principles.md)

Goals say *why* the homelab exists; requirements say *what it must achieve* for those goals to be met. They are the bridge between the personal motivation in [Goals and Scope](./goals-and-scope.md) and the [architecture](../02-architecture/README.md): every network boundary, host, storage choice and service later in this book should be traceable to at least one requirement, or be an explicitly bounded experiment.

This chapter answers the second half of the storyline's second question: **what requirements guide the design?** It explains how to write requirements as outcomes, which areas a homelab usually needs requirements for, how to classify components by how much they matter, and how to keep requirements traceable to design and verification without turning a hobby into a requirements-engineering project.

---

## Table of Contents

- [Write Requirements as Outcomes](#write-requirements-as-outcomes)
- [Requirement Areas](#requirement-areas)
- [Service Tiers: How Much Does It Matter?](#service-tiers-how-much-does-it-matter)
- [Availability and Recovery Requirements](#availability-and-recovery-requirements)
- [Constraints](#constraints)
- [Prioritizing Requirements](#prioritizing-requirements)
- [Traceability: From Requirement to Verification](#traceability-from-requirement-to-verification)
- [Reviewing Requirements](#reviewing-requirements)

---

## Write Requirements as Outcomes

A requirement describes an observable result, not a product or a topology. "Restore important files within a day after a disk failure" is a requirement; "buy a NAS" is one possible solution to it, and often not the cheapest one. Writing outcomes keeps several solutions open, makes it possible to check whether the requirement is met, and avoids purchasing equipment that answers a question nobody asked.

A useful requirement has four parts:

1. **Outcome:** what must be true, from the point of view of a person or a dependent system.
2. **Reason:** which goal it supports and what happens if it is not met.
3. **Evidence:** how you would check it—a test, a measurement or a rehearsal—rather than assuming it from configuration.
4. **Priority:** whether it is essential, useful or a deferred idea (see [prioritizing requirements](#prioritizing-requirements)).

Tools appear in requirements only when learning that tool is itself the goal—for example, "practise rolling Kubernetes upgrades." Even then, write down what the exercise should demonstrate and when it is finished, so it does not quietly become a permanent dependency.

---

## Requirement Areas

The table lists the areas a small home-and-lab environment usually needs to consider. Not every homelab needs a requirement in every row; an empty row should be a conscious "not needed" rather than an oversight.

| Area | Example outcome statement | Questions to answer | Where the book answers it |
|---|---|---|---|
| Household connectivity | "Internet, Wi-Fi and name resolution keep working while lab compute is off." | Who depends on the connection, for what, and how long can it be down? | [Network](../02-architecture/network.md), [DNS](../03-platform-services/dns.md) |
| Learning | "I can rebuild a Kubernetes cluster from versioned configuration and explain each step." | Which skills or concepts should the lab teach, and what evidence shows they were learned? | [Platform](../02-architecture/platform.md), [Automation](../04-automation/README.md) |
| Useful services | "A household photo service is reachable from home devices and survives a host reboot." | Which services provide lasting value, and who uses them? | [Services](../05-services/README.md) |
| Data protection | "Losing a disk loses at most one day of household photos." | Which data is unique, how valuable is it, and how much loss is acceptable? | [Storage](../02-architecture/storage.md), [Backups](../06-operations/backups.md) |
| Availability | "Experiments may be down for days; the household gateway may be down for at most an evening." | Which components need to be available, when, and what happens when they are not? | [Service tiers](#service-tiers-how-much-does-it-matter), [Disaster Recovery](../06-operations/disaster-recovery.md) |
| Security and access | "Only approved administrator devices can reach management interfaces." | Who administers what, from where, and which paths must be denied? | [Network segmentation](../02-architecture/network.md#segmentation-and-access), [Identity](../03-platform-services/identity.md), [Secrets](../03-platform-services/secrets.md) |
| Remote access | "Administration from outside the home is possible without exposing management interfaces publicly." | Is remote access needed at all? For whom, and for which services? | [ISP handoff and remote access](../02-architecture/network.md#isp-handoff-nat-and-remote-access) |
| Capacity and performance | "The host runs the planned VMs with memory headroom for maintenance." | What are the active CPU, memory, storage and network needs, measured rather than guessed? | [Compute](../02-architecture/compute.md), [Storage](../02-architecture/storage.md) |
| Visibility | "I am notified when a backup fails or a disk is close to full." | Which failures would otherwise go unnoticed, and who acts on an alert? | [Observability](../06-operations/observability.md) |
| Maintainability | "Routine updates take less than an hour per month." | How much time is realistically available, and which work is acceptable to own? | [Maintenance](../06-operations/maintenance.md), [Design Principles](./design-principles.md) |
| Budget and running cost | "The always-on equipment stays below an agreed monthly electricity cost." | What may be spent on purchase, electricity and replacement over the expected lifetime? | [Cost](./cost.md), [Scheduling](../03-platform-services/scheduling.md) |
| Physical environment | "Equipment fits a ventilated shelf, stays quiet in a living area and is safe to service." | Where can equipment go, and what heat, noise and electrical limits apply? | [Physical](../02-architecture/physical.md) |

These areas overlap deliberately: a single outcome such as "household internet survives lab maintenance" touches connectivity, availability and physical power at once. Record it once and link it from each chapter that implements part of it, rather than writing three slightly different copies.

---

## Service Tiers: How Much Does It Matter?

Requirements become much easier to apply when components are grouped by how much the household or the owner depends on them. A tier sets the default expectations for availability, data protection, change caution and maintenance effort; individual requirements can then tighten or relax those defaults where needed.

| Tier | Typical components | Availability expectation | Data expectation | Change and maintenance expectations |
|---|---|---|---|---|
| Foundation | Modem/ONT handoff, gateway, essential switching, Wi-Fi, household name resolution | Highest in the homelab; outages are noticed by everyone. Recovery is planned, not improvised | Configuration exports and recovery notes protected and available offline | Changes are staged, scheduled and reversible; updates are reviewed; independent of lab compute |
| Household service | Self-hosted services that other people rely on, such as a shared photo or document service | Agreed with the people using it; short planned outages are usually acceptable | Unique data has independent, tested backups with an agreed loss window | Patching, monitoring, access control and restore tests are part of owning it |
| Lab platform | Hypervisor, management VMs, Kubernetes cluster, monitoring, automation controllers | Outages are acceptable but inconvenient; rebuild should be possible from code and documentation | Platform state recoverable or reproducible; important configuration is versioned | Upgrades follow supported paths; rebuilds are rehearsed; failures must not affect the foundation tier |
| Experiment | Short-lived workloads, evaluations, deliberately broken environments | None; may be deleted at any time | Disposable; no unique data | Created, observed, then removed or deliberately promoted to a higher tier with its obligations |

The tier boundary is the most important line in this table. Promoting an experiment to a household service is a design decision: it adds patching, access control, [observability](../06-operations/observability.md), [backups](../06-operations/backups.md) and [recovery](../06-operations/disaster-recovery.md) obligations. Likewise, a foundation component should never silently depend on a lab-platform component—for example, household DNS that only works while a VM is running. The [workload chapter](../05-services/workloads.md#workload-design) applies these tiers to individual services.

---

## Availability and Recovery Requirements

Two measures make availability and data-protection requirements concrete:

- **Recovery Point Objective (RPO):** the maximum acceptable data-loss window. An RPO of one day means a recovery may lose at most the last day of changes, which in turn means a successful, independent backup must exist at least daily.
- **Recovery Time Objective (RTO):** the maximum acceptable time to restore service after a failure. An RTO of an evening means the replacement hardware, backups, credentials and instructions must all be reachable and usable within that time.

Set RPO and RTO per tier or per important service, not once for the whole homelab. Disposable experiments may have no RPO at all, while a household photo library may need a short RPO but tolerate a longer RTO. Agree the foundation tier's RTO with the people who depend on the connection: waiting for a replacement gateway may be acceptable, or remote work may justify a cold spare or alternate uplink.

These objectives drive concrete choices later in the book: backup frequency and media in [Backups](../06-operations/backups.md), spare hardware and recovery order in [Disaster Recovery](../06-operations/disaster-recovery.md), and gateway recovery options in [Network availability](../02-architecture/network.md#availability-backups-and-recovery). Accepting downtime is a legitimate requirement for a homelab—see [Learn Enterprise Patterns Without Enterprise Availability](./design-principles.md#learn-enterprise-patterns-without-enterprise-availability)—but accepted downtime is not the same as accepted data loss or weak security.

---

## Constraints

Constraints are requirements that limit the solution space rather than describe an outcome. They are often the most decisive inputs for a homelab and deserve the same explicit recording:

| Constraint | Why it matters | Typical effect on the design |
|---|---|---|
| Purchase budget | Equipment money is finite and easy to overspend on impressive but unused hardware | Reuse existing equipment, prototype in VMs, buy only after the [purchase test](./cost.md#the-most-important-constraint-buy-for-a-real-need) |
| Electricity and running cost | Always-on equipment costs money every hour, for years | Fewer suitably sized hosts, [scheduled or on-demand compute](../03-platform-services/scheduling.md), measured consumption |
| Space, heat and noise | Living spaces and meter cabinets have limited room, ventilation and tolerance for fan noise | Compact hosts, ventilated shelves instead of racks, [placement choices](../02-architecture/physical.md) |
| Electrical and building safety | Utility areas and wiring have safety rules that the homelab must respect | No obstruction of meters/breakers, rated power accessories, no improvised electrical changes |
| Household acceptance | Other people share the network, the space and the consequences of outages | Stable foundation tier, scheduled maintenance windows, deliberate exceptions only |
| Available time and skills | Maintenance time competes with learning time and the rest of life | Low-maintenance foundation, automation where it saves real work, fewer moving parts |
| Provider and equipment limits | ISP handoff, CGNAT, device firmware and model features constrain what is possible | Verify capabilities before designing around them; see [ISP handoff](../02-architecture/network.md#isp-handoff-nat-and-remote-access) |
| Publication boundary | The public documentation must not reveal the actual household network | Generic patterns in public, actual values privately, as defined in the [repository guidelines](../repository-guidelines.md#publication-boundary) |

---

## Prioritizing Requirements

A simple three-level priority is enough for a personal project:

- **Must:** without it, the homelab fails its purpose or harms the household—for example, household connectivity independent of experiments, or backups for unique data.
- **Should:** clearly valuable and planned, but the homelab is still useful without it for now—for example, alerting on failed backups, or private remote administration.
- **Could / deferred:** interesting ideas to revisit when a measured need or sustained learning goal appears—for example, distributed storage or a highly available control plane.

Deferred ideas are not rejected; they are parked with the evidence that would justify them. This keeps enthusiasm for advanced topics without letting it drive purchases ahead of real needs, which is the central concern of the [cost chapter](./cost.md).

---

## Traceability: From Requirement to Verification

Each requirement should be traceable through the rest of the book. The chain is:

```text
Requirement -> design principle(s) -> architecture decision -> implementation (code/config) -> verification -> recovery
```

For example, the synthetic requirement "household internet keeps working while lab compute is off" leads to the principle [keep the household foundation independent of the lab](./design-principles.md#keep-the-household-foundation-independent-of-the-lab), to a [network design option](../02-architecture/network.md#design-options) that keeps gateway, DHCP and DNS outside lab compute, to the configuration of that gateway, to a verification step that turns lab compute off and checks connectivity, and to a [recovery plan](../06-operations/disaster-recovery.md) for the gateway itself. If any link in that chain is missing, either the requirement is not really met or the design contains a component without a reason.

Keep the traceability lightweight. A short requirement list with links from the relevant chapters is enough; a formal requirements matrix with identifiers is optional. What matters is that a reader—or the owner, a year later—can answer "why does this exist?" and "how do I know it works?" for every important component.

---

## Reviewing Requirements

Requirements change when circumstances change: new household dependencies, a move, a changed budget, new hardware, or a learning goal that has been achieved. Review them when:

- A component is promoted between [service tiers](#service-tiers-how-much-does-it-matter).
- A purchase is being considered.
- An incident shows that an assumed requirement was wrong, such as an outage that turned out to matter more than expected.
- A periodic [maintenance review](../06-operations/maintenance.md) finds components that no longer serve any requirement and can be retired.

The public book describes the method and generic examples. The actual requirement list, its priorities and the household context belong in the private record, consistent with the [repository guidelines](../repository-guidelines.md#content-ownership).
