# Part 6 · Operations

> **Layer: Operations — how do we run it?** · [Book overview](../README.md) · Previous part: [Part 5 · Services](../05-services/README.md)

Building the homelab is only the first half of the story. This final part answers the storyline's seventh question: **how is it operated, observed and recovered?** It covers knowing whether everything works, keeping independent copies of what matters, recovering when something fails and maintaining the whole system over time.

Operations is where the earlier decisions are tested in practice. The [requirements](../01-foundations/requirements.md) set recovery objectives and maintenance limits; the [architecture](../02-architecture/README.md) decided which failure domains are shared; [automation](../04-automation/README.md) decides how much can be rebuilt from code. This part makes those decisions observable and recoverable.

---

## Table of Contents

- [Chapters in This Part](#chapters-in-this-part)
- [Cross-Cutting Chapters](#cross-cutting-chapters)
- [The Operating Loop](#the-operating-loop)
- [What Stays Private](#what-stays-private)

---

## Chapters in This Part

| # | Chapter | Question it answers | What you take away |
|---|---|---|---|
| 1 | [Observability and Monitoring](./observability.md) | How do I know it works, and when do I need to act? | Signals and their placement, monitoring/logging/alerting tools and actionable alerts |
| 2 | [Backups](./backups.md) | How are independent copies of important data kept? | Backup setups compared by cost and recovery value, backup tools by scope, independent copies and retention, restore verification |
| 3 | [Disaster Recovery](./disaster-recovery.md) | How does everything come back after a failure? | Failure scenarios, recovery order, the recovery kit, rebuild versus restore and rehearsals |
| 4 | [Maintenance](./maintenance.md) | How is it run and kept healthy over the years? | Updates by tier, change safety, recurring reviews, experiment cleanup, hardware lifecycle and documentation upkeep |

---

## Cross-Cutting Chapters

The chapters in this part are **cross-cutting**: they apply to every layer described earlier. To avoid the overlap that a split by document type used to create, they own the *method* and the subject chapters own their *specific application*:

- [Observability](./observability.md) owns signal and tool choices; each subject chapter says what to check for that subject.
- [Backups](./backups.md) owns backup tools, copies and retention; the [network chapter](../02-architecture/network.md#backups-that-survive-the-outage) owns how gateway configuration is backed up, and [storage](../02-architecture/storage.md) owns where primary data lives.
- [Disaster Recovery](./disaster-recovery.md) owns scenarios, order and rehearsals; each subject owns its own recovery options, such as [gateway recovery](../02-architecture/network.md#availability-backups-and-recovery).
- [Maintenance](./maintenance.md) owns the routine; every option table's **Maintenance** explanation owns the upkeep of that specific option.

---

## The Operating Loop

Operations is a loop rather than a sequence:

```text
Observe (signals, alerts) -> act (fix, update, change safely) -> protect (backups)
       ^                                                            |
       +---- review and improve (rehearsals, incidents, reviews) <---+
```

Decide what remains observable and recoverable when the workload platform is down. A dashboard stored only in that platform cannot report its complete outage; a replicated volume is not an independent backup. Document these accepted limits or add justified independent components.

Each pass through the loop should leave evidence: an alert acted upon, a restore that worked, a change that was verified, a requirement that was revisited. That evidence is what distinguishes a homelab that *should* work from one that is known to work.

---

## What Stays Private

This part explains the methods and generic checks. Actual monitoring targets, alert routes, backup locations, recovery orders, recovery-kit contents, maintenance schedules and test results belong in the private record, with credentials and sensitive endpoints only in protected storage under the [repository guidelines](../repository-guidelines.md#secrets-and-sensitive-artifacts).
