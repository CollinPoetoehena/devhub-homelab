---
name: homelab-public-docs
description: "Use when writing or updating public homelab category documentation, design options, hardware buying guidance, broad implementation considerations or reusable lessons. Layers public homelab context on the generic DevHub docs workflow."
---

# Public Homelab Documentation

1. Apply [homelab-public-project](../homelab-public-project/SKILL.md) and read the relevant public guides.
2. Invoke the generic DevHub [`docs` workflow](https://github.com/CollinPoetoehena/DevHub/blob/main/.github/skills/docs/SKILL.md) if available, otherwise read its linked guidance when accessible. A URL is not a skill installation.
3. Put homelab-specific theory and option comparisons here; link broadly reusable DevHub material. Give selection criteria rather than naming the actual owner's choice. Keep scope concise and practical.
4. Use goals → HLD network/hosts → LLD platform/services, with design first and broad implementation/verification considerations inside each category. Hardware/software and IaC/configuration/GitOps tools belong with their responsibility, not global catalogs or a separate setup tree. Avoid installation scripts, actual inventory, endpoint values, port/SSID maps, live topology or operational output.
5. For private-source adaptations, apply [homelab-sanitize](../homelab-sanitize/SKILL.md) before writing public output. Documentation for actual YAML/Terraform/Ansible capabilities should be generic Markdown; extraction is a separate task.
6. Preserve Table of Contents and `---` dividers between `##` sections. Validate local links/anchors and `main` cross-repository links, example consistency and publication safety using the [guideline](../../../docs/Repository_Guidelines.md). Sensitive public endpoints remain encrypted/protected even in private source material.

Return the changed public documents and any deliberately omitted/unverified content without repeating private details.
