---
name: homelab-public-implement-change
description: "Use when implementing changes to the public devhub-homelab knowledge base or reusable examples. Adds publication-safe homelab scope to the generic DevHub implement-change workflow; does not deploy the private lab."
---

# Public Homelab Changes

1. Apply [homelab-public-project](../homelab-public-project/SKILL.md).
2. Invoke the generic [`implement-change` workflow](https://github.com/CollinPoetoehena/DevHub/blob/main/.github/skills/implement-change/SKILL.md) if available, or follow its linked guidance when accessible.
3. Confirm the change fits the public knowledge base. Do not import private roots, inventories, rendered manifests, exports or live configuration. For actual-code topics, document the capability in Markdown unless separately asked to create a genuinely reusable component.
4. Apply [homelab-sanitize](../homelab-sanitize/SKILL.md) to private-derived material; follow the [repository guideline](../../../docs/Repository_Guidelines.md) for synthetic examples and secret handling.
5. Make focused changes, update related docs, and run the smallest appropriate checks. Never run a live apply/deployment as validation of a public example.
6. Review the complete public diff, links, comments and generated artifacts; report validation and unresolved publication concerns. Do not commit or push unless requested.
