# Homelab Repository Guidelines

> **Central repository guidelines:** This file is the shared source of truth for repository policies, including documentation conventions, content ownership, publication boundaries and protected-data handling. Both homelab repositories, their Copilot customizations and other documentation reference the applicable sections here.

This is the shared guideline for the [public homelab](https://github.com/CollinPoetoehena/devhub-homelab) and [private homelab](https://github.com/CollinPoetoehena/devhub-homelab-private). The [public README](./README.md#why-two-homelab-repositories) explains why the split exists: protect home-network internals while keeping the public guide useful and independently readable.

---

## Table of Contents

- [Content ownership](#content-ownership)
- [DevHub Relationship and Scope](#devhub-relationship-and-scope)
- [Portfolio and Project Structure](#portfolio-and-project-structure)
- [Documentation Structure & Guideline](#documentation-structure--guideline)
  - [The Book Model: Layers and Storyline](#the-book-model-layers-and-storyline)
  - [Why Not High-Level and Low-Level Design Folders](#why-not-high-level-and-low-level-design-folders)
  - [One File per Subject](#one-file-per-subject)
  - [Chapter READMEs as Entry Points](#chapter-readmes-as-entry-points)
  - [Subject Chapters and Cross-Cutting Chapters](#subject-chapters-and-cross-cutting-chapters)
  - [Private Repository Structure](#private-repository-structure)
  - [Option Comparisons and Table Rationale](#option-comparisons-and-table-rationale)
  - [Design, Implementation and Evidence](#design-implementation-and-evidence)
  - [Navigation, Links and Maintained Overviews](#navigation-links-and-maintained-overviews)
- [Publication boundary](#publication-boundary)
- [Sanitization workflow](#sanitization-workflow)
- [Secrets and sensitive artifacts](#secrets-and-sensitive-artifacts)
- [If information is exposed](#if-information-is-exposed)

---

## Content ownership

| Information | Owner |
|---|---|
| Homelab-relevant concepts, design options, trade-offs, selection criteria, generic setup and lessons | Public homelab, with links to official standards and product documentation |
| Shared personal-project practices, repository conventions, reusable-workflow guidance, AI customizations and task tracking | [DevHub](https://github.com/CollinPoetoehena/DevHub) |
| Actual equipment, non-sensitive internal settings, topology, decisions and operating procedures | Private homelab |
| Sensitive public IPs, global prefixes and identifying public domains/access endpoints | Encrypted/protected storage; private prose uses references |
| Packages maintained in DevHub | [DevHub package index](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/README.md); check each entry for its actual implementation and release status |
| Other reusable Ansible roles, Terraform modules, Helm charts or packages | Their owning component repositories and documentation |
| Credentials, private keys, decryption keys, sensitive state and raw backups | Protected secret/backup storage, not ordinary tracked files in either repository |

The [main README](./README.md#how-devhub-and-the-homelab-repositories-fit-together) explains the repository roles. The public guide owns the concepts and choices needed to understand a homelab; it is not an index into a separate theory library or an attempt to reproduce general technical theory. Give only enough context to explain the homelab-specific decision, trade-off and consequence. Link to authoritative standards or official product documentation for deeper detail, then state the home/lab-specific implications here. The [main README](./README.md) and [Goals and Scope](./01-foundations/goals-and-scope.md#define-what-is-out-of-scope) explain this boundary to readers.

The public repository is a standalone homelab guide and portfolio—the readable, open-source story of the project—**not a code repository or sanitized deployment mirror**. It may recommend approaches and explain trade-offs, but never exposes the exact architecture and low-level details that could help an attacker; they are not needed to tell the story. Do not copy private inventory, manifests, Terraform roots, switch exports, implementation code, or generated configuration into it just because credentials have been removed. Fenced examples may illustrate an idea, but must be generic, clearly labeled and non-deployable as a copy of the actual environment. Document capabilities in Markdown. Extract genuinely reusable code only as an explicit, separately reviewed component in an appropriate package repository, with generic interfaces and tests; the repository split itself does not move or publish code.

---

## DevHub Relationship and Scope

DevHub is the author's shared home for personal-project practices and reusable tooling. It is not a prerequisite for this guide, and it is not the source of homelab theory: each public chapter explains the relevant context, options and trade-offs, then links to authoritative external documentation where further detail is useful. This repository follows shared practices while keeping its homelab content self-contained.

Useful DevHub entry points for the author's project workflow:

- [Task tracking](https://github.com/CollinPoetoehena/DevHub/blob/main/docs/TASK-TRACKING.md) describes the optional private backlog shared across personal projects.
- [AI customizations](https://github.com/CollinPoetoehena/DevHub/blob/main/docs/USAGE.md#ai-customizations) describes the APM-based common agents, skills and scoped instructions; [project-specific guidance](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/apm/common/README.md#project-specific-guidance) explains how a project adds local context.
- [Project conventions](https://github.com/CollinPoetoehena/DevHub/blob/main/docs/ARCHITECTURE.md#naming-conventions) and [repository boundaries](https://github.com/CollinPoetoehena/DevHub/blob/main/docs/ARCHITECTURE.md#separate-repositories) describe shared organization choices.
- [Reusable GitHub Actions guidance](https://github.com/CollinPoetoehena/DevHub/blob/main/docs/GITHUB-ACTIONS.md) describes the intended consumer model; check its current status before adopting a workflow.
- [DevHub packages](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/README.md) lists the packages maintained there; check each package's README for its current capabilities and release status.

Other relevant [DevHub standards also apply](https://github.com/CollinPoetoehena/DevHub/blob/main/docs/USAGE.md#applying-devhub-standards-in-a-project).

---

## Portfolio and Project Structure

**You do not have to publish actual implementation code for this portfolio.** A high-level project structure can show how documentation, provisioning, configuration management and workload delivery connect. Combined with the generic design/setup explanations, it is sufficient for the intended architectural portfolio: reasoning, responsibilities and trade-offs matter more here than a concrete subnet value, credential or rendered manifest.

Withholding private code is deliberate, not an incomplete public repository. The [publication boundary](#publication-boundary) explains why implementation relationships can reveal sensitive home-network internals even after passwords are removed. Structure alone is not proof that a system is deployed or tests passed; describe evidence honestly without exposing private outputs.

Keep the single maintained overview at [Project Structure](./04-automation/project-structure.md#general-homelab-project-structure). Update it as a sanitized responsibility-level summary, **not raw `tree` output**. Include the main README, documentation and integration areas; omit device/application/environment listings and sensitive filenames. Label planned areas and do not invent directories. Reusable packages are independently maintained; any extraction or publication requires separate authorization.

---

## Documentation Structure & Guideline

> **Central documentation guideline:** This section is the authoritative place for documentation structure and writing conventions for both homelab repositories. Repository instructions, READMEs and other documentation reference it. Maintain the rules and their reasoning here rather than duplicating them.

### The Book Model: Layers and Storyline

The public documentation is designed to **read like a book from start to finish**: the open-source story of building and running a homelab. It is organized by the **reader's journey, not by document type**. Four documentation layers are read from top to bottom:

| Layer | Question | Parts |
|---|---|---|
| Foundations | Why are we doing it? | `01-foundations/` |
| Architecture | What are we building? | `02-architecture/`, `03-platform-services/` |
| Implementation | How is it realized (Ansible, Terraform, delivery)? | `04-automation/`, `05-services/` |
| Operations | How do we run it? | `06-operations/` |

Within those layers, the parts follow one storyline of seven questions: (1) why am I building this? (2) what principles and requirements guide it? (3) what architecture solves those requirements? (4) what capabilities make up that architecture? (5) how are those capabilities implemented? (6) what services run on it? (7) how is it operated, observed and recovered? The [book overview](./README.md) shows the layers, the storyline and the full table of contents.

Parts are numbered folders (`01-foundations/` to `06-operations/`) so that the file system shows the reading order. File names are lowercase kebab-case subjects (`network.md`, `backups.md`). The order is a reasoning path, not an inflexible waterfall: later chapters may feed requirements back to earlier ones, and chapters say so explicitly when they do.

### Why Not High-Level and Low-Level Design Folders

Earlier versions used `1_Goals/`, `2_HLD/` and `3_LLD/` folders (and before that, separate design and setup trees). That structure was retired because **HLD and LLD are levels of abstraction, not categories of subjects**. Using them as folders caused three recurring problems:

1. **Overlap.** Most subjects have both levels. Storage had a high-level design in the HLD and backup choices in the LLD; the network had its topology in one place and setup considerations in another; monitoring was filed as "low-level" although it has its own architecture. The same concept was explained—or half explained—in several places.
2. **No storyline.** A reader following one subject had to jump between folders and between `README`, `Design`, `Setup` and `Verification` files, and the documentation as a whole did not read from start to finish.
3. **Misleading categories.** Whether something counted as "high" or "low" level became a filing debate, and supporting files were added to compensate for the taxonomy instead of explaining the subject.

The high-level and low-level views have not disappeared: **the high-level view is the start of each subject chapter, and the low-level detail is its later sections**, followed by links to the executable implementation. A single subject therefore reads naturally as *why → requirements → design → detailed choices → implementation → verification → recovery*.

### One File per Subject

Use **one file per subject**—network, compute, storage, DNS, backups, observability and so on—even when that file becomes large. The network chapter is long on purpose: it contains everything about the network, from the choice of gateway arrangement down to switches, cabling and Wi-Fi, and that makes it a clear, complete story. A large, well-structured document with a table of contents is easier to read and maintain than a tree of small nested documents that each hold part of the picture:

- Readers follow one decision from its reason to its implementation checks without leaving the page.
- Maintainers know exactly where a change belongs, and duplicate explanations become obvious.
- Links are stable: anchors in one file replace fragile paths between many files.

Split a subject only when it has genuinely become two subjects with different responsibilities (for example, storage architecture versus backups), never to separate its high-level and low-level parts or its design from its setup. Avoid directories containing `README.md`, `Design.md` and `Setup.md` for one subject.

**Decisions belong to their subject.** There is no separate decisions folder. The option comparison, the reasoning and—in the private record—the actual choice live in the chapter of the subject they affect, so a decision is read in its context.

### Chapter READMEs as Entry Points

Each part folder has exactly **one `README.md`**, which is the entry point and introduction for that part. It explains which storyline question the part answers, lists its chapters in reading order with what each one decides, and describes how the part connects to the parts before and after it. Part READMEs may contain content that genuinely belongs to the part as a whole—such as the architecture overview or the automation ownership model—but subject detail belongs in the subject files.

Every chapter starts with a navigation line naming its part and linking to the part overview and the previous and next chapters, so the book can be read in order without returning to the table of contents.

### Subject Chapters and Cross-Cutting Chapters

Some chapters describe a **subject** (network, compute, storage); others describe a **cross-cutting** concern that applies to every subject (backups, disaster recovery, observability, maintenance, secrets). To avoid recreating the overlap of the HLD/LLD split, the cross-cutting chapter owns the *method* and the subject chapter owns its *specific application*. For example, [Backups](./06-operations/backups.md) owns backup tools, copies and retention, while the [network chapter](./02-architecture/network.md#backups-that-survive-the-outage) owns how gateway configuration is backed up; [Disaster Recovery](./06-operations/disaster-recovery.md) owns scenarios and recovery order, while each subject owns its own recovery options. Link between them instead of repeating either.

Chapters keep connected decisions together. Network architecture needs networking gear and management software; compute and storage design connects host capacity, local disks, electricity, virtualization and recovery. Splitting all hardware into one catalog and all software into another makes readers jump between related topics instead of following a decision through.

Likewise, there is **no separate public setup tree**. Each chapter starts with design and options, then ends with broad implementation/verification considerations. Think before installing. Setup is part of the subject's complete documentation, not a parallel book duplicating its structure. Exact commands and vendor installation details belong in official guides or private runbooks, not an imitation of my own deployment.

Automation is a cross-cutting principle across infrastructure and services, and also a part of the book: IaC and configuration-management choices (such as Terraform/OpenTofu/Pulumi and Ansible/Puppet) belong in [Part 4 · Automation](./04-automation/README.md), alongside the ownership contracts and workflows they support, and GitOps/CD and CI choices belong in its [deployment chapter](./04-automation/deployment.md). Tool names are not documentation folders. **Code describes how; documentation describes why, what and the relationships**: do not duplicate in prose what Ansible roles, Terraform modules or manifests already encode—link to them and explain their purpose, ownership and verification instead.

Start a public chapter from a homelab problem and provide enough context to follow the decision without another repository. Summarize the relevant concepts briefly, compare homelab-relevant options and trade-offs, and link to official standards or product documentation for deeper detail. Keep home/lab role examples, equipment choices and recovery implications here. When moving an explanation, update its navigation and incoming links instead of maintaining two copies.

Do not grow chapters into general theory references or vendor manuals. Cite the authoritative source where a concept or tool is discussed, especially for protocol details, supported behavior, configuration options and lifecycle claims. Use official standards bodies, project maintainers or product vendors when available; distinguish these source references from the homelab-specific reasoning in this book. The [Goals and Scope chapter](./01-foundations/goals-and-scope.md#define-what-is-out-of-scope) owns the scope rationale.

Implementation code does not have to be public for this portfolio. The [project-structure overview](./04-automation/project-structure.md) and generic guidance explain design, ownership and setup approaches without household inputs. This is an intentional [publication boundary](#portfolio-and-project-structure), not a promise to eventually mirror the private repository.

### Private Repository Structure

The [private repository](https://github.com/CollinPoetoehena/devhub-homelab-private) follows the **same book design**: the same layers, the same numbered parts, one `README.md` per part, one file per subject, and decisions inside the subject they affect. Its chapters use the same names as their public counterparts where both exist, so a reader can move between the public explanation and the private record of the same subject. Private parts and chapters exist only where there is actual private content; empty placeholder chapters are not created to mirror the public book.

The private repository is the **actual implementation**: real architecture, inventory, addressing, topology, chosen equipment and software, decisions and their rationale, exact non-sensitive inputs, runbooks, verification results and recovery records. It does not mirror the public prose. Each private subject chapter links the matching public chapter for the options and reasoning, then records the actual choice and moves from design to setup, verification and recovery in the same file. The [content ownership](#content-ownership) rules decide what goes where.

### Option Comparisons and Table Rationale

**Public design:** explain the problem, then put **all alternative/option comparisons in detailed Markdown tables**, not separate prose-only tool catalogs. Use the consistent columns **Option / tool**, **What it is**, **Benefits**, **Drawbacks / responsibilities**, and **Use cases / when to choose**. Link each named tool/product to its official documentation in the first column using its name as the link text, for example `[Talos Linux](https://docs.siderolabs.com/)`; generic patterns without an official documentation owner need no invented link. Split materially different tools into separate rows when their lifecycle or limitations differ.

**Why tables are the preferred format for comparisons:** readers can scan the available options, read across one row to understand an option, or read down one column to compare the same aspect across alternatives. Keeping benefits next to drawbacks and use cases makes trade-offs visible without jumping between scattered paragraphs or remembering each tool's explanation. The consistent columns also help newcomers distinguish what a tool actually does from why they might choose it, while experienced readers can quickly find the maintenance or compatibility constraint relevant to them.

This structure encourages balanced coverage: every option must explain both its value and its responsibilities, so a longer or more enthusiastic description does not accidentally become a recommendation. Missing use cases or limitations are easier to spot during review, and later updates can revise the relevant row without maintaining a second prose catalog. Official-documentation links provide a clear path from comparison to authoritative implementation detail. Tables are best **for this comparison task**, not for every kind of explanation: dependencies, reasoning sequences and verification still need prose. Keep cells readable, group large comparisons into focused tables and retain meaningful detail rather than trading understanding for compactness.

Preserve the existing explanatory depth in the cells: describe how each option works, the work it reduces, the ownership/limitations it adds and the requirements that make it appropriate. Do not compress a detailed comparison into slogans merely to fit a table. Group tables by the chapter's responsibility; use surrounding prose for shared principles, dependencies and evaluation/verification guidance rather than duplicating each row as a prose catalog. Factual measurement, quorum, role/policy and navigation tables may retain their purpose-specific columns because they are not option comparisons.

Include an explicit **Maintenance:** explanation in **Drawbacks / responsibilities for every option row**, not just selected tools or installation difficulty. Describe applicable updates, compatibility checks, configuration drift, credential handling, backups and tested recovery. For passive hardware, purchasing channels or conceptual choices, explain the limited direct upkeep and the ongoing responsibilities they leave with the operator rather than inventing a software lifecycle. Qualitative effort descriptions should explain what creates the work and which dependencies remain; they are not measured time estimates or universal product rankings. Keep initial setup effort distinct from recurring operations. Networking comparisons should have greater explanatory depth overall, but every chapter still requires maintenance coverage for every row.

Go deeper on host purchasing/virtualization and network boundaries without turning guidance into an exact private installation. Use role-based diagrams and clearly synthetic examples independent of the actual setup. There is no universal best hardware, Wi-Fi system or Kubernetes installer. Check official sources when revising claims, and distinguish release-specific behavior from stable concepts; do not claim a current release/support status without verification.

### Design, Implementation and Evidence

Within one subject file, design sections come first and setup, verification and recovery sections follow; the labels below describe those kinds of content, not separate files.

**Private design:** link the relevant public chapter, then record the choice, date when known, specific rationale, accepted costs/risks, and a short explanation of why a few relevant alternatives were not selected. Do not reproduce the full alternatives catalog. Include actual topology, capacities, dependencies and open decisions; product names are welcome where they clarify the real design.

**Public implementation considerations:** describe prerequisites, dependency order, configuration intent, validation and rollback broadly inside the chapter, normally in its final "Implementation and Verification" section. Link official installation guides rather than maintaining an exact walkthrough or revealing the environment that inspired it.

**Private setup:** in the later sections of the same subject file, link back to the design decisions instead of repeating selected values, and record exact verified non-sensitive inputs, versions, port/SSID maps, executed steps, deviations, results and recovery references. Sensitive endpoint values remain encrypted/protected under the [security guideline](#secrets-and-sensitive-artifacts), rather than plaintext in prose. An unverified example remains an example until someone confirms the actual value. Distinguish chosen, planned, installed, and verified; never infer deployment status from a document.

### Navigation, Links and Maintained Overviews

Add a **Table of Contents** for documents with sections, and `---` dividers between `##` sections. Start every chapter with the part navigation line described in [chapter READMEs as entry points](#chapter-readmes-as-entry-points), and update the neighbouring chapters' navigation lines and the [book overview](./README.md#table-of-contents-of-the-book) when adding, removing or reordering a chapter. Keep ToC links synchronized with headings.

Use relative links within one repository. **Between repositories, always use full GitHub URLs on `main`**, for example `https://github.com/CollinPoetoehena/devhub-homelab/blob/main/02-architecture/network.md`. Do not use sibling filesystem paths, feature branches, or commit-specific links for this shared documentation boundary.

When project-structure changes are in scope, update the final overview in [Project Structure](./04-automation/project-structure.md#maintaining-the-structure-summary-with-ai) according to that page's maintenance procedure; do not create another drifting summary.

The final documentation review TODO at the top of the [public README](./README.md) stays open until most of the homelab is complete and a thorough review has actually been performed. Use the experience to improve generic guidance, update documentation links and release-dependent claims, and recheck the [publication boundary](#publication-boundary); do not turn that review into a private deployment mirror or mark it complete after an individual document update.

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

Use this documented procedure for every explicitly authorized public preparation. No skill or tool is required; if a detail cannot be confidently generalized, leave it private and report the omission.

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
