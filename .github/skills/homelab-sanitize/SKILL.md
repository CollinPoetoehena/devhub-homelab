---
name: homelab-sanitize
description: "Use when sanitizing, generalizing or preparing private homelab documentation, designs, setup notes or automation lessons for the public devhub-homelab portfolio. Removes real network internals and follows the repository split/publication guideline; not a private deployment mirror."
---

> **Invocation-specific instructions take precedence:** Follow explicit directions in the current request over conflicting defaults here. For example, honor a request to skip a documentation check and report it as skipped. Higher-priority instructions, host constraints, and safety requirements still apply.

# Sanitize Homelab Material for Publication

---

## Table of Contents

- [Required Context](#required-context)
- [Procedure](#procedure)
- [Result](#result)

---

## Required Context

This is a self-contained publication-preparation workflow; do not invoke another skill as a prerequisite or follow-up. When loaded through a parent-workspace symbolic link, resolve the project-policy links below against the owning `devhub-homelab` clone, not the parent folder. Confirm that clone as the public destination before editing.

Identify the authorized source and public destination repositories. Discover and read the applicable instructions and policies in both, starting with `.github/copilot-instructions.md`, `README.md`, `docs/Repository_Guidelines.md`, and `docs/Project_Structure.md` when present. Read only source files within the authorized scope; do not enumerate unrelated private records or sensitive artifacts. Follow relevant links to locate authoritative project-specific rules. These paths are starting points, not an exhaustive search: if guidance is missing or a repository uses another layout, inspect its structure and relevant documentation/configuration for the applicable source of truth. Do not invent policy or treat a link as authorization. If scope, destination, or authorization cannot be established safely, stop before editing and report the blocker.

In this repository, the [repository guideline](../../../docs/Repository_Guidelines.md) is the policy source for [content ownership](../../../docs/Repository_Guidelines.md#content-ownership), the [publication boundary](../../../docs/Repository_Guidelines.md#publication-boundary), the [sanitization workflow](../../../docs/Repository_Guidelines.md#sanitization-workflow), and the [Documentation Structure & Guideline](../../../docs/Repository_Guidelines.md#documentation-structure--guideline). The [two-repository explanation](../../../README.md#why-two-homelab-repositories) establishes the public/private split; [Project Structure maintenance rules](../../../docs/Project_Structure.md#maintaining-the-structure-summary-with-ai) apply when that summary is in scope. These internal links provide project context, not prerequisites for understanding the self-contained safety rules below.

---

## Procedure

1. Confirm the authorized source scope and public destination. Read only material relevant to the reusable lesson. Do not send private content to external services, open private endpoints, or enumerate unrelated credentials/artifacts.
2. Extract the lesson locally and write a **new generic explanation**, not a renamed copy of actual topology/configuration. Keep source content unchanged unless explicitly asked to revise private docs.
3. Adapt design lessons under the writing guideline. Do not preserve actual quantities, placement, mappings, chosen-device combinations or household narrative. Link generic DevHub theory where it applies beyond homelabs.
4. Keep real implementation inputs/results private; synthetic examples must be clearly labeled and independent of the source environment.
5. For **code** such as YAML, Terraform and Ansible, publish Markdown explaining generic capabilities/interfaces. Do not copy actual inventory, manifests, roots, switch exports or generated configuration. A reusable package extraction requires separate authorization and [DevHub publishing guidance](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/README.md), generic inputs and appropriate tests.
6. Remove real public/private addresses, prefixes, identifiers, domains, SSIDs, firewall exceptions, administrative paths, version inventories, account/device metadata and secrets. Review combinations of details; redacting passwords alone is insufficient. Do not add real values even temporarily.
7. Review prose, diagrams, links, code blocks, comments, filenames and any assets/metadata against the guideline. Do not copy raw screenshots/logs/backups or private Git history. If uncertain, omit the detail and explain the omission without quoting it.
8. Discover relevant safe validation commands from repository instructions, documentation, manifests/scripts and CI configuration. Run the narrowest applicable checks for factual consistency, links, synthetic-value consistency and publication safety; do not deploy or contact private endpoints as validation. Review the final public diff explicitly; a secret scan alone is not publication approval. Private Git also must not hold plaintext sensitive public IPs, global prefixes or identifying endpoints; refer to Ansible Vault or equivalent protected values instead. If no relevant automated check exists or a check cannot safely run, state that limitation.
9. For project-structure requests, follow [Project Structure maintenance rules](../../../docs/Project_Structure.md#maintaining-the-structure-summary-with-ai) and the [portfolio/structure policy](../../../docs/Repository_Guidelines.md#portfolio-and-project-structure).

---

## Result

Report the generic documents changed, categories of information deliberately omitted, and checks performed; keep the report free of private values. Also identify the project files actually read for publication policy, documentation structure, and validation commands, explain each file's role, distinguish inferred conventions from explicit rules, and name any missing guidance or discovery limitations. Do not claim to have read files that were not inspected.

Before composing any commit-message suggestion, inspect `git diff`, `git diff --cached`, and `git status --short` in every affected repository, and review relevant untracked files (for example, with `git ls-files --others --exclude-standard`). Base the suggested message on the complete pending change set that `git add -A` would stage, including earlier session changes. Flag unrelated or unreviewed changes without claiming ownership or validation of them. Inspect the current branch, configured remotes, and upstream; use `git push` only for a verified upstream, or `git push --set-upstream <verified-remote> <current-branch>` when no upstream exists. Never invent a remote, branch, or push target.

When files changed in a Git repository and a safe commit/push sequence is known, warn that `git add -A` stages all changes, including unrelated edits, and ask the user to review the complete diff before showing a copy-ready `sh` block with `git add -A && git commit -m "..."` followed by the verified push command. For example, a sanitized homelab-guide change might use `docs: publish sanitized homelab guidance`; select the actual message from the complete reviewed pending changes. Do not leave placeholders in the reported commands or add a co-author trailer. These commands are for the user: never commit or push unless explicitly requested. If no files changed, the work is outside Git, or no safe target can be verified, explain why the commands are omitted or incomplete.
