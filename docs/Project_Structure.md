# Homelab Project Structure

A useful structure makes responsibilities and dependencies understandable without publishing site-specific implementation code. This page describes how to organize homelab integration and ends with the maintained high-level structure summary for this project.

---

## Table of Contents

- [Structure by Responsibility](#structure-by-responsibility)
- [Reuse Packages Rather Than Copying Them](#reuse-packages-rather-than-copying-them)
- [Portfolio Without Publishing Implementation Code](#portfolio-without-publishing-implementation-code)
- [Maintaining the Structure Summary with AI](#maintaining-the-structure-summary-with-ai)
- [General Homelab Project Structure](#general-homelab-project-structure)

---

## Structure by Responsibility

Start with a root README explaining purpose, documentation entry points and operational boundaries. Keep goals, HLD and LLD together under documentation, with design before implementation/results in each category. Code can follow its tool's ecosystem conventions without turning tool names into competing documentation storylines.

Separate provisioning, host configuration and workload reconciliation because they have different inputs, credentials and lifecycles. Define which owns each property. Keep environment-specific composition close to its consumer, reusable components independently maintained, and secrets/state/generated outputs outside ordinary tracked content.

Prefer a small number of meaningful folders over a directory for every idea. Split when ownership or lifecycle genuinely differs. Document prerequisites, input/output contracts and validation beside the relevant integration. Avoid duplicating inventory across several tools without a declared authority and a deliberate handoff.

---

## Reuse Packages Rather Than Copying Them

[DevHub](https://github.com/CollinPoetoehena/DevHub) remains the shared development/reference hub. Its [package guidelines](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/README.md) explain independently versioned reusable components, distribution and shared publishing conventions.

- Use [Ansible roles](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/Ansible.md) for reusable host/platform responsibilities; homelab playbooks/inventory compose them.
- Use [Terraform modules](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/Terraform.md) behind a defined input/output contract; homelab roots supply the private environment configuration and protected backend.
- Use [Helm charts](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/Helm.md) for reusable application packaging where appropriate; deployment configuration and environment values remain integration concerns.

Pin compatible versions, review upgrades and test the consumer. A reusable component should not embed a household subnet, endpoint or credential. Existing locally housed components can remain until a separately authorized extraction; this structure guide does not move or publish their code.

---

## Portfolio Without Publishing Implementation Code

Publishing actual code is **not required for this homelab portfolio**. The purpose is to explain how the system is designed, how responsibilities connect and how a generic setup is approached. A sanitized structure, options/trade-offs and validation methodology can communicate that without revealing which subnet, secret or live endpoint the implementation uses.

The [publication boundary](./Repository_Guidelines.md#publication-boundary) explains why even password-free inventory/configuration can expose home-network internals. Keep implementation details private; share genuinely reusable packages only through their own review and lifecycle. Do not misrepresent a structure diagram as proof of deployment, test results or code availability.

---

## Maintaining the Structure Summary with AI

**Update the final section of this page in place** when documentation or integration responsibilities change. Use [homelab-public-docs](../.github/skills/homelab-public-docs/SKILL.md) and [homelab-sanitize](../.github/skills/homelab-sanitize/SKILL.md). Do not create another drifting structure overview elsewhere.

Inspect only the authorized, relevant top-level structure locally. Summarize responsibility-level folders rather than pasting `tree` output or enumerating hosts, inventories, applications, environment names, secret filenames or generated artifacts. Review the summary against the guideline before publication.

Mark absent/planned areas explicitly. AI must not create implementation folders, claim deployments or infer live values just to make the diagram look complete. Keep the summary at the bottom of this document; earlier sections explain the general conventions.

---

## General Homelab Project Structure

This is an **AI-maintained, sanitized responsibility view**, not an exact directory listing. The public repository contains documentation/customizations; implementation areas belong privately. Paths describe the main organization, not a deployable copy.

```text
Public homelab knowledge base
  README.md                 purpose, split and entry point
  .github/                  public-context instructions and skills
  docs/
    Repository_Guidelines.md
    Project_Structure.md     this maintained overview
    1_Goals/                generic purpose and constraints
    2_HLD/                  network/Wi-Fi, hosts and host automation
    3_LLD/                  orchestration, delivery, monitoring, data protection

Private homelab integration
  README.md                 actual project context and navigation
  .github/                  private-context instructions and skills
  docs/                     matching goals/HLD/LLD categories;
                            actual decisions, setup, checks and recovery
  ansible/                  host/platform configuration composition;
                            reusable roles and protected variable references
  terraform/                provisioning composition and reusable modules;
                            protected runtime inputs and state
  GitOps integration        planned responsibility for workload reconciliation;
                            directory/repository layout not yet established
```

The GitOps line is a planned area, not an existing `gitops/` folder or verified deployment. Reusable packages may have their own repositories linked through DevHub. Credentials, decrypted values, state/plans and raw backups are intentionally not represented as public paths.
