# DevHub Homelab

> **TODO: Final documentation review.** Once most of the homelab is complete, thoroughly review the entire public documentation again. Use practical lessons to correct, expand and improve the generic guidance, not to publish the actual installation. Revisit goals, HLD/LLD categories, option tables, project structure and guidelines; verify official documentation links and update release-dependent features, support/lifecycle assumptions and recommendations to the current supported versions at review time. Recheck completeness, consistency, Table of Contents, anchors and cross-repository `main` links. Review the final public diff against the [publication boundary](./docs/Repository_Guidelines.md#publication-boundary), keeping actual internals, secrets and operational results private. This is a future review milestone, not a claim that the homelab or documentation is finished now.

**The public entry point, knowledge base, and portfolio for my homelab project.** This repository collects homelab-focused design guidance, trade-offs, setup approaches, and reusable lessons. It is not a description of my live home network or a deployable copy of it.

---

## Table of Contents

- [Why two homelab repositories?](#why-two-homelab-repositories)
- [Why separate from DevHub?](#why-separate-from-devhub)
- [How DevHub and the Homelab Repositories Fit Together](#how-devhub-and-the-homelab-repositories-fit-together)
- [Working Across DevHub and the Homelab Repositories](#working-across-devhub-and-the-homelab-repositories)
- [Task Tracking](#task-tracking)
- [Documentation](#documentation)
- [AI and Copilot Skills](#ai-and-copilot-skills)
  - [AI usage](#ai-usage)

---

## Why two homelab repositories?

The primary reason is **security**. Publishing the internals of a home network can help someone identify devices, locate management services, understand access paths, or target household infrastructure. Credentials are not the only sensitive information: a combination of real topology, addressing, inventory, firewall exceptions, software versions, and operational details can be revealing too. The public repository therefore describes generic patterns, not a redacted map of my actual network.

| Repository | Purpose | What belongs there |
|---|---|---|
| [devhub-homelab](https://github.com/CollinPoetoehena/devhub-homelab) (public) | Homelab knowledge base, portfolio, and entry point | Options, selection criteria, generic designs, sanitized setup guidance, reusable lessons, and links to reusable components |
| [devhub-homelab-private](https://github.com/CollinPoetoehena/devhub-homelab-private) (private; access required) | Actual implementation and operational record | Chosen hardware/software, decision rationale, internal topology and inventory, configuration, runbooks, validation results, and recovery details |

For a convenient local setup with all three projects, see [Working Across DevHub and the Homelab Repositories](#working-across-devhub-and-the-homelab-repositories).

One repository forced an awkward compromise: either publish too much, or replace useful operational details with examples and leave the real setup undocumented in my head. The split lets the private record be genuinely useful while the public reference remains safe and reusable.

Two repositories do add a little coordination: links must stay correct and a change may need documentation in both places. But **they make the content easier to maintain**. Public guidance owns the explanations and option comparisons once. Private designs link to that guidance and concentrate on what I chose, why, which relevant alternatives I rejected, and how the actual pieces fit together. There is no need to repeat a full Kubernetes distribution comparison inside my cluster decision record.

The public side can stay curated, concise, and focused on material worth sharing. The private side can contain detailed working notes, incomplete investigations, exact configurations, and recovery procedures without making every note a portfolio item. This removes the pressure to sanitize every operational record or polish everything for an external audience.

**Private does not mean safe to commit secrets.** Both repositories still need deliberate access controls, protected secret storage, patching, and backups. Separation reduces disclosure risk; it is not a substitute for network security. Public changes are explicit publications, never an automatic mirror or a copy of private Git history.

See [Repository guidelines](./docs/Repository_Guidelines.md) for content ownership, publication review, and secret handling.

---

## Why separate from DevHub?

The [DevHub separate-projects explanation](https://github.com/CollinPoetoehena/DevHub#separate-projects) describes the distinction: a homelab is related to development, but is still a distinct project with its own purpose, design, lifecycle, issues, automation, and maintenance.

The split is about **reusable principles versus their application**, not theory versus design. DevHub covers general architecture as well as fundamentals; this repository applies those ideas to homelabs. The [division of responsibilities below](#how-devhub-and-the-homelab-repositories-fit-together) explains where to read and maintain each kind of information.

---

## How DevHub and the Homelab Repositories Fit Together

**DevHub explains the reusable principle; the public homelab explains how to apply it at home scale; the private homelab records what I actually chose and implemented.** Keep one authoritative explanation of each subject and link to it from the places that use it.

| Location | Owns | Examples |
|---|---|---|
| [DevHub reference](https://github.com/CollinPoetoehena/DevHub/tree/main/reference) | General theory, architectural principles, commands, and workflows useful across projects and environments | Networking layers, segmentation, trust zones, permitted traffic flows, OS/hardware fundamentals, and general development practices |
| Public homelab (this repository) | Homelab applications, generic example designs, options, and practical trade-offs | Shared versus separate home/lab gateways, isolated routing exercises, small-host purchasing and capacity, power/noise constraints, equipment placement, and home-scale recovery |
| [Private homelab](https://github.com/CollinPoetoehena/devhub-homelab-private) (access required) | Actual selected design, implementation, and operations | Equipment and topology, confirmed internal settings, decision rationale, runbooks, verification results, and recovery records |

For example, [DevHub's network design reference](https://github.com/CollinPoetoehena/DevHub/blob/main/reference/network/Network_Design.md) owns the detailed segmentation/access explanation and illustrative trust-zone model. The [public network guide](./docs/2_HLD/Network/README.md) applies those principles to keeping household connectivity independent of lab experiments, selecting a gateway/switch/AP arrangement, and exposing a lab application deliberately. The private network category records the chosen configuration and what was actually verified. A reader designing a business or cloud network should not need a homelab project as the authoritative source for general network architecture.

The same rule applies to hardware and other topics. General [OS and hardware references](https://github.com/CollinPoetoehena/DevHub/tree/main/reference/os_hardware) explain the underlying concepts and inspection commands; [Hosts and Nodes](./docs/2_HLD/Hosts/README.md) evaluates their use for small lab machines, capacity, electricity, and accepted failure domains. Actual inventory, allocations, and operational results stay private. Keep reusable [AI customizations](https://github.com/CollinPoetoehena/DevHub/blob/main/.github/AI.md) and [packages](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/README.md) in their own shared lifecycle rather than copying them into the homelab.

Brief contextual summaries and safety reminders are useful in an applied guide; a second full tutorial, comparison, or identical general-purpose diagram is not. Link to the authoritative explanation, then add the homelab-specific requirements, reasoning, constraints, and checks. Cross-links may work in both directions to help readers navigate, but they do not make either repository a mirror of the other. The private record can link directly to DevHub when the relevant material is general rather than a homelab example.

The [repository guidelines](./docs/Repository_Guidelines.md#content-ownership) turn this relationship into maintenance and publication rules. This separation does not authorize moving private configuration into either public repository.

---

## Working Across DevHub and the Homelab Repositories

**For my own workflow, keeping DevHub, the public homelab and the private homelab together in one VS Code workspace is the easiest and most practical setup.** I can move between related projects in one window, compare their documentation side by side and make coordinated updates without repeatedly opening separate workspaces. Clone [DevHub](https://github.com/CollinPoetoehena/DevHub), [devhub-homelab](https://github.com/CollinPoetoehena/devhub-homelab) and [devhub-homelab-private](https://github.com/CollinPoetoehena/devhub-homelab-private) under a shared parent folder, such as `homelab/`, and open that folder in VS Code. Alternatively, add the three repositories to a multi-root workspace. Each clone remains its own Git repository.

This setup is also useful when working with AI: workspace-aware tools can access relevant shared principles in DevHub, generic homelab guidance and private design records—even when a task changes only one repository. That context helps the AI follow the right ownership boundaries, reuse existing explanations and keep related documentation consistent. It can avoid effort and AI usage spent switching workspaces or retrieving context through separate GitHub URLs; private content may not be accessible through a URL at all, and remote retrieval can require extra steps and tokens. Local availability does not mean every file is automatically read, however: tools differ in how they access the workspace, so point the AI at the target repository, relevant files and applicable instructions.

Opening an individual repository in its own workspace is still possible for deliberately isolated work, but it is not the recommended default. The AI may not be able to inspect another workspace, and links to other repositories are not a reliable substitute for local context. Keeping all three repositories together makes that context available when needed, even if the other two are not being changed. Preserve their information boundaries: DevHub owns reusable principles, the public homelab owns generic homelab applications, and the private homelab owns actual design and operations. Private access is not permission to publish private details; public changes still require deliberate review against the [publication boundary](./docs/Repository_Guidelines.md#publication-boundary). See [AI usage](#ai-usage) for making shared and homelab-specific customizations available across the workspace.

---

## Task Tracking

**Why track tasks, and what is the goal?** Keep useful ideas, unfinished work and maintenance follow-ups out of my head, so I can return to the homelab without reconstructing what I was doing. The goal is **minimal effort, but enough structure to help**: each task has a clear next action, a short description of what completion means and a visible status. That makes it easier to choose something manageable, see what is already in progress and avoid forgetting small fixes or starting the same investigation twice. A board is more useful here than a growing list in a Word document or scattered notes because tasks can move through consistent statuses and link directly to their supporting issues, documentation and changes. Keep detailed design and runbooks in the repository rather than copying them into task descriptions. Capture tasks briefly, update them when work changes and close them after checking the result; no exhaustive backlog, reporting routine or elaborate prioritization system is needed. The board should support learning and steady progress, not turn a hobby into project-management work.

**Why GitHub Projects?** It is free to use for this task-tracking setup, so no paid planning subscription is needed. The repositories already live on GitHub, so task tracking stays close to the documentation and implementation it describes. Issues provide descriptions and discussion, pull requests can link the related work, and a project brings tasks from both repositories into one overview. This avoids a separate planning tool, another account or self-hosted service to maintain, and copying or synchronizing task details between systems. GitHub hosts the tool; I only need to maintain the tasks, statuses and access settings. For a personal homelab, that low overhead makes it easier to keep the board useful instead of spending time administering the planning system.

Track **all homelab tasks, both public and private**, in one **private** GitHub Project named **Homelab**: [Backlog · Homelab](https://github.com/users/CollinPoetoehena/projects/7), linked to [devhub-homelab-private](https://github.com/CollinPoetoehena/devhub-homelab-private) and not the public repository because it should remain private. This includes improvements to [devhub-homelab](https://github.com/CollinPoetoehena/devhub-homelab), implementation, maintenance and operational work. Project access is required.

Both repositories belong to the same homelab, so one board avoids splitting related work or coordinating two backlogs. Tasks may contain private information, and the planning is only for myself, so publishing it is unnecessary even when the resulting documentation is public. It is a hobby, not a team delivery process; sprints, estimates, deadlines and formal planning are unnecessary. Technical design and safe-change checks still matter.

I used GitHub's existing **Team Planning** template because it already has the main task-tracking aspects I wanted. I do not need to use everything it provides: extra views and sprint/current-iteration features can remain unused, or serve as a loose grouping of current work without committing to a time-based schedule. One status board remains the everyday workflow.

> **Warning: explicitly set the GitHub Project's visibility to Private before adding tasks.** Linking it to a private repository does not make the project private automatically. A private project also does not hide public issues: keep private details in private-repository issues or private project draft items, never in public issues. Follow the [publication boundary](./docs/Repository_Guidelines.md#publication-boundary); secrets still belong in [protected storage](./docs/Repository_Guidelines.md#secrets-and-sensitive-artifacts).

1. Start from the **Team Planning** template, keep the GitHub Project private and use one **Board** view for everyday tracking. Link it to the private repository. In the project description, link to [this Task Tracking explanation](https://github.com/CollinPoetoehena/devhub-homelab/blob/main/README.md#task-tracking) so the workflow is easy to find.
2. Use a **Status** field with **TODO**, **In progress** and **Done**. Add an issue or draft item for each useful task, with a short description of what completion means. Public-repository issues may be added when their contents are safe to publish; keep other tasks private.
3. Pick a TODO when ready, move it to In progress, then close the issue (if applicable) and move the item to Done after checking the result. Keep ideas in TODO; no extra boards or planning ceremony are needed.

> **Central reference:** The [private homelab README](https://github.com/CollinPoetoehena/devhub-homelab-private/blob/main/README.md#task-tracking) references this section instead of repeating the task-tracking explanation. Both repositories use the same private project; there is no separate public planning board. Only the workflow explanation and safe documentation are published.

---

## Documentation

- [Repository guidelines](./docs/Repository_Guidelines.md): public/private boundary, writing rules, sanitization, and security.
- [Documentation storyline](./docs/README.md): the entry point and reading order.
- [Project structure](./docs/Project_Structure.md): integration responsibilities, package reuse and an AI-maintained sanitized overview.
- [Goals](./docs/1_Goals/README.md): purpose, constraints and household dependencies.
- [High-level design](./docs/2_HLD/README.md): network, Wi-Fi, host selection, power/capacity, virtualization and host automation.
- [Low-level design](./docs/3_LLD/README.md): container orchestration, services/delivery, monitoring, storage and backups.
- [Local Copilot skills](.github/skills/README.md): the homelab-specific sanitization workflow; generic skills come from DevHub.

Examples are synthetic and must be adapted to the reader's requirements. A documented design or checklist is not evidence of a completed deployment. The private repository references these documents through GitHub `blob/main` URLs; access to it is not required to use the public guidance.

---

## AI and Copilot Skills

> **Central reference:** This section is the single place for the homelab repositories' shared-skill mapping and AI usage guidance. The public and private `.github/skills/README.md` files intentionally link here instead of duplicating this information; other README references should do the same.

Use [DevHub's AI Customizations guide](https://github.com/CollinPoetoehena/DevHub/blob/main/.github/AI.md) as the source of general explanations, customization choices, design principles and setup guidance. Only add local customizations for genuinely homelab-specific needs, not wrappers around generic documentation or implementation workflows.

The main shared skills used in this repository are:

| DevHub skill | Use here |
|---|---|
| [devhub-docs](https://github.com/CollinPoetoehena/DevHub/blob/main/.github/skills/devhub-docs/SKILL.md) | Write and maintain public homelab documentation using discovered local guidelines and structure |
| [devhub-generic-change](https://github.com/CollinPoetoehena/DevHub/blob/main/.github/skills/devhub-generic-change/SKILL.md) | Implement and verify generic changes using discovered project conventions |
| [devhub-terraform-change](https://github.com/CollinPoetoehena/DevHub/blob/main/.github/skills/devhub-terraform-change/SKILL.md) | Authorized changes to genuinely reusable Terraform examples/components, not importing or deploying private roots |
| [devhub-ansible-change](https://github.com/CollinPoetoehena/DevHub/blob/main/.github/skills/devhub-ansible-change/SKILL.md) | Authorized changes to genuinely reusable Ansible examples/components, not importing private inventories or targeting live hosts |

These skills are maintained and installed through DevHub; they do **not** need to be present in this project. Choose the matching skill directly rather than invoking the generic change skill before a tool-specific one. Public automation topics normally remain Markdown explanations; listing these skills does not authorize code extraction or live operations.

A project-context skill is unnecessary: DevHub skills automatically discover instructions, documentation structure, guidelines and relevant validation commands. Keep any additional persistent homelab context in [copilot-instructions.md](./.github/copilot-instructions.md). The only [local skill](./.github/skills/README.md) is [homelab-sanitize](./.github/skills/homelab-sanitize/SKILL.md), for explicitly authorized publication preparation.

**Do not chain skills.** Each skill must be self-contained and usable without another skill being invoked first or afterward. Background and project-policy links are fine; required cross-skill invocation is not. See [DevHub's design principles](https://github.com/CollinPoetoehena/DevHub/blob/main/.github/AI.md#general-design-principles) and its [skill design explanation](https://github.com/CollinPoetoehena/DevHub/blob/main/.github/skills/README.md#create-a-skill) for why dependency chains are unreliable and how to keep skills self-contained.

**Documentation owns project policy.** The [repository guidelines](./docs/Repository_Guidelines.md), including the [Documentation Structure & Guideline](./docs/Repository_Guidelines.md#documentation-structure--guideline), remain authoritative for both homelab repositories. Reference them rather than duplicating their rules in skills. AI must read applicable guidance, preserve the publication boundary, distinguish planned work from verified results and validate changes. Publication and live infrastructure operations still require explicit authorization.

### AI usage

Install the shared DevHub skills using [Use external or centralized customizations across repositories](https://github.com/CollinPoetoehena/DevHub/blob/main/.github/AI.md#use-external-or-centralized-customizations-across-repositories), which also links the DevHub installer and explains manual copies and symbolic links. A GitHub link is a reference, not an installation. Verify discovery in the selected host, on the machine where the agent runs.

For a shared parent workspace as described in [Working Across DevHub and the Homelab Repositories](#working-across-devhub-and-the-homelab-repositories), you can add a workspace-local `.github/skills/` directory and symbolic links to the **additional homelab-specific** skill directories. For example, from that parent folder on a host supporting symlink discovery:

```sh
mkdir -p .github/skills
ln -s /absolute/path/to/homelab/devhub-homelab/.github/skills/homelab-sanitize .github/skills/homelab-sanitize
```

Replace the absolute source path with your clone location, ensure the destination does not already exist, and link the complete skill directory so its resources remain available. Keep the clone at that path. Add links for future genuinely homelab-specific skills in the same way; no private-specific skill is currently needed. Do not commit these machine-local workspace links into either repository. If the host does not discover symlinks, use the supported copy/setup options in the DevHub guide instead.

**Centralizing homelab-specific customizations is not necessary.** They are specific to my homelab projects, unlike the generic DevHub customizations reused across unrelated projects. Keep their source in the owning repository; parent-workspace links are optional convenience, not a reason to move them into DevHub or install them globally.

Opening the parent folder does not guarantee discovery of nested repository instructions. Explicitly select the target repository and provide its [public instructions](./.github/copilot-instructions.md) or [private instructions](https://github.com/CollinPoetoehena/devhub-homelab-private/blob/main/.github/copilot-instructions.md) as context when necessary. Verify that the intended skills are available. Access to all three repositories never authorizes publishing private content.
