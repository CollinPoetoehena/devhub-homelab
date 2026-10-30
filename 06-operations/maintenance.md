# Maintenance

> **Part 6 · Operations** — [Part overview](./README.md) · Previous: [Disaster Recovery](./disaster-recovery.md) · Next: back to the [book overview](../README.md)

A homelab is never finished. Firmware and software need updates, disks wear out, certificates expire, experiments accumulate, requirements change and documentation drifts. Maintenance is the routine that keeps the homelab safe and useful over years instead of months—and the place where the cost of every earlier decision shows up as recurring time. That is why every option table in this book includes an explicit **Maintenance** explanation.

This chapter closes the storyline by answering: **how is the homelab run day to day?** It covers update management, change safety, capacity and lifecycle reviews, cleaning up experiments, and keeping the documentation itself current.

---

## Table of Contents

- [Purpose and Requirements](#purpose-and-requirements)
- [Updates and Patching](#updates-and-patching)
- [Change Safety](#change-safety)
- [Recurring Reviews](#recurring-reviews)
- [Experiment Cleanup](#experiment-cleanup)
- [Hardware Lifecycle and Retirement](#hardware-lifecycle-and-retirement)
- [Keeping the Documentation Current](#keeping-the-documentation-current)
- [Implementation and Verification](#implementation-and-verification)
- [Further Reading](#further-reading)

---

## Purpose and Requirements

Maintenance requirements come from the [foundations](../01-foundations/requirements.md): the time available, the household's tolerance for outages and the security exposure of each component. A realistic maintenance routine is small, regular and predictable. If it does not fit the time available, the answer is usually to reduce what is operated—fewer services, simpler foundations—rather than to skip updates. This is the principle [spend maintenance effort where the learning is](../01-foundations/design-principles.md#spend-maintenance-effort-where-the-learning-is).

Respect [relaxation and enjoyment over productivity and pressure](../01-foundations/goals-and-scope.md#relaxation-and-enjoyment-over-productivity-and-pressure): plan routine upkeep for chosen lab time, not alongside work, errands or rest. Optional [timeboxes](../01-foundations/goals-and-scope.md#timebox-without-turning-it-into-a-deadline) can give maintenance a start/stop boundary without turning it into a quota. [Track follow-ups](../README.md#task-tracking) so routine work can wait without being forgotten; configure ordinary lab alerts not to interrupt or demand assessment outside a chosen lab session. Reserve prompt notification for a credible emergency, as described in [Notifications Are Not Automatic Interruptions](../01-foundations/goals-and-scope.md#notifications-are-not-automatic-interruptions). If routine obligations repeatedly demand immediate attention, reduce or move the services that create them rather than accepting permanent personal on-call coverage.

---

## Updates and Patching

Update by tier, with different caution for different components:

| Tier | Approach |
|---|---|
| Foundation (gateway, switches, APs) | Read release notes, take a configuration backup, update during an agreed window with a wired recovery path, verify connectivity and segmentation afterwards |
| Lab platform (hypervisor, cluster, automation tooling) | Follow supported upgrade paths and version-compatibility matrices, update one member at a time where possible, verify health before continuing |
| Household services | Patch regularly; for stateful services, back up first and read migration notes; verify the service and its data afterwards |
| Experiments | Do not patch indefinitely—remove or rebuild them |

Prefer automation for repeatable updates and pin versions where reproducibility matters, then update the pins deliberately. Security updates for exposed components—anything reachable from the internet or from less-trusted segments—take priority. Keep an eye on support lifecycles: hardware and software that no longer receive security updates should be replaced or isolated before that date, not after.

---

## Change Safety

Most homelab outages are self-inflicted. A few habits prevent most of them:

- **Change one boundary at a time** and verify it before the next change.
- **Save the last working configuration** before changing it, and know how to roll back.
- **Keep a known-good access path** while changing access: establish the new administration path before removing the old one.
- **Prefer reviewed declarations to ad-hoc clicks** where automation exists, and record manual changes where it does not.
- **Schedule disruptive work** for times when the household is not relying on the affected component.
- **Keep a short change log** for the foundation tier: what changed, why and how it was verified.

These rules appear in context throughout the book—for example in the [network implementation checks](../02-architecture/network.md#implementation-and-verification)—and are collected here as the operational routine.

---

## Recurring Reviews

A short, regular review catches slow problems before they become outages. A generic checklist:

| Interval | Review |
|---|---|
| Weekly or after alerts | Failed backups, disk and capacity warnings, certificate-expiry warnings, unexpected restarts |
| Monthly | Pending updates, backup verification results, resource usage trends, stale experiments |
| Quarterly | A restore test, firmware/support status, accounts and tokens, firewall exceptions and allowed flows, DNS records |
| Yearly or after major changes | Requirements and service tiers, recovery kit and rehearsal, energy cost against measurements, hardware lifecycle, documentation review |

Adjust the intervals to the homelab's size and the time available. The value comes from regularity, not from the exact schedule. [Observability](./observability.md) supplies most of the evidence these reviews need.

---

## Experiment Cleanup

Experiments are meant to end. When an experiment has answered its question, keep the learning—notes, diagrams, scripts and measurements—and remove the infrastructure: VMs, volumes, credentials and tokens, DNS records, firewall exceptions, monitoring targets and scheduled jobs. If an experiment should continue, [promote it deliberately](../05-services/workloads.md#promoting-an-experiment) with the obligations of its new tier. An experiment that just keeps running is the most common way for a homelab to accumulate unmaintained, unpatched and unnoticed components.

Before cleanup, [record the results and outcomes](../05-services/workloads.md#recording-experiment-and-workload-outcomes), including failures, limitations and remaining questions. Short Markdown notes preserve what the temporary environment taught you so you can revisit or repeat the exercise without leaving it running.

---

## Hardware Lifecycle and Retirement

Track the age, warranty, firmware support and health of hardware—especially disks, batteries and fans, which wear out first. Plan replacements before failures where the component's tier justifies it, and re-check the [cost chapter](../01-foundations/cost.md) before replacing working equipment purely for efficiency: measured savings should cover the purchase over a realistic period.

When retiring hardware, migrate or delete its data, remove it from inventories, automation and monitoring, revoke its credentials and certificates, and securely erase storage before reuse, sale or disposal. Configuration exports and backups of retired devices remain sensitive.

---

## Keeping the Documentation Current

Documentation is part of the system. After a change, update the subject chapter that owns it—one file per subject makes the owner obvious—and keep the distinction between chosen, planned, installed and verified honest. During yearly reviews, read the book from start to finish: outdated statements are easier to notice when the story no longer flows.

For this public book there is an additional milestone: the final documentation review described at the top of the [README](../README.md), in which practical lessons from the actual homelab are used to improve the generic guidance, release-dependent claims are rechecked against official documentation, and the [publication boundary](../GUIDELINES.md#publication-boundary) is verified again. Private operational details remain in the private record.

---

## Implementation and Verification

Write the actual maintenance routine for the installed components in the private record: update windows, intervals, owners and the change log. Automate what is repeatable—update checks, backup verification reports, expiry warnings—and keep the manual steps short and documented.

Verify the routine itself occasionally: are reviews actually happening, are alerts acted on, and is the time spent on maintenance in line with the [requirements](../01-foundations/requirements.md#constraints)? If maintenance consistently takes more time than planned, simplify the homelab rather than letting updates slip.

## Further Reading

For patch-management planning and enterprise patching practices, see NIST's [Guide to Enterprise Patch Management Planning (SP 800-40 Rev. 4)](https://csrc.nist.gov/pubs/sp/800/40/r4/final). Apply only the controls that match the lab's exposure and availability needs; the guide is not a mandate to reproduce enterprise process. See [Goals and Scope](../01-foundations/goals-and-scope.md#define-what-is-out-of-scope).
