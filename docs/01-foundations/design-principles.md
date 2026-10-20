# Design Principles

> **Part 1 · Foundations** — [Part overview](./README.md) · Previous: [Requirements](./requirements.md) · Next: [Cost, Scope and Buying Decisions](./cost.md)

Requirements say *what* must be achieved; design principles say *how to choose* between the many designs that could achieve it. They are the recurring reasoning patterns of this book. Most chapters apply several of them, and when two reasonable options remain after the requirements are checked, the principles decide.

This chapter completes the storyline's second question: **what principles guide the design?** Each principle below explains the idea, why it matters at home scale and where the book applies it. The most important one for cost and availability—learning enterprise patterns without paying for enterprise availability—has its own detailed section.

---

## Table of Contents

- [The Principles at a Glance](#the-principles-at-a-glance)
- [Purpose Before Products](#purpose-before-products)
- [Learn Enterprise Patterns Without Enterprise Availability](#learn-enterprise-patterns-without-enterprise-availability)
- [Keep the Household Foundation Independent of the Lab](#keep-the-household-foundation-independent-of-the-lab)
- [Start Simple and Expand on Evidence](#start-simple-and-expand-on-evidence)
- [Recoverability Before Redundancy](#recoverability-before-redundancy)
- [Make Failure Domains Explicit](#make-failure-domains-explicit)
- [Least Privilege and Explicit Boundaries](#least-privilege-and-explicit-boundaries)
- [One Owner per Responsibility](#one-owner-per-responsibility)
- [Automate the Repeatable, Document the Exceptions](#automate-the-repeatable-document-the-exceptions)
- [Spend Maintenance Effort Where the Learning Is](#spend-maintenance-effort-where-the-learning-is)
- [Evidence Over Assumption](#evidence-over-assumption)
- [Publish Lessons, Not Internals](#publish-lessons-not-internals)
- [When Principles Conflict](#when-principles-conflict)

---

## The Principles at a Glance

| Principle | In one sentence | Applied most visibly in |
|---|---|---|
| [Purpose before products](#purpose-before-products) | Buy and install for a real, continuing need, not because something is impressive | [Cost](./cost.md), [Compute](../02-architecture/compute.md) |
| [Enterprise patterns without enterprise availability](#learn-enterprise-patterns-without-enterprise-availability) | Learn the concept at affordable scale and state which failures are accepted | [Network](../02-architecture/network.md#availability-backups-and-recovery), [Platform](../02-architecture/platform.md) |
| [Household foundation independent of the lab](#keep-the-household-foundation-independent-of-the-lab) | Experiments must not take household connectivity down with them | [Network](../02-architecture/network.md#design-options), [DNS](../03-platform-services/dns.md) |
| [Start simple, expand on evidence](#start-simple-and-expand-on-evidence) | Add hardware, platforms and services after measurements or real limits show a gap | [Storage](../02-architecture/storage.md), [Cost](./cost.md#a-practical-expansion-rule) |
| [Recoverability before redundancy](#recoverability-before-redundancy) | Tested backups and restore paths come before highly available pairs | [Backups](../06-operations/backups.md), [Disaster Recovery](../06-operations/disaster-recovery.md) |
| [Explicit failure domains](#make-failure-domains-explicit) | Name the shared host, disk, power and gateway dependencies instead of hiding them | [Architecture overview](../02-architecture/README.md), [Compute](../02-architecture/compute.md#quorum-and-failure-domains) |
| [Least privilege and explicit boundaries](#least-privilege-and-explicit-boundaries) | Deny by default, allow specific paths, protect management | [Network segmentation](../02-architecture/network.md#segmentation-and-access), [Identity](../03-platform-services/identity.md) |
| [One owner per responsibility](#one-owner-per-responsibility) | Every property, value and explanation has exactly one source of truth | [Automation](../04-automation/README.md), [Repository guidelines](../repository-guidelines.md#content-ownership) |
| [Automate the repeatable](#automate-the-repeatable-document-the-exceptions) | Declare what is repeated or recovery-critical; document deliberate manual steps | [Infrastructure as Code](../04-automation/infrastructure-as-code.md), [Configuration Management](../04-automation/configuration-management.md) |
| [Maintenance effort where the learning is](#spend-maintenance-effort-where-the-learning-is) | Keep essential infrastructure boring so time goes to the interesting layers | [Network equipment](../02-architecture/network.md#should-you-build-your-own-firewall-or-buy-an-appliance), [Maintenance](../06-operations/maintenance.md) |
| [Evidence over assumption](#evidence-over-assumption) | A design or configuration is not proof that something works | Every chapter's verification section, [Observability](../06-operations/observability.md) |
| [Publish lessons, not internals](#publish-lessons-not-internals) | Share reasoning and generic patterns; keep the actual environment private | [Repository guidelines](../repository-guidelines.md#publication-boundary) |

---

## Purpose Before Products

Every component should exist because a [requirement](./requirements.md) or a bounded learning goal needs it. Products, tutorials and other people's racks are sources of ideas, not requirements. A capable device that does not serve a continuing purpose costs money, electricity, space and maintenance time without returning value.

Apply the [purchase test](./cost.md#the-most-important-constraint-buy-for-a-real-need) before buying, and prefer a temporary experiment on existing equipment when the goal is to understand a concept once. This principle is deliberately first: most other principles become much easier to follow when the homelab is not carrying equipment and services that nobody needs.

Purpose also sets an attention and responsibility budget. Follow [relaxation and enjoyment over productivity and pressure](./goals-and-scope.md#relaxation-and-enjoyment-over-productivity-and-pressure): household independence, bounded maintenance and focused lab sessions protect the freedom to experiment without making the hobby a second job.

---

## Learn Enterprise Patterns Without Enterprise Availability

**A homelab does not need an enterprise production-availability target to be useful.** Redundant gateways, independent physical hosts, switches, power paths and ISP links cost money to buy, power, support and replace. They also add configuration, monitoring, upgrades and failover tests. Most learning goals do not justify all of that recurring cost. Choose an acceptable outage and recovery process rather than copying an enterprise equipment list.

Learn the pattern at the scale you can afford. For example, several Kubernetes VMs on one hypervisor can teach scheduling, control-plane quorum, rolling upgrades and recovery from a guest failure. They **cannot** survive failure of their shared physical host, storage or power supply. Testing guest failure is useful evidence about that exercise, not proof of physical high availability. Similarly, VLANs can teach trust boundaries on shared hardware without making that hardware redundant.

State the distinction explicitly in a design: "For a production service with a strict availability requirement, distribute these roles across independent failure domains; for this lab, share a host and accept host-level downtime to reduce cost." Enterprise requirements vary too: spend on redundancy because an agreed recovery target demands it, not because every enterprise must have the same topology.

Do not confuse accepted downtime with accepted data loss or weak security. Protect important data, restrict access and keep recovery instructions and backups accessible independently of the failed system. Household internet may deserve a more stable foundation than disposable workloads without requiring an automatic-failover gateway pair. The [network availability and backup guidance](../02-architecture/network.md#availability-backups-and-recovery) applies this trade-off to the shared home-and-lab network.

---

## Keep the Household Foundation Independent of the Lab

Household connectivity—internet access, Wi-Fi, address assignment and essential name resolution—belongs to the [foundation tier](./requirements.md#service-tiers-how-much-does-it-matter). It should keep working when the hypervisor is rebooted, the Kubernetes cluster is rebuilt or an experiment misbehaves. In practice, that means the gateway, DHCP and household DNS do not run on disposable lab compute, management remains reachable without the lab, and lab experiments are contained by [segmentation](../02-architecture/network.md#segmentation-and-access).

Independence is a dependency rule, not a hardware count. One physical gateway with VLANs can satisfy it; a dedicated lab router VM can violate it if household traffic starts depending on that VM. Check the rule by turning lab compute off and verifying household connectivity, as described in the [network implementation checks](../02-architecture/network.md#implementation-and-verification).

---

## Start Simple and Expand on Evidence

Begin with the smallest design that meets the current requirements and can teach the planned concepts: often one host, local SSD storage, an independent backup drive and one well-segmented network. Expand when measurements, an actual limitation or a specific, sustained learning objective show what is missing—not because a more elaborate option exists.

Each expansion should record the benefit, the unique evidence it enables, new failure modes, full purchase and running cost, expected period of use and the retirement plan. The [practical expansion rule](./cost.md#a-practical-expansion-rule) applies this to purchases; the [storage chapter](../02-architecture/storage.md#match-the-design-to-the-need) shows the same reasoning for NAS and distributed storage.

---

## Recoverability Before Redundancy

Redundancy reduces how often an outage happens; recoverability determines whether an outage becomes a loss. For a homelab, a tested backup and a documented, rehearsed restore usually provide more value per euro and per hour than a redundant pair. Redundancy, snapshots and replication are not backups: they can replicate deletion, corruption or misconfiguration just as faithfully as good data.

Design the recovery path first—what is backed up, where independent copies live, which credentials and instructions are needed during an outage, and how long a restore takes—then add redundancy only where an agreed [RTO](./requirements.md#availability-and-recovery-requirements) cannot be met otherwise. [Backups](../06-operations/backups.md) and [Disaster Recovery](../06-operations/disaster-recovery.md) apply this principle in detail.

---

## Make Failure Domains Explicit

A failure domain is everything that fails together: VMs on one host, volumes on one disk, devices on one power strip, segments on one gateway. Homelabs share failure domains deliberately to save money; the mistake is to hide that sharing behind a diagram that looks redundant. Three Kubernetes VMs on one hypervisor are one physical failure domain, even though they teach three-node behavior.

State accepted single points of failure in each design, together with the recovery path that compensates for them. The [architecture overview](../02-architecture/README.md#dependencies-and-failure-domains) collects them at the system level; the [compute chapter](../02-architecture/compute.md#quorum-and-failure-domains) explains quorum and the limits of virtual redundancy.

---

## Least Privilege and Explicit Boundaries

Allow only the paths, permissions and identities that a requirement needs, and deny the rest by default. Management interfaces are the most sensitive part of a homelab: they should be reachable only from approved administrator clients, not from every household device and never publicly by default. Exceptions—an IoT controller path, a lab application reachable from home—are recorded with their reason and owner, and removed when no longer needed.

The same principle applies beyond the network: separate administrator and automation accounts, scoped API tokens, protected secret storage and short-lived experiment credentials. See [network segmentation](../02-architecture/network.md#segmentation-and-access), [Identity and Access](../03-platform-services/identity.md) and [Secrets](../03-platform-services/secrets.md).

---

## One Owner per Responsibility

Every property should have exactly one source of truth. If Terraform creates a VM, Ansible should not also resize its disk; if GitOps reconciles a workload, nobody should edit it by hand in the cluster; if a selected VLAN value is recorded in a design section, setup sections link to it instead of repeating it. Two owners for one property produce drift and confusing failures.

This principle also shapes the documentation itself: DevHub owns general theory, this public book owns homelab applications and options, and the private record owns the actual environment, as described in the [content ownership rules](../repository-guidelines.md#content-ownership). The [automation part](../04-automation/README.md#keep-ownership-clear) applies it to tools.

---

## Automate the Repeatable, Document the Exceptions

Automate the lifecycle that is repeated, error-prone or critical during recovery: resource creation, host configuration, workload delivery, validation and backup scheduling. A reviewed declaration is easier to reproduce than remembered clicks, and replacing a failed host should not require reconstructing its setup from memory.

Automation is a means, not a goal in itself. Physical installation, unsupported appliance settings and one-time bootstrap steps may remain manual when automating them would add more fragility than it removes. Document those exceptions, back them up and revisit them when they become repeated. Code then describes *how*; documentation describes *why*, *what* and the relationships, without duplicating every task the code already encodes. See the [automation part](../04-automation/README.md).

---

## Spend Maintenance Effort Where the Learning Is

Maintenance time is limited. Spend it on the layers that serve the learning goals, and keep the essential foundation predictable and low-maintenance. For example, an integrated network appliance can leave more time for hypervisors, VMs and Kubernetes than a self-assembled router—while still teaching VLANs, routing and firewall policy. Someone whose goal is firewall engineering may reasonably choose the opposite.

Low maintenance is not maintenance-free: supported updates, backups and verification remain. The point is to choose consciously which integration work to own. The [firewall build-versus-buy discussion](../02-architecture/network.md#should-you-build-your-own-firewall-or-buy-an-appliance) and the [maintenance chapter](../06-operations/maintenance.md) apply this principle.

---

## Evidence Over Assumption

A design, a configuration file or a successful installation is not evidence that something works. Verify behavior: test denied paths as well as allowed paths, restore backups into an isolated target, turn lab compute off to check independence, measure power at the wall rather than reading a specification. Record what was actually tested and what was not.

The same honesty applies to documentation status: distinguish **chosen**, **planned**, **installed** and **verified**. Every architecture chapter ends with implementation and verification considerations, and [observability](../06-operations/observability.md) provides ongoing evidence after the initial checks.

---

## Publish Lessons, Not Internals

This public book shares reasoning, options, trade-offs and generic patterns. It never publishes the actual household topology, addressing, inventory, firewall exceptions, endpoints or credentials—not because those details are embarrassing, but because they can help someone target the network and they are not needed to tell the story. The private repository holds the actual implementation.

This principle is defined precisely by the [publication boundary](../repository-guidelines.md#publication-boundary) and explained in the [README](../../README.md#why-two-homelab-repositories). It also explains why examples throughout the book are synthetic and clearly labeled.

---

## When Principles Conflict

Principles sometimes pull in different directions. Learning a self-built router conflicts with keeping the foundation low-maintenance; automating everything conflicts with avoiding fragile automation for one-time steps; distributed storage is a valid learning goal but conflicts with starting simple.

Resolve conflicts with the [requirements and their priorities](./requirements.md#prioritizing-requirements), not with personal taste alone. A common resolution is to **separate the tiers**: keep the foundation conservative and run the ambitious variant as a bounded experiment in the lab tier—for example, a disposable router VM downstream of a stable household gateway. Record the trade-off in the relevant chapter's decision, including what was accepted and when it should be revisited.
