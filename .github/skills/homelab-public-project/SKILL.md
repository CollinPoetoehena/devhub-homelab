---
name: homelab-public-project
description: "Use when answering questions, designing, documenting or changing devhub-homelab public content. Applies generic homelab knowledge-base scope and the public/private publication boundary; not for the actual private setup."
---

# Public Homelab Project

1. Read the [public README](../../../README.md), [repository guidelines](../../../docs/Repository_Guidelines.md), and the relevant [category](../../../docs/README.md).
2. Confirm that the target is the public repository. Generic homelab criteria/options belong here; actual decisions, topology, inventory and operations belong in the private repository. Broad networking/Linux/AI concepts belong in [DevHub](https://github.com/CollinPoetoehena/DevHub/tree/main/reference).
3. Follow goals → HLD network/hosts → LLD platform/services. Explain options with enough narrative to understand how they work, benefits/limitations and when they fit, then broad implementation/verification considerations within that category. Do not recreate a separate setup tree or global hardware/software/tool catalog, or describe the owner's installed equipment, exact relationships, identifiers or access paths.
4. Use [homelab-sanitize](../homelab-sanitize/SKILL.md) whenever deriving a public lesson from private content. Do not copy private history, raw assets, configuration or generated output. If the boundary is uncertain, leave the detail private and report the omission.
5. Link reusable components through [DevHub packages](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/README.md); describe actual automation capabilities in Markdown rather than publishing site-specific code. Public implementation code is not required for this portfolio. Keep the high-level structure summary in the final section of [Project Structure](../../../docs/Project_Structure.md) when relevant changes are in scope, with planned areas labeled.
6. Check links, examples, claims and publication safety. Cross-repository links use full GitHub URLs on `main`; local document links may be relative. Report unverified behavior honestly.

Do not change the private installation, publish/push changes, or extract component code without a separate explicit request. Public guidance never authorizes live operations.
