# Homelab Repository Guidelines

This is the shared guideline for the [public homelab](https://github.com/CollinPoetoehena/devhub-homelab) and [private homelab](https://github.com/CollinPoetoehena/devhub-homelab-private). The [public README](../README.md#why-two-homelab-repositories) explains why the split exists, primarily to avoid exposing home-network internals and secondarily to keep reusable knowledge separate from operational detail.

## Content ownership

| Information | Owner |
|---|---|
| General networking, Linux, AI, and development concepts useful beyond homelabs | [DevHub reference](https://github.com/CollinPoetoehena/DevHub/tree/main/reference) |
| Homelab design patterns, options, trade-offs, buying criteria, generic setup and lessons | Public homelab |
| Actual equipment, addresses, domains, topology, access paths, configuration, decisions and operating procedures | Private homelab |
| Reusable Ansible roles, Terraform modules, Helm charts or packages | Independently maintained component repositories, indexed through [DevHub packages](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/README.md) |
| Credentials, private keys, decryption keys, sensitive state and raw backups | Protected secret/backup storage, not ordinary tracked files in either repository |

The public repository is a knowledge base and portfolio, **not a sanitized deployment mirror**. Do not copy private inventory, manifests, Terraform roots, switch exports, or generated configuration into it just because credentials have been removed. Document those capabilities in Markdown. Extract genuinely reusable code only as an explicit, separately reviewed component with generic interfaces and tests; the repository split itself does not move or publish code.

## Writing design and setup documentation

**Public design:** explain the problem, realistic options, benefits, drawbacks, and when each fits. Use role-based diagrams and clearly synthetic examples independent of the actual setup. There is no universal best hardware, Wi-Fi system, or Kubernetes installer.

**Private design:** link the relevant public document, then record the choice, date when known, specific rationale, accepted costs/risks, and a short explanation of why a few relevant alternatives were not selected. Do not reproduce the full alternatives catalog. Include actual topology, capacities, dependencies and open decisions; product names are welcome where they clarify the real design.

**Public setup:** describe prerequisites, dependency order, configuration intent, validation, and rollback generically. Tool-specific instructions may link to official guides, but must not reveal the environment that inspired them.

**Private setup:** record exact non-secret inputs, versions, port/SSID maps, executed steps, deviations, results, and recovery locations. An unverified example remains an example until someone confirms the actual value. Distinguish chosen, planned, installed, and verified; never infer deployment status from a document.

Use relative links within one repository. **Between repositories, always use full GitHub URLs on `main`**, for example `https://github.com/CollinPoetoehena/devhub-homelab/blob/main/docs/1_Design/3_Design_Network.md`. Do not use sibling filesystem paths, feature branches, or commit-specific links for this shared documentation boundary.

## Publication boundary

Keep the following out of public documents, issues, comments, screenshots, commits and artifacts:

- Actual home topology, device placement, inventory, VLAN/subnet maps, switch ports, SSIDs, firewall exceptions, management paths and recovery arrangements.
- Real public or private IP addresses, IPv6 prefixes, hostnames, domains, VPN endpoints, account identifiers, MAC addresses, serial numbers or identifying version inventories.
- Passwords, tokens, private keys, recovery codes, Wi-Fi credentials, certificate/key bundles, session data and decryption material.
- Private logs, terminal output, screenshots, raw exports, backups, Terraform state/plans, generated manifests, and metadata linking a generic example back to the household.
- Personal details such as location, household routines, device ownership or purchasing records.

RFC 1918 addresses and VLAN numbers are not credentials, but **the actual mapping still belongs privately**. Do not publish a real design with renamed labels: its relationships can remain identifying. Tool names and model families can be discussed as general options without saying which ones are installed at home.

Use reserved domains such as `example.com` and documentation address ranges `192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24` and `2001:db8::/32` for written examples. They are not production address allocations. A practical isolated-lab example needing private addressing must be independently invented, explicitly labeled synthetic, and checked for overlap before use. Placeholders must be visibly nonfunctional; never insert a real secret temporarily.

## Sanitization workflow

1. Identify the reusable lesson and its intended audience. Select a narrow scope; do not mirror a directory.
2. Write a standalone generic explanation from that lesson, not a line-by-line redaction of the live configuration. Remove actual structure, counts, mappings, dependencies and personal narrative where they reveal the household.
3. For design, replace the personal decision with neutral options and selection criteria. For setup, retain intent, prerequisites, validation and rollback without live inputs. For code, describe capabilities in Markdown or propose a separately maintained generic component.
4. Check prose, diagrams, code blocks, URLs, comments, filenames, image metadata, generated output and the staged diff. Do not import private history or raw assets. Review combinations of details, not just individual tokens.
5. Check link targets and example consistency. Do not claim tested behavior without evidence. If any detail cannot be confidently generalized, leave it private and report what was omitted.
6. Review the final public diff explicitly before publication. Content review is required even when an automated secret scan passes. No automatic sync, commit, push or live infrastructure change is implied by sanitization.

The [public sanitization skill](../.github/skills/homelab-sanitize/SKILL.md) operationalizes this process.

## Secrets and sensitive artifacts

Private access is not encryption and can change accidentally. Real **non-secret** environment details may be tracked privately; plaintext credentials must not be tracked in either repository.

- **Ansible:** encrypt necessary versioned values with Ansible Vault, verify the vault header, keep its password and private SSH keys outside Git, and prevent decrypted values leaking through task output, diffs or logs.
- **GitOps:** SOPS with age is one option; protect the private decryption key separately. Kubernetes Secret base64 encoding is not encryption. Ensure the chosen controller has an explicit, supported secret-delivery integration; choosing SOPS alone does not wire it into every controller.
- **Terraform:** keep state, plans, backups and crash output in secured storage with encryption, access control, locking and backups as appropriate. `sensitive` redacts some display; it does not encrypt state. Private roots may contain non-secret endpoint configuration, but must obtain tokens at runtime.
- **CI/CD:** use scoped platform secrets or an approved external secret manager, short-lived identities where supported, deployment approvals and protections against untrusted pull requests. Do not upload private operational artifacts to public jobs.
- **Device backups:** store raw exports and recovery material in protected backup storage accessible during a lab outage, not public Git. Inspect an artifact before assuming it contains no secrets.

Ignore rules are only a guardrail: they do not protect tracked files, history, arbitrary output filenames or accidentally staged content.

## If information is exposed

Stop further publication and assess the affected information privately. Revoke or rotate exposed credentials and keys first; deleting a file or rewriting history does not undo disclosure. For exposed internal details, assess reachable services, access controls and targeting risk rather than assuming private addresses are harmless. Restrict affected artifacts and follow the hosting platform's removal procedures where appropriate. Do not repost sensitive values in a public incident report.
