# Homelab Documentation

Follow the story from purpose to infrastructure to workloads. Each category keeps its requirements, design options, hardware/software choices and broad implementation considerations together. **Design first; setup follows within the same topic**, not in a separate installation tree.

---

## Table of Contents

- [The Storyline](#the-storyline)
- [How to Use Each Category](#how-to-use-each-category)
- [Guidelines](#guidelines)

---

## The Storyline

| Stage | Question | Category |
|---|---|---|
| 1. Goals | What should the lab teach or provide, and what can it cost? | [Goals and constraints](./1_Goals/README.md) |
| 2. High-level design | What physical and logical foundation supports those goals? | [Network and hosts](./2_HLD/README.md) |
| 3. Low-level design | What platform and services run on that foundation? | [Orchestration, services, monitoring and data](./3_LLD/README.md) |

The [root README](../README.md#documentation-structure) explains the organization. HLD/LLD describe levels of detail, not separate phases to blindly complete: workload/storage requirements can send you back to revisit host capacity or network design.

> **Old Design+Setup Approach:** Previously, design and setup were treated as separate phases, often leading to repeated adjustments and inefficiencies. The current approach integrates design and setup within the same topic to streamline the process and maintain context. It is now a complete and cohesive workflow that explains a story from start to finish with all necessary details included.

---

## How to Use Each Category

Start with the design problem and options; select the hardware/software that meets it; then read the short implementation and verification considerations. This public reference is a decision guide, **not a vendor installation manual or a copy of the private deployment sequence**.

Actual selections, detailed runbooks and results belong in the matching private category. General protocols, commands and development workflows stay in [DevHub](https://github.com/CollinPoetoehena/DevHub/tree/main/reference).

---

## Guidelines

Use the [repository guidelines](./Repository_Guidelines.md) for the public/private boundary, sensitive values, publication and writing conventions. Tables of contents and dividers keep longer documents navigable.
