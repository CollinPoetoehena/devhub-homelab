---
name: homelab-sanitize
description: "Use when sanitizing, generalizing or preparing private homelab documentation, designs, setup notes or automation lessons for the public devhub-homelab portfolio. Removes real network internals and follows the repository split/publication guideline; not a private deployment mirror."
---

# Sanitize Homelab Material for Publication

---

## Table of Contents

- [Required Context](#required-context)
- [Procedure](#procedure)
- [Result](#result)

---

## Required Context

Read the [repository guideline](../../../docs/Repository_Guidelines.md), especially [content ownership](../../../docs/Repository_Guidelines.md#content-ownership), [publication boundary](../../../docs/Repository_Guidelines.md#publication-boundary), and [sanitization workflow](../../../docs/Repository_Guidelines.md#sanitization-workflow). Read the [two-repository explanation](../../../README.md#why-two-homelab-repositories).

---

## Procedure

1. Confirm the authorized source scope and public destination. Read only material relevant to the reusable lesson. Do not send private content to external services, open private endpoints, or enumerate unrelated credentials/artifacts.
2. Extract the lesson locally and write a **new generic explanation**, not a renamed copy of actual topology/configuration. Keep source content unchanged unless explicitly asked to revise private docs.
3. For **design**, describe options, benefits/drawbacks and selection criteria. Do not preserve actual quantities, placement, mappings, chosen-device combinations or household narrative. Link generic DevHub theory where it applies beyond homelabs.
4. Keep broad **implementation considerations** after design in the owning goals/HLD/LLD category, not a separate setup tree or an installation walkthrough modeled on the private lab. Hardware/software and automation options belong with their category. Keep real inputs/results private; synthetic examples must be clearly labeled and independent of the source environment.
5. For **code** such as YAML, Terraform and Ansible, publish Markdown explaining generic capabilities/interfaces. Do not copy actual inventory, manifests, roots, switch exports or generated configuration. A reusable package extraction requires separate authorization and [DevHub publishing guidance](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/README.md), generic inputs and appropriate tests.
6. Remove real public/private addresses, prefixes, identifiers, domains, SSIDs, firewall exceptions, administrative paths, version inventories, account/device metadata and secrets. Review combinations of details; redacting passwords alone is insufficient. Do not add real values even temporarily.
7. Review prose, diagrams, links, code blocks, comments, filenames and any assets/metadata against the guideline. Do not copy raw screenshots/logs/backups or private Git history. If uncertain, omit the detail and explain the omission without quoting it.
8. Preserve document ToCs and `---` between `##` sections; validate links/anchors, synthetic-value consistency and factual claims. Cross-repository references must be GitHub URLs on `main`. Review the final public diff explicitly; a secret scan alone is not publication approval. Private Git also must not hold plaintext sensitive public IPs, global prefixes or identifying endpoints; refer to Ansible Vault or equivalent protected values instead.

---

## Result

Report the generic documents changed, categories of information deliberately omitted, and checks performed. Keep the report free of private values. Do not commit, push, automatically sync repositories, publish component code or alter infrastructure unless separately requested.
