# Homelab Repository Guidelines

This is the shared guideline for the [public homelab](https://github.com/CollinPoetoehena/devhub-homelab) and [private homelab](https://github.com/CollinPoetoehena/devhub-homelab-private). The [public README](../README.md#why-two-homelab-repositories) explains why the split exists, primarily to avoid exposing home-network internals and secondarily to keep reusable knowledge separate from operational detail.

---

## Table of Contents

- [Content ownership](#content-ownership)
- [Portfolio and Project Structure](#portfolio-and-project-structure)
- [Writing design and setup documentation](#writing-design-and-setup-documentation)
- [Publication boundary](#publication-boundary)
- [Sanitization workflow](#sanitization-workflow)
- [Secrets and sensitive artifacts](#secrets-and-sensitive-artifacts)
- [If information is exposed](#if-information-is-exposed)

---

## Content ownership

| Information | Owner |
|---|---|
| General networking, Linux, AI, and development concepts useful beyond homelabs | [DevHub reference](https://github.com/CollinPoetoehena/DevHub/tree/main/reference) |
| Homelab design patterns, options, trade-offs, buying criteria, generic setup and lessons | Public homelab |
| Actual equipment, non-sensitive internal settings, topology, decisions and operating procedures | Private homelab |
| Sensitive public IPs, global prefixes and identifying public domains/access endpoints | Encrypted/protected storage; private prose uses references |
| Reusable Ansible roles, Terraform modules, Helm charts or packages | Independently maintained component repositories, indexed through [DevHub packages](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/README.md) |
| Credentials, private keys, decryption keys, sensitive state and raw backups | Protected secret/backup storage, not ordinary tracked files in either repository |

The public repository is a knowledge base and portfolio, **not a sanitized deployment mirror**. Do not copy private inventory, manifests, Terraform roots, switch exports, or generated configuration into it just because credentials have been removed. Document those capabilities in Markdown. Extract genuinely reusable code only as an explicit, separately reviewed component with generic interfaces and tests; the repository split itself does not move or publish code.

---

## Portfolio and Project Structure

**You do not have to publish actual implementation code for this portfolio.** A high-level project structure can show how documentation, provisioning, configuration management and workload delivery connect. Combined with the generic design/setup explanations, it is sufficient for the intended architectural portfolio: reasoning, responsibilities and trade-offs matter more here than a concrete subnet value, credential or rendered manifest.

Withholding private code is deliberate, not an incomplete public repository. The [publication boundary](#publication-boundary) explains why implementation relationships can reveal sensitive home-network internals even after passwords are removed. Structure alone is not proof that a system is deployed or tests passed; describe evidence honestly without exposing private outputs.

Keep the single maintained overview at [Project Structure](./Project_Structure.md#general-homelab-project-structure). AI may regenerate that final section as a sanitized responsibility-level summary, **not raw `tree` output**. Include the main README, documentation and integration areas; omit device/application/environment listings and sensitive filenames. Label planned areas and do not invent directories. Reusable packages still follow [DevHub package guidance](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/README.md), with separate authorization for extraction/publication.

---

## Writing design and setup documentation

**Public design:** explain the problem, realistic options, benefits, drawbacks and when each fits. Tables provide orientation; add enough narrative to explain how options work, what ownership they add and how to evaluate them. Go deeper on host purchasing/virtualization and network boundaries without turning guidance into an exact private installation. Use role-based diagrams and clearly synthetic examples independent of the actual setup. There is no universal best hardware, Wi-Fi system or Kubernetes installer.

**Private design:** link the relevant public document, then record the choice, date when known, specific rationale, accepted costs/risks, and a short explanation of why a few relevant alternatives were not selected. Do not reproduce the full alternatives catalog. Include actual topology, capacities, dependencies and open decisions; product names are welcome where they clarify the real design.

**Public implementation considerations:** describe prerequisites, dependency order, configuration intent, validation and rollback broadly inside the topic. Link official installation guides rather than maintaining an exact walkthrough or revealing the environment that inspired it.

Organize both repositories by the [category storyline](../README.md#documentation-structure): goals, HLD network/hosts, then LLD orchestration/services/monitoring/storage. Within a category, **design comes before implementation**. Keep hardware and software with the responsibility they support; setup is part of that category, not a separate public setup tree. Public implementation guidance stays broad and does not reproduce the private installation sequence. Private categories may contain detailed design and setup/runbook documents together.

Add a **Table of Contents** for documents with sections, and `---` dividers between `##` sections. Keep ToC links synchronized with headings; do not confuse Markdown dividers with a skill's YAML frontmatter.

**Private setup:** record exact verified non-sensitive inputs, versions, port/SSID maps, executed steps, deviations, results and recovery references. Sensitive endpoint values remain encrypted/protected rather than plaintext in prose. An unverified example remains an example until someone confirms the actual value. Distinguish chosen, planned, installed, and verified; never infer deployment status from a document.

Use relative links within one repository. **Between repositories, always use full GitHub URLs on `main`**, for example `https://github.com/CollinPoetoehena/devhub-homelab/blob/main/docs/2_HLD/Network/README.md`. Do not use sibling filesystem paths, feature branches, or commit-specific links for this shared documentation boundary.

---

## Publication boundary

Keep the following out of public documents, issues, comments, screenshots, commits and artifacts:

- Actual home topology, device placement, inventory, VLAN/subnet maps, switch ports, SSIDs, firewall exceptions, management paths and recovery arrangements.
- Real public or private IP addresses, IPv6 prefixes, hostnames, domains, VPN endpoints, account identifiers, MAC addresses, serial numbers or identifying version inventories.
- Passwords, tokens, private keys, recovery codes, Wi-Fi credentials, certificate/key bundles, session data and decryption material.
- Private logs, terminal output, screenshots, raw exports, backups, Terraform state/plans, generated manifests, and metadata linking a generic example back to the household.
- Personal details such as location, household routines, device ownership or purchasing records.

RFC 1918 addresses and VLAN numbers are not credentials, but **the actual mapping still belongs privately**. Do not publish a real design with renamed labels: its relationships can remain identifying. Tool names and model families can be discussed as general options without saying which ones are installed at home.

Actual non-sensitive internal VLAN IDs, subnets, gateways, host allocations, port mappings and SSID labels may be recorded directly in the **private** documentation. They do not need artificial example labels merely because the public repository uses generic content. A value confirmed by the owner as the chosen setting can be a real design input without claiming it has already been applied or tested. Preserve genuine illustrative commands and unresolved facts as such; private visibility is not evidence of deployment.

Use reserved domains such as `example.com` and documentation address ranges `192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24` and `2001:db8::/32` for written examples. They are not production address allocations. A practical isolated-lab example needing private addressing must be independently invented, explicitly labeled synthetic, and checked for overlap before use. Placeholders must be visibly nonfunctional; never insert a real secret temporarily.

---

## Sanitization workflow

1. Identify the reusable lesson and its intended audience. Select a narrow scope; do not mirror a directory.
2. Write a standalone generic explanation from that lesson, not a line-by-line redaction of the live configuration. Remove actual topology, counts, mappings, dependencies and personal narrative where they reveal the household. A responsibility-level project structure may be shown under the [structure policy](#portfolio-and-project-structure), without operational paths or inventories.
3. For design, replace the personal decision with neutral options and selection criteria. For setup, retain intent, prerequisites, validation and rollback without live inputs. For code, describe capabilities in Markdown or propose a separately maintained generic component.
4. Check prose, diagrams, code blocks, URLs, comments, filenames, image metadata, generated output and the staged diff. Do not import private history or raw assets. Review combinations of details, not just individual tokens.
5. Check link targets and example consistency. Do not claim tested behavior without evidence. If any detail cannot be confidently generalized, leave it private and report what was omitted.
6. Review the final public diff explicitly before publication. Content review is required even when an automated secret scan passes. No automatic sync, commit, push or live infrastructure change is implied by sanitization.

The [public sanitization skill](../.github/skills/homelab-sanitize/SKILL.md) operationalizes this process.

---

## Secrets and sensitive artifacts

Private access is not encryption and can change accidentally. Real ordinary non-secret environment details may be tracked privately, but **sensitive access/identifying values must not be exposed in plaintext even in the private repository**. This includes actual public IPv4 addresses, globally routable IPv6 prefixes, internet-facing hostnames/domains and VPN/management endpoints, as well as credentials. Use a named placeholder/reference in prose and store the actual value in Ansible Vault, SOPS-encrypted values or an approved external secret store when needed. Do not include decrypted values in screenshots, command history, logs or chat.

Internal role/topology decisions can remain useful private records; classify actual values before writing them rather than assuming every non-password value is harmless. Encryption must have separately protected keys, access controls and a recovery process. A private endpoint variable and a Terraform `sensitive` label alone do not encrypt its value.

- **Ansible:** encrypt necessary versioned values, including sensitive public endpoint/address data, with Ansible Vault; verify the vault header, keep its password and private SSH keys outside Git, and prevent decrypted values leaking through task output, diffs or logs.
- **GitOps:** SOPS with age is one option; protect the private decryption key separately. Kubernetes Secret base64 encoding is not encryption. Ensure the chosen controller has an explicit, supported secret-delivery integration; choosing SOPS alone does not wire it into every controller.
- **Terraform:** keep state, plans, backups and crash output in secured storage with encryption, access control, locking and backups as appropriate. `sensitive` redacts some display; it does not encrypt state. Private roots may contain ordinary non-sensitive configuration; obtain sensitive endpoints and tokens from protected runtime storage rather than plaintext tracked values.
- **CI/CD:** use scoped platform secrets or an approved external secret manager, short-lived identities where supported, deployment approvals and protections against untrusted pull requests. Do not upload private operational artifacts to public jobs.
- **Device backups:** store raw exports and recovery material in protected backup storage accessible during a lab outage, not public Git. Inspect an artifact before assuming it contains no secrets.

Ignore rules are only a guardrail: they do not protect tracked files, history, arbitrary output filenames or accidentally staged content.

---

## If information is exposed

Stop further publication and assess the affected information privately. Revoke or rotate exposed credentials and keys first; deleting a file or rewriting history does not undo disclosure. For exposed internal details, assess reachable services, access controls and targeting risk rather than assuming private addresses are harmless. Restrict affected artifacts and follow the hosting platform's removal procedures where appropriate. Do not repost sensitive values in a public incident report.
