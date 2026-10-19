# Part 2 · Architecture

> **Layer: Architecture — what are we building?** · [Book overview](../README.md) · Previous part: [Part 1 · Foundations](../01-foundations/README.md) · Next part: [Part 3 · Platform Services](../03-platform-services/README.md)

The [foundations](../01-foundations/README.md) defined why the homelab exists, what it must achieve and what it may cost. This part answers the storyline's third question: **what architecture solves those requirements?** It describes the building blocks of a home-and-lab environment from the ground up—the physical environment, network, compute, storage and workload platform—and how they depend on each other.

Each chapter covers one building block completely. It starts with that block's purpose and requirements, compares the design options, zooms into equipment and configuration considerations, and ends with implementation and verification. There is no separate high-level and low-level design document for the same subject: the high-level view is the start of the chapter, and the low-level detail is its later sections. See the [documentation design](../repository-guidelines.md#why-not-high-level-and-low-level-design-folders) for why.

> **Scope:** These chapters apply shared principles to home-and-lab infrastructure. Read the [repository relationship](../../README.md#how-devhub-and-the-homelab-repositories-fit-together) for what belongs in DevHub, this public book and the private implementation record.

---

## Table of Contents

- [The Architecture at a Glance](#the-architecture-at-a-glance)
- [Chapters in This Part](#chapters-in-this-part)
- [Physical and Network Foundation](#physical-and-network-foundation)
- [Compute and Storage Together](#compute-and-storage-together)
- [The Workload Platform](#the-workload-platform)
- [Dependencies and Failure Domains](#dependencies-and-failure-domains)
- [Workload Placement](#workload-placement)
- [What a Good Architecture Output Looks Like](#what-a-good-architecture-output-looks-like)

---

## The Architecture at a Glance

The homelab is a stack of layers. Each layer depends on the ones below it, and each chapter of this part zooms into one of them; the following parts add the capabilities and services that run on top.

```text
                         Homelab
                            |
      +---------------------+----------------------+
      |                                            |
  Services (Part 5)                     Operations (Part 6)
      |                                 observe, back up, recover
  Platform services (Part 3)            and maintain every layer
  DNS, certificates, identity,
  secrets, scheduling
      |
  Platform  ------------------- platform.md
  containers / Kubernetes
      |
  Compute  +  Storage --------- compute.md, storage.md
  hosts, hypervisor, disks
      |
  Network --------------------- network.md
  gateway, segments, switching, Wi-Fi
      |
  Physical -------------------- physical.md
  location, power, cooling, cabling
```

Automation ([Part 4](../04-automation/README.md)) is not a separate layer in this picture: it is *how* every layer is created and configured repeatably. The architecture chapters describe what is built and why; the automation part describes how that is realized as code.

---

## Chapters in This Part

Read the chapters in order, from the ground up:

| # | Chapter | What it decides | Key cross-links |
|---|---|---|---|
| 1 | [Physical](./physical.md) | Where equipment goes; meter cabinets, ventilation, racks and rack units, power strips, UPS and energy monitoring | [Constraints](../01-foundations/requirements.md#constraints), [Cost](../01-foundations/cost.md) |
| 2 | [Network](./network.md) | Gateway arrangement, ISP handoff, segmentation and trust roles, networks inside the lab, router/switch/AP choices, Wi-Fi, gateway recovery | [DevHub network design](https://github.com/CollinPoetoehena/DevHub/blob/main/reference/network/Network_Design.md), [DNS](../03-platform-services/dns.md) |
| 3 | [Compute](./compute.md) | Host roles and capacity, buying and sizing nodes, electricity and consolidation, quorum and failure domains, virtualization and guest operating systems | [Scheduling](../03-platform-services/scheduling.md), [Infrastructure as Code](../04-automation/infrastructure-as-code.md) |
| 4 | [Storage](./storage.md) | Storage roles, storage platforms (local, ZFS, NAS, iSCSI, Ceph, CSI), matching the design to the need | [Backups](../06-operations/backups.md), [Disaster Recovery](../06-operations/disaster-recovery.md) |
| 5 | [Platform](./platform.md) | Compose versus Kubernetes and alternatives, Kubernetes node lifecycle, cluster networking and persistence | [Deployment](../04-automation/deployment.md), [Workloads](../05-services/workloads.md) |

---

## Physical and Network Foundation

The [physical chapter](./physical.md) covers safe locations, meter cabinets, ventilation, racks, rack units, power strips, optional UPSes and whole-lab energy monitoring. Actual placements and purchased models belong in the private homelab.

The [network chapter](./network.md) applies [DevHub's network design principles](https://github.com/CollinPoetoehena/DevHub/blob/main/reference/network/Network_Design.md) to household/lab boundaries, gateway/switch choices, Wi-Fi coverage and backhaul, and safe implementation considerations. Wi-Fi is part of that chapter, not a second independent routing stack.

Choose which connectivity must remain available independently of experiments, then locate routing, switching, wireless access and policy enforcement. A small network can collapse several logical layers into one switch while still having clear ownership. Segmentation, equipment capabilities and recovery access should be decided together; an attractive gateway cannot implement a policy its switch/AP paths do not support.

---

## Compute and Storage Together

Compute hosts and storage are closely related infrastructure decisions: most systems use local storage inside the compute host, while shared or distributed storage can add separate machines, network dependencies and power. Consider those responsibilities and failure domains together, while keeping the detailed design in two focused chapters:

- [Compute](./compute.md) covers host roles, capacity, form factors, virtualization and whole-system power. It explains how to size physical hosts before assigning VMs or cluster nodes. A hypervisor host cluster belongs in the compute discussion; the [container-orchestration cluster](./platform.md) is a separate platform concern.
- [Storage](./storage.md) covers the choice between local storage in a compute host, external backup devices, NAS/shared storage and distributed storage. The local SSD is commonly part of the compute system, which is why these chapters are read together; their requirements and failure domains should still be considered explicitly. For backup and recovery options, continue to [Backups](../06-operations/backups.md).

Physical capacity and failure domains constrain everything above them. Decide reuse versus purchases, consolidated versus independent hosts, and bare metal versus virtualization before allocating logical nodes. Reserve maintenance/recovery capacity and count whole-system power. A hypervisor cluster is a management/availability choice with its own quorum/storage obligations, not an automatic consequence of owning two PCs. Review the central [cost and scope guidance](../01-foundations/cost.md) before buying equipment.

---

## The Workload Platform

The [platform chapter](./platform.md) covers Kubernetes and alternatives, node lifecycle and cluster networking. This is the workload cluster, **not a hypervisor host cluster**.

Decide whether workloads need single-host containers, a scheduler or a broader application platform. Then choose node OS/bootstrap, network/policy and persistence integrations as one compatible lifecycle. Kubernetes objects alone do not install a host OS, replace a gateway or make data durable. Platform requirements can feed back into the lower layers: a highly available control plane needs independent failure domains in compute, and persistent volumes need a storage design that matches their recovery requirements. The order of the chapters is a reasoning path, not an inflexible waterfall.

---

## Dependencies and Failure Domains

Check which services must survive lab shutdown, which links carry administration/workloads, and which failure domains are shared. A homelab shares failure domains deliberately to save money; the architecture should make that sharing visible instead of implying redundancy that does not exist.

| Shared dependency | What fails together | Typical accepted compensation |
|---|---|---|
| Gateway | Internet access, routing between segments, often DHCP/DNS for every segment | Protected configuration backups, a documented [gateway recovery option](./network.md#availability-backups-and-recovery) |
| Switch and power strip | Every wired device and PoE-powered AP behind them | Labeled cabling, spare capacity or a compatible spare where outage impact justifies it |
| Hypervisor host | Every VM and container on that host, including several "redundant" Kubernetes nodes | Accepted host-level downtime, guest backups, rebuild from code; see [quorum and failure domains](./compute.md#quorum-and-failure-domains) |
| Local disk or pool | Every VM disk and volume on it | Health monitoring and independent [backups](../06-operations/backups.md) |
| Site and household power | Everything in the home | Off-site copies of important data, optional UPS for short interruptions, [recovery planning](../06-operations/disaster-recovery.md) |
| Cloud/vendor account | Controller access, hosted backups or DNS | Local management path, offline recovery material, documented account recovery |

Use these constraints to evaluate the [platform services](../03-platform-services/README.md), [services](../05-services/README.md) and [operations](../06-operations/README.md); revisit the architecture if they exceed its resources or availability. The [design principle](../01-foundations/design-principles.md#make-failure-domains-explicit) behind this table is to name accepted single points of failure together with their recovery path.

---

## Workload Placement

Placement decides which layer and which failure domain each component lives in. A few rules keep the architecture understandable:

- **Foundation services stay outside lab compute.** Household routing, DHCP, essential DNS and Wi-Fi management should not depend on a VM or cluster that experiments can break. See [keep the household foundation independent of the lab](../01-foundations/design-principles.md#keep-the-household-foundation-independent-of-the-lab).
- **Management and recovery tools stay reachable when the workload platform fails.** A management VM or workstation, the hypervisor interface and recovery credentials should not depend on Kubernetes being healthy.
- **Observability placement matches the failure you need to see.** Monitoring inside the cluster cannot report a complete cluster outage; see [observability placement](../06-operations/observability.md#design-signals-and-placement).
- **Backups live outside the failure domain they protect.** Another directory on the same disk or a replica on the same host is not an independent copy.
- **Experiments are contained.** Disposable workloads run in the lab segment and lab tier, with no unnoticed dependencies from household or platform components.

These are placement *rules*, not a map of an actual installation. The private record contains the actual placement of each component.

---

## What a Good Architecture Output Looks Like

A useful architecture output is a role-based view of dependencies, capacity assumptions, trust boundaries and accepted single points of failure. It does not need every DHCP reservation or application manifest: those details are implementation, recorded privately and encoded in configuration. The public book explains how to reason about the architecture; the private record supplies the chosen equipment and actual mappings. For the platform and the layers above it, a good output also names clear component responsibilities, compatibility constraints, update/recovery workflows and validation criteria. Actual releases, resource allocations, endpoints and results stay in the matching private chapter; public documentation shows the reasoning and generic implementation approach. Use the [project-structure guide](../04-automation/project-structure.md) to keep implementation ownership aligned with that design, and continue to [Part 3 · Platform Services](../03-platform-services/README.md) for the capabilities that make the platform usable.
