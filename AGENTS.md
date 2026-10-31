# Agent Guidance

This is a standalone, public homelab book—not a deployment mirror. Use the [README](README.md) for an overview. Consult [repository architecture](ARCHITECTURE.md) for boundary or design changes, [guidelines](GUIDELINES.md) for documentation and publication work, and only the affected part README or chapter for subject changes.

- Keep public content generic and non-deployable. Do not inspect or copy private-repository inventories, configuration, or operational records for public work. Follow the [publication boundary](GUIDELINES.md#publication-boundary) and [sanitization workflow](GUIDELINES.md#sanitization-workflow).
- Explain homelab concepts, options, and trade-offs in this guide; link authoritative sources for deeper detail. DevHub is optional tooling, not a prerequisite or theory source.
- Distinguish illustrative or planned material from verified behavior; documentation is not evidence of deployment.
- `AGENTS.md` discovery is not universal; follow any additional guidance actually discovered by the active host and surface conflicts rather than assuming automatic precedence.
- The [APM manifest](apm.yml) declares the common baseline and shared homelab package for Copilot. Follow the [common engineering instructions](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/apm/common/.apm/instructions/common.instructions.md) alongside local guidance. The [homelab package](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/apm/homelab/README.md) is currently a placeholder with no active customizations. Declaration is not installation; follow the [repository setup and activation checks](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/apm/README.md#repository-setup).
- Keep one root `AGENTS.md` by default. Follow the [shared nested-instruction criteria](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/apm/README.md#nested-instructions) before considering an exception.
