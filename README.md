# DevHub Homelab

> **TODO: Final documentation review.** Once most of the homelab is complete, thoroughly review the entire public documentation again. Use practical lessons to correct, expand and improve the generic guidance, not to publish the actual installation. Revisit the book structure (layers, parts and chapters), goals, requirements, option tables, project structure and guidelines; verify official documentation links and update release-dependent features, support/lifecycle assumptions and recommendations to the current supported versions at review time. Recheck completeness, consistency, Table of Contents, anchors and cross-repository `main` links. Review the final public diff against the [publication boundary](./docs/repository-guidelines.md#publication-boundary), keeping actual internals, secrets and operational results private. This is a future review milestone, not a claim that the homelab or documentation is finished now.

**The public entry point, knowledge base and portfolio for my homelab project—written as a book you can read from start to finish.** This repository is the open-source version of my homelab journey: why I build a homelab, which requirements and principles guide it, which architecture and capabilities solve them, how it is implemented and how it is operated and recovered. It collects homelab-focused design guidance, trade-offs, recommendations, setup approaches and reusable lessons. It is not a description of my live home network or a deployable copy of it.

**Start reading:** [The Homelab Book](./docs/README.md).

---

## Table of Contents

- [Why two homelab repositories?](#why-two-homelab-repositories)
- [Reading the Public Repository as a Book](#reading-the-public-repository-as-a-book)
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
| [devhub-homelab](https://github.com/CollinPoetoehena/devhub-homelab) (public) | **The homelab book:** the open-source storyline of my homelab journey and of a homelab in general, designed to be read from start to finish; knowledge base, portfolio and entry point | Goals, requirements and principles; options, selection criteria and trade-offs; generic reference designs; recommendations and hints about what I would choose; sanitized setup guidance, reusable lessons and links to reusable components. **Never** the exact architecture or low-level details that could help an attacker—they are not needed to tell the story |
| [devhub-homelab-private](https://github.com/CollinPoetoehena/devhub-homelab-private) (private; access required) | **The actual implementation** behind the book, and its operational record | The real architecture, chosen hardware/software, decision rationale, internal topology, addressing and inventory, Ansible/Terraform code and configuration, runbooks, validation results and recovery details—organized in the same book structure |

For a convenient local setup with all three projects, see [Working Across DevHub and the Homelab Repositories](#working-across-devhub-and-the-homelab-repositories).

One repository forced an awkward compromise: either publish too much, or replace useful operational details with examples and leave the real setup undocumented in my head. The split lets the private record be genuinely useful while the public reference remains safe and reusable.

Two repositories do add a little coordination: links must stay correct and a change may need documentation in both places. But **they make the content easier to maintain**. Public guidance owns the explanations and option comparisons once. Private designs link to that guidance and concentrate on what I chose, why, which relevant alternatives I rejected, and how the actual pieces fit together. There is no need to repeat a full Kubernetes distribution comparison inside my cluster decision record.

The public side can stay curated and focused on material worth sharing, and can be shaped as a readable story from start to finish. It may give hints about what I recommend or would choose, but it never exposes my exact architecture or low-level details such as addressing, inventory, firewall exceptions or management paths: an attacker could use them, and the story does not need them. The private side can contain detailed working notes, incomplete investigations, exact configurations, and recovery procedures without making every note a portfolio item. This removes the pressure to sanitize every operational record or polish everything for an external audience.

**Private does not mean safe to commit secrets.** Both repositories still need deliberate access controls, protected secret storage, patching, and backups. Separation reduces disclosure risk; it is not a substitute for network security. Public changes are explicit publications, never an automatic mirror or a copy of private Git history.

See [Repository guidelines](./docs/repository-guidelines.md) for content ownership, publication review, and secret handling.

---

## Reading the Public Repository as a Book

The public documentation is deliberately designed as a **book with a storyline**, not as a collection of reference pages. It follows four documentation layers—**Foundations** (why are we doing it?), **Architecture** (what are we building?), **Implementation** (how is it realized, with Ansible, Terraform and delivery workflows?) and **Operations** (how do we run it?)—and, within them, seven questions: why build it, which principles and requirements guide it, which architecture solves them, which capabilities make it usable, how it is implemented, which services run on it, and how it is operated, observed and recovered.

| Part | Layer | What it covers |
|---|---|---|
| [1 · Foundations](./docs/01-foundations/README.md) | Foundations | Goals and scope, requirements, design principles, cost and buying decisions |
| [2 · Architecture](./docs/02-architecture/README.md) | Architecture | Physical environment, network (including Wi-Fi), compute, storage, workload platform |
| [3 · Platform Services](./docs/03-platform-services/README.md) | Architecture | DNS, certificates, identity and access, secrets, scheduling and power |
| [4 · Automation](./docs/04-automation/README.md) | Implementation | Infrastructure as code, configuration management, deployment, project structure |
| [5 · Services](./docs/05-services/README.md) | Implementation | Workload design and lifecycle, self-hosted service options |
| [6 · Operations](./docs/06-operations/README.md) | Operations | Observability, backups, disaster recovery, maintenance |

Each part has one README as its entry point, and each subject—such as the network—is one chapter that moves from high-level design to low-level detail, implementation and verification. Earlier versions split the documentation into high-level design (HLD) and low-level design (LLD) folders; that created overlapping documents for the same subject and no readable story, so it was replaced. The [book overview](./docs/README.md) explains how to read it, and the [Documentation Structure & Guideline](./docs/repository-guidelines.md#documentation-structure--guideline) explains the design and its rules.

The private repository uses the same structure for the actual implementation, so each public chapter has a matching private record where one exists. The public book can say *what I recommend and why*; the private record says *exactly what was built*.

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
| [devhub-sdks](https://github.com/CollinPoetoehena/devhub-sdks) | Independently versioned Python and Go packages for reusable integrations | SDK packaging and release foundations; concrete capabilities are added from consumer needs, not copies of private homelab configuration |
| Public homelab (this repository) | Homelab applications, generic example designs, options, and practical trade-offs | Shared versus separate home/lab gateways, isolated routing exercises, small-host purchasing and capacity, power/noise constraints, equipment placement, and home-scale recovery |
| [Private homelab](https://github.com/CollinPoetoehena/devhub-homelab-private) (access required) | Actual selected design, implementation, and operations | Equipment and topology, confirmed internal settings, decision rationale, runbooks, verification results, and recovery records |

For example, [DevHub's network design reference](https://github.com/CollinPoetoehena/DevHub/blob/main/reference/network/Network_Design.md) owns the detailed segmentation/access explanation and illustrative trust-zone model. The [public network guide](./docs/02-architecture/network.md) applies those principles to keeping household connectivity independent of lab experiments, selecting a gateway/switch/AP arrangement, and exposing a lab application deliberately. The private network chapter records the chosen configuration and what was actually verified. A reader designing a business or cloud network should not need a homelab project as the authoritative source for general network architecture.

The same rule applies to hardware and other topics. DevHub's [compute hardware fundamentals](https://github.com/CollinPoetoehena/DevHub/tree/main/reference/compute) and [operating-system references](https://github.com/CollinPoetoehena/DevHub/tree/main/reference/os) explain the underlying concepts and inspection commands; the [compute and storage chapters](./docs/02-architecture/README.md#compute-and-storage-together) apply them to compute capacity, storage, electricity, and accepted failure domains. Actual inventory, allocations, and operational results stay private. Keep reusable [AI customizations](https://github.com/CollinPoetoehena/DevHub/blob/main/.github/AI.md) and [packages](https://github.com/CollinPoetoehena/DevHub/blob/main/reference/engineering/Packages.md) in their own shared lifecycle rather than copying them into the homelab.

Brief contextual summaries and safety reminders are useful in an applied guide; a second full tutorial, comparison, or identical general-purpose diagram is not. Link to the authoritative explanation, then add the homelab-specific requirements, reasoning, constraints, and checks. Cross-links may work in both directions to help readers navigate, but they do not make either repository a mirror of the other. The private record can link directly to DevHub when the relevant material is general rather than a homelab example.

The [repository guidelines](./docs/repository-guidelines.md#content-ownership) turn this relationship into maintenance and publication rules. This separation does not authorize moving private configuration into either public repository.

---

## Working Across DevHub and the Homelab Repositories

**My recommended default is one focused VS Code workspace containing DevHub, the SDKs, the public homelab and the private homelab.** I can move between shared guidance, reusable code and its consumers in one window, compare their documentation side by side and make coordinated updates without repeatedly opening separate workspaces. Clone [DevHub](https://github.com/CollinPoetoehena/DevHub), [devhub-sdks](https://github.com/CollinPoetoehena/devhub-sdks), [devhub-homelab](https://github.com/CollinPoetoehena/devhub-homelab) and [devhub-homelab-private](https://github.com/CollinPoetoehena/devhub-homelab-private) under a shared parent folder, such as `homelab/` or `devhub` or a different name if you prefer that name, and open that folder in VS Code. Alternatively, add those repositories to a multi-root workspace. Each clone remains its own Git repository; private access is not required to read or work with the public projects.

**Everything beyond that default set is optional.** Keep only these repositories for a focused workspace, or add consuming applications, separately maintained Ansible roles or collections, Terraform modules, Helm charts, and shared tooling when their context helps the work. Choose whichever set suits the task and your preferences; there is no requirement to clone every related project. Additional repositories can make it easier to inspect a dependency's public contract and coordinate changes without copying its implementation into the homelab.

This shared workspace is useful for work across these related projects: AI tools can use relevant local context to help maintain consistency, and I can avoid repeatedly switching windows or retrieving guidance. I can still open a repository individually for isolated work. See DevHub's [guidance on grouping related repositories in VS Code workspaces](https://github.com/CollinPoetoehena/DevHub/blob/main/reference/Code_Editor.md#grouping-related-repositories-in-vs-code-workspaces) for the broader rationale, examples and trade-offs.

Preserve the repositories' information boundaries: DevHub owns reusable principles, the SDK and other component repositories own reusable implementations, the public homelab owns generic homelab applications, and the private homelab owns actual design and operations. Sharing a workspace does not merge their Git histories, release lifecycles, or permissions. Private access is not permission to publish private details; public changes still require deliberate review against the [publication boundary](./docs/repository-guidelines.md#publication-boundary). See [AI usage](#ai-usage) for making shared and homelab-specific customizations available across the workspace.

---

## Task Tracking

Start with [Goals and Scope](./docs/01-foundations/goals-and-scope.md): tasks should serve the reasons for having a homelab, not become goals in themselves. In particular, keep [relaxation and enjoyment over productivity and pressure](./docs/01-foundations/goals-and-scope.md#relaxation-and-enjoyment-over-productivity-and-pressure) in mind; the board is there to make it easier to return to interesting work, not to create pressure to finish everything quickly. If it helps keep the lab from taking over free time, use [optional timeboxes](./docs/01-foundations/goals-and-scope.md#timebox-without-turning-it-into-a-deadline) as a gentle start/stop boundary, not as a deadline or task quota.

**Why track tasks, and what is the goal?** Keep useful ideas, unfinished work and maintenance follow-ups out of my head, so I can return to the homelab without reconstructing what I was doing. The goal is **minimal effort, but enough structure to help**: each task has a clear next action, a short description of what completion means and a visible status. That makes it easier to choose something manageable, see what is already in progress and avoid forgetting small fixes or starting the same investigation twice. It also makes the work more deliberate: prioritize what matters now, break larger efforts into manageable tasks and work from a clear next action instead of jumping into something without a plan. If another idea comes up, capture it as a TODO and return to it later rather than abandoning the task at hand. A board is more useful here than a growing list in a Word document or scattered notes because tasks can move through consistent statuses and link directly to their supporting issues, documentation and changes. Keep detailed design and runbooks in the repository rather than copying them into task descriptions. Capture tasks briefly, update them when work changes and close them after checking the result; no exhaustive backlog, reporting routine or elaborate prioritization system is needed. The board should support learning and steady progress, not turn a hobby into project-management work.

**Personal note: why I needed this structure.** I initially kept my homelab ideas and things to do in a Word document. Writing them down helped capture them, but the way I used that document did not give me a clear structure for prioritizing work, splitting larger efforts into smaller tasks or recording where I had left off. I often just started doing something without first deciding its scope or next steps. Once I started, I felt unnecessary mental pressure to finish: stopping meant leaving an unclear piece of work that I would have to reconstruct later. The problem was not Word itself, but the lack of a task-tracking workflow I could trust. Small tasks, visible statuses and a short note about the next step give unfinished work a clear place to wait. That makes it easier to pause deliberately, capture a new idea for later and return to one manageable task without feeling that I need to finish the whole effort in one sitting. The purpose of this structure is to reduce pressure, not add more obligations.

This is a lightweight application of the thinking behind DevHub's [software development lifecycle (SDLC)](https://github.com/CollinPoetoehena/DevHub/blob/main/reference/engineering/Concepts.md#software-development-lifecycle): understand the goal, consider the approach, break implementation into manageable tasks, validate the result and feed follow-ups back into planning. These activities can overlap and repeat; they do not require a rigid sequence, sprints or deadlines. The board helps me choose and track the work, while the repository keeps the supporting design, implementation and verification context.

If it helps keep the hobby in balance, set an **optional timebox** for when you might do lab work—for example, a couple of hours on a chosen weekend morning, or a flexible window that varies each week. This is a boundary on when the lab gets your attention, not a commitment to attend or a quota to finish. Skip a planned session if you would rather do something else, and stop when the time is up even if the task remains open. Leave a short resumption note in the task comments or a document so the next session can start easily. See [Timebox Without Turning It into a Deadline](./docs/01-foundations/goals-and-scope.md#timebox-without-turning-it-into-a-deadline).

**Why GitHub Projects?** It is free to use for this task-tracking setup, so no paid planning subscription is needed. The repositories already live on GitHub, so task tracking stays close to the documentation and implementation it describes. Issues provide descriptions and discussion, pull requests can link the related work, and a project brings tasks from both repositories into one overview. This avoids a separate planning tool, another account or self-hosted service to maintain, and copying or synchronizing task details between systems. GitHub hosts the tool; I only need to maintain the tasks, statuses and access settings. For a personal homelab, that low overhead makes it easier to keep the board useful instead of spending time administering the planning system.

Track **all homelab tasks, both public and private**, in one **private** GitHub Project named **Homelab**: [Backlog · Homelab](https://github.com/users/CollinPoetoehena/projects/7), linked to [devhub-homelab-private](https://github.com/CollinPoetoehena/devhub-homelab-private) and not the public repository because it should remain private. This includes improvements to [devhub-homelab](https://github.com/CollinPoetoehena/devhub-homelab), implementation, maintenance and operational work. Project access is required.

Both repositories belong to the same homelab, so one board avoids splitting related work or coordinating two backlogs. Tasks may contain private information, and the planning is only for myself, so publishing it is unnecessary even when the resulting documentation is public. It is a hobby, not a team delivery process; sprints, estimates, deadlines and formal planning are unnecessary. Technical design and safe-change checks still matter.

I used GitHub's existing **Team Planning** template because it already has the main task-tracking aspects I wanted. I do not need to use everything it provides: extra views and sprint/current-iteration features can remain unused, or serve as a loose grouping of current work without committing to a time-based schedule. One status board remains the everyday workflow.

> **Warning: explicitly set the GitHub Project's visibility to Private before adding tasks.** Linking it to a private repository does not make the project private automatically. A private project also does not hide public issues: keep private details in private-repository issues or private project draft items, never in public issues. Follow the [publication boundary](./docs/repository-guidelines.md#publication-boundary); secrets still belong in [protected storage](./docs/repository-guidelines.md#secrets-and-sensitive-artifacts).

1. Start from the **Team Planning** template, keep the GitHub Project private and use one **Board** view for everyday tracking. Link it to the private repository. In the project description, link to [this Task Tracking explanation](https://github.com/CollinPoetoehena/devhub-homelab/blob/main/README.md#task-tracking) so the workflow is easy to find.
2. Use a **Status** field with **TODO**, **In progress** and **Done**. Add an issue or draft item for each useful task, with a short description of what completion means. Public-repository issues may be added when their contents are safe to publish; keep other tasks private.
3. Split larger tasks into small, closely related subtasks with a clear next action and completion check. Keep their supporting context linked to the parent task, so the structure helps focus instead of creating separate competing projects.
4. Pick a TODO when you have time and want to work on the lab, move it to In progress, then close the issue (if applicable) and move the item to Done after checking the result. Keep the active set small. Before stopping, note the last result and where to resume; unfinished work does not need to stay in your head. Keep ideas in TODO, defer or drop them freely, and do not treat a growing backlog as a debt.

Follow the [one-context-at-a-time boundary](./docs/01-foundations/goals-and-scope.md#focus-on-one-context-at-a-time): do not alternate work tasks and personal homelab tasks, or start fixing an alert while shopping or resting. Capture a brief follow-up when appropriate, then leave it for a suitable lab session. Related subtasks within the current larger task can be useful while waiting for an AI result or a build, but neither waiting nor a notification requires starting more work. When a lab session ends, note where you stopped and what to do next in the task comments or another convenient note; you do not need to keep working until a task is done. See [write it down and return when ready](./docs/01-foundations/goals-and-scope.md#write-it-down-and-return-when-ready) and [timebox without turning it into a deadline](./docs/01-foundations/goals-and-scope.md#timebox-without-turning-it-into-a-deadline) for how tracking and optional timeboxes support that boundary.

> **Central reference:** The [private homelab README](https://github.com/CollinPoetoehena/devhub-homelab-private/blob/main/README.md#task-tracking) references this section instead of repeating the task-tracking explanation. Both repositories use the same private project; there is no separate public planning board. Only the workflow explanation and safe documentation are published.

---

## Documentation

- [The Homelab Book](./docs/README.md): the entry point, layers, storyline, reading order and full table of contents.
- [Part 1 · Foundations](./docs/01-foundations/README.md): goals and scope, requirements, design principles and the central [cost and scope guide](./docs/01-foundations/cost.md).
- [Part 2 · Architecture](./docs/02-architecture/README.md): physical placement and power, network and Wi-Fi, compute, storage and the workload platform.
- [Part 3 · Platform Services](./docs/03-platform-services/README.md): DNS, certificates, identity, secrets and scheduling.
- [Part 4 · Automation](./docs/04-automation/README.md): infrastructure as code, configuration management, deployment and the [project structure](./docs/04-automation/project-structure.md).
- [Part 5 · Services](./docs/05-services/README.md): workload design and self-hosted service options.
- [Part 6 · Operations](./docs/06-operations/README.md): observability, backups, disaster recovery and maintenance.
- [Repository guidelines](./docs/repository-guidelines.md): documentation structure, public/private boundary, writing rules, sanitization and security.
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

A project-context skill is unnecessary: DevHub skills automatically discover instructions, documentation structure, guidelines and relevant validation commands. Keep any additional persistent homelab context in [copilot-instructions.md](.github/copilot-instructions.md). The only [local skill](.github/skills/README.md) is [homelab-sanitize](.github/skills/homelab-sanitize/SKILL.md), for explicitly authorized publication preparation.

**Do not chain skills.** Each skill must be self-contained and usable without another skill being invoked first or afterward. Background and project-policy links are fine; required cross-skill invocation is not. See [DevHub's design principles](https://github.com/CollinPoetoehena/DevHub/blob/main/.github/AI.md#general-design-principles) and its [skill design explanation](https://github.com/CollinPoetoehena/DevHub/blob/main/.github/skills/README.md#create-a-skill) for why dependency chains are unreliable and how to keep skills self-contained.

**Documentation owns project policy.** The [repository guidelines](./docs/repository-guidelines.md), including the [Documentation Structure & Guideline](./docs/repository-guidelines.md#documentation-structure--guideline), remain authoritative for both homelab repositories. Reference them rather than duplicating their rules in skills. AI must read applicable guidance, preserve the publication boundary, distinguish planned work from verified results and validate changes. Publication and live infrastructure operations still require explicit authorization.

### AI usage

Install the shared DevHub skills using [Use external or centralized customizations across repositories](https://github.com/CollinPoetoehena/DevHub/blob/main/.github/AI.md#use-external-or-centralized-customizations-across-repositories), which also links the DevHub installer and explains manual copies and symbolic links. A GitHub link is a reference, not an installation. Verify discovery in the selected host, on the machine where the agent runs.

For a shared parent workspace as described in [Working Across DevHub and the Homelab Repositories](#working-across-devhub-and-the-homelab-repositories), you can add a workspace-local `.github/skills/` directory and symbolic links to the **additional homelab-specific** skill directories. For example, from that parent folder on a host supporting symlink discovery:

```sh
mkdir -p .github/skills
ln -s /absolute/path/to/homelab/devhub-homelab/.github/skills/homelab-sanitize .github/skills/homelab-sanitize
```

Replace the absolute source path with your clone location, ensure the destination does not already exist, and link the complete skill directory so its resources remain available. Keep the clone at that path. Add links for future genuinely homelab-specific skills in the same way; no private-specific skill is currently needed. Do not commit these machine-local workspace links into either repository. If the host does not discover symlinks, use the supported copy/setup options in the DevHub guide instead.

**Centralizing homelab-specific customizations is not necessary.** They are specific to my homelab projects, unlike the generic DevHub customizations reused across unrelated projects. Keep their source in the owning repository; parent-workspace links are optional convenience, not a reason to move them into DevHub or install them globally.

Opening the parent folder does not guarantee discovery of nested repository instructions. Explicitly select the target repository and provide its [public instructions](.github/copilot-instructions.md) or [private instructions](https://github.com/CollinPoetoehena/devhub-homelab-private/blob/main/.github/copilot-instructions.md) as context when necessary. Verify that the intended skills are available. Access to all three repositories never authorizes publishing private content.
