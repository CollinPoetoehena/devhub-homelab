# Public Homelab Context

This repository is a public homelab knowledge base and portfolio, not the actual home setup. Read [repository guidelines](../docs/Repository_Guidelines.md) before changes.

- Keep homelab explanations generic, option-oriented and independent of the actual household network. Do not mirror private configuration, assets or history.
- Read and apply the [Documentation Structure & Guideline](../docs/Repository_Guidelines.md#documentation-structure--guideline). It owns structure, comparison format/rationale, navigation and maintenance conventions; do not repeat them in skills or instructions.
- Broad concepts/workflows belong in DevHub; link them rather than duplicate them. Actual decisions and non-secret configuration belong in the private homelab.
- Use the shared DevHub skills listed in [AI and Copilot Skills](../README.md#ai-and-copilot-skills); they discover local context automatically, so no project-context or generic workflow wrapper skill is needed. Keep persistent homelab context here and only add genuinely homelab-specific customizations under [local skills](./skills/README.md).
- Follow [DevHub's AI customization guidance](https://github.com/CollinPoetoehena/DevHub/blob/main/.github/AI.md) for general design and setup. Do not chain skills: each must be self-contained. Public adaptation of private material is a separate authorized task using [homelab-sanitize](./skills/homelab-sanitize/SKILL.md), not an automatic follow-up. Keep public output free of real internals and secrets; omit uncertain details and report omissions without disclosing them.
- Apply the [portfolio/structure policy](../docs/Repository_Guidelines.md#portfolio-and-project-structure) and [Project Structure maintenance rules](../docs/Project_Structure.md#maintaining-the-structure-summary-with-ai) when relevant.
- Private-repository status does not make sensitive public IPs, global prefixes or identifying access endpoints safe for plaintext Git; use encrypted/protected storage under the shared guideline.
- Do not commit, push, extract code or change infrastructure unless explicitly requested. Never use live deployment as validation of public examples.
