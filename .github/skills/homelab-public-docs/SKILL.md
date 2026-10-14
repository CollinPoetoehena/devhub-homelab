---
name: homelab-public-docs
description: "Use when writing or updating public homelab category documentation, design options, hardware buying guidance, implementation considerations, project structure or reusable lessons. Layers public homelab context on the generic DevHub docs workflow."
---

# Public Homelab Documentation

1. Apply [homelab-public-project](../homelab-public-project/SKILL.md) and read the relevant public guides.
2. Invoke the generic DevHub [`docs` workflow](https://github.com/CollinPoetoehena/DevHub/blob/main/.github/skills/docs/SKILL.md) if available, otherwise read its linked guidance when accessible. A URL is not a skill installation.
3. Read and apply the [Documentation Structure & Guideline](../../../docs/Repository_Guidelines.md#documentation-structure--guideline) for organization, comparisons, writing scope and maintenance. Do not duplicate those conventions in this skill.
4. Put homelab-specific theory here and link broadly reusable DevHub material. Keep actual inputs and operational output private under the [publication boundary](../../../docs/Repository_Guidelines.md#publication-boundary).
5. For private-source adaptations, apply [homelab-sanitize](../homelab-sanitize/SKILL.md) before writing public output. Documentation for actual YAML/Terraform/Ansible capabilities should be generic Markdown; extraction is a separate task.
6. Validate the changed documents against the writing guideline and [repository guidelines](../../../docs/Repository_Guidelines.md), including factual consistency and publication safety.
7. For project-structure or final-review requests, follow the maintenance rules linked from the writing guideline.

Return the changed public documents and any deliberately omitted/unverified content without repeating private details.
