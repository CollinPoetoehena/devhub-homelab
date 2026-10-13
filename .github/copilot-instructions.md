# Public Homelab Context

This repository is a public homelab knowledge base and portfolio, not the actual home setup. Read [repository guidelines](../docs/Repository_Guidelines.md) before changes.

- Keep homelab explanations generic, option-oriented and independent of the actual household network. Do not mirror private configuration, assets or history.
- Follow goals → HLD network/hosts → LLD platform/services. Put hardware/software and automation options in their owning category. Design comes before broad implementation/verification considerations; do not create a separate setup tree or tool catalog. Keep document ToCs and `---` between level-two sections.
- Broad concepts/workflows belong in DevHub; link them rather than duplicate them. Actual decisions and non-secret configuration belong in the private homelab.
- Use the focused [public skills](./skills/README.md), especially `homelab-sanitize` for adapting private material for publication. Keep public output free of real internals and secrets.
- Public implementation code is not required for this portfolio. Maintain the sanitized responsibility-level overview at the end of [Project Structure](../docs/Project_Structure.md) when structure changes are in scope; never paste raw directory output or invent planned integration folders.
- Private-repository status does not make sensitive public IPs, global prefixes or identifying access endpoints safe for plaintext Git; use encrypted/protected storage under the shared guideline.
- Cross-repository documentation links must be full GitHub URLs on `main`. Within this repository, use relative links.
- Do not commit, push, extract code or change infrastructure unless explicitly requested.
