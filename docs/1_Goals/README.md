# Goals and Constraints

Choose the purpose before buying equipment or installing tools.

---

## Table of Contents

- [Design the Purpose](#design-the-purpose)
- [Keep the Scope Sustainable](#keep-the-scope-sustainable)
- [From Goals to Categories](#from-goals-to-categories)

---

## Design the Purpose

A homelab may be a disposable learning environment, a development sandbox, a self-hosted household platform, or a mixture. Learning Kubernetes on demand is different from hosting the only copy of household files. Define what must remain available when experiments fail.

Record learning topics, useful services, data importance, acceptable downtime/data loss, budget, electricity, space/noise, cabling and maintenance time. Decide whether remote access is actually needed and who administers the system.

| Purpose | What to optimize | What to avoid assuming |
|---|---|---|
| Disposable learning | Fast rebuilds, isolated experiments, useful diagnostics | That every exercise needs permanent hardware or HA |
| Development/testing | Reproducible environments and realistic integration paths | That test credentials/data can share production trust |
| Household services | Predictable maintenance, backups and usable recovery | That family members accept lab-style downtime |
| Enterprise-platform practice | Specific management, upgrade and failure exercises | That copying an enterprise topology is required for other goals |

Write requirements as outcomes: "restore important data within the accepted time" or "keep household connectivity working during compute maintenance," not "buy three nodes." Identify must-haves, useful learning goals and deferred ideas. A tool is a means of achieving those outcomes, not the goal unless learning that tool is explicitly the exercise.

---

## Keep the Scope Sustainable

Prefer a small stable foundation and disposable experiments. One host can teach many concepts; extra machines are justified by capacity, physical-failure practice or actual availability requirements, not an enterprise-rack aesthetic.

Self-hosting adds patching, backup and recovery obligations. Keep essential household routing separate from experimental compute dependencies. Automation is useful when it saves work or supports learning, not when it makes every simple task a development project.

Include time and operational cost as well as purchase cost. Software that is free to download still needs updates and recovery work; repurposed hardware still consumes electricity. Prefer one well-understood service with tested recovery over a long list that nobody can operate after an outage.

Define a stable foundation and an experiment lifecycle: create, observe, learn, then remove or promote deliberately. Automate repeatable rebuilds where supported, and decide what evidence an experiment should produce. Promotion to a household dependency should add ownership, monitoring, access controls and data recovery, not merely a permanent restart policy.

---

## From Goals to Categories

Turn goals into [high-level network/host requirements](../2_HLD/README.md), then select [platforms and services](../3_LLD/README.md). Revisit goals when a choice adds cost or operational work without supporting them.

Personal priorities and the actual decision record remain private under the [repository guidelines](../Repository_Guidelines.md).

For each major choice, record the requirement it satisfies and the cost/risk accepted. Review goals after changes in workload, available hardware or household reliance. This prevents expanding a cluster simply because a feature exists and helps explain the reasoning in a portfolio without publishing actual network inputs.
