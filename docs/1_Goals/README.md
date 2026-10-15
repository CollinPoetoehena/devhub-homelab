# Goals and Constraints

Choose the purpose before buying equipment or installing tools.

---

## Table of Contents

- [Design the Purpose](#design-the-purpose)
- [Keep the Scope Sustainable](#keep-the-scope-sustainable)
- [Learn Enterprise Patterns Without Enterprise Availability](#learn-enterprise-patterns-without-enterprise-availability)
- [From Goals to Categories](#from-goals-to-categories)

---

## Design the Purpose

A homelab may be a disposable learning environment, a development sandbox, a self-hosted household platform, or a mixture. Learning Kubernetes on demand is different from hosting the only copy of household files. Define what must remain available when experiments fail.

Record learning topics, useful services, data importance, acceptable downtime/data loss, budget, electricity, space/noise, cabling and maintenance time. Decide whether remote access is actually needed and who administers the system.

| Option | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Disposable learning | Short-lived isolated exercises | Fast rebuilds, bounded scope and useful diagnostics | Remove/promote deliberately; not every exercise needs permanent hardware or HA. **Maintenance:** Little permanent upkeep if exercises are removed; refresh rebuild inputs and clean up resources/credentials after each exercise. | Explore tools/protocols without household dependencies |
| Development/testing | Reproducible environments for application integration | Realistic test paths and repeatable development | Test credentials/data must not share production trust; cleanup and capacity still matter. **Maintenance:** Keep test images, dependencies and fixtures current; reset environments and verify that stale credentials or data do not persist. | Build/test applications and infrastructure changes |
| Household services | Self-hosted useful services with ongoing users/data | Practical value and operational learning | Predictable maintenance, backups and usable recovery; household users may not accept lab-style downtime. **Maintenance:** Recurring patching, backup checks and restore exercises are essential; schedule changes around users and retain usable recovery instructions. | You can own the service's reliability/data lifecycle |
| Enterprise-platform practice | Targeted management, upgrade and failure exercises | Experience with specific enterprise concepts | Cost/resources and operational scope; copying enterprise topology is unnecessary for unrelated goals. **Maintenance:** Budget time for supported release paths and management components; retire exercises that no longer serve the learning goal. | Enterprise learning itself is the requirement |

Write requirements as outcomes: "restore important data within the accepted time" or "keep household connectivity working during compute maintenance," not "buy three nodes." Identify must-haves, useful learning goals and deferred ideas. A tool is a means of achieving those outcomes, not the goal unless learning that tool is explicitly the exercise.

---

## Keep the Scope Sustainable

Prefer a small stable foundation and disposable experiments. One host can teach many concepts; extra machines are justified by capacity, physical-failure practice or actual availability requirements, not an enterprise-rack aesthetic.

Self-hosting adds patching, backup and recovery obligations. Keep essential household routing separate from experimental compute dependencies. Automation is useful when it saves work or supports learning, not when it makes every simple task a development project.

Include time and operational cost as well as purchase cost. Software that is free to download still needs updates and recovery work; repurposed hardware still consumes electricity. Prefer one well-understood service with tested recovery over a long list that nobody can operate after an outage.

Define a stable foundation and an experiment lifecycle: create, observe, learn, then remove or promote deliberately. Automate repeatable rebuilds where supported, and decide what evidence an experiment should produce. Promotion to a household dependency should add ownership, monitoring, access controls and data recovery, not merely a permanent restart policy.

---

## Learn Enterprise Patterns Without Enterprise Availability

**A homelab does not need an enterprise production-availability target to be useful.** Redundant gateways, independent physical hosts, switches, power paths and ISP links cost money to buy, power, support and replace. They also add configuration, monitoring, upgrades and failover tests. Most learning goals do not justify all of that recurring cost. Choose an acceptable outage and recovery process rather than copying an enterprise equipment list.

Learn the pattern at the scale you can afford. For example, several Kubernetes VMs on one hypervisor can teach scheduling, control-plane quorum, rolling upgrades and recovery from a guest failure. They **cannot** survive failure of their shared physical host, storage or power supply. Testing guest failure is useful evidence about that exercise, not proof of physical high availability. Similarly, VLANs can teach trust boundaries on shared hardware without making that hardware redundant.

State the distinction explicitly in a design: "For a production service with a strict availability requirement, distribute these roles across independent failure domains; for this lab, share a host and accept host-level downtime to reduce cost." Enterprise requirements vary too: spend on redundancy because an agreed recovery target demands it, not because every enterprise must have the same topology.

Do not confuse accepted downtime with accepted data loss or weak security. Protect important data, restrict access and keep recovery instructions and backups accessible independently of the failed system. Household internet may deserve a more stable foundation than disposable workloads without requiring an automatic-failover gateway pair. The [network availability and backup guidance](../2_HLD/Network/README.md#availability-backups-and-recovery) applies this trade-off to the shared home-and-lab network.

---

## From Goals to Categories

Turn goals into [high-level network/host requirements](../2_HLD/README.md), then select [platforms and services](../3_LLD/README.md). Revisit goals when a choice adds cost or operational work without supporting them.

Personal priorities and the actual decision record remain private under the [repository guidelines](../Repository_Guidelines.md).

For each major choice, record the requirement it satisfies and the cost/risk accepted. Review goals after changes in workload, available hardware or household reliance. This prevents expanding a cluster simply because a feature exists and helps explain the reasoning in a portfolio without publishing actual network inputs.
