# Goals and Scope

> **Part 1 · Foundations** — [Part overview](./README.md) · Next: [Requirements](./requirements.md)

Choose the purpose before buying equipment or installing tools. This is the first chapter of the book because every later decision—how many hosts, which network boundaries, which services, how much automation, how much recovery effort—only makes sense against a clear answer to *why this homelab exists and what it should and should not become*.

The chapter answers the first question of the storyline: **why am I building this?** It describes the kinds of purpose a homelab can serve, why the hands-on exploration is worth it, and how to keep the scope small enough to remain enjoyable and maintainable. The next chapters turn that purpose into [requirements](./requirements.md), [design principles](./design-principles.md) and a [cost boundary](./cost.md).

---

## Table of Contents

- [Design the Purpose](#design-the-purpose)
  - [Why Build a Homelab?](#why-build-a-homelab)
- [Relaxation and Enjoyment over Productivity and Pressure](#relaxation-and-enjoyment-over-productivity-and-pressure)
  - [A Hobby, Not a Second Job](#a-hobby-not-a-second-job)
  - [Timebox Without Turning It into a Deadline](#timebox-without-turning-it-into-a-deadline)
  - [Focus on One Context at a Time](#focus-on-one-context-at-a-time)
  - [Notifications Are Not Automatic Interruptions](#notifications-are-not-automatic-interruptions)
  - [Keep Household Dependencies out of Experiments](#keep-household-dependencies-out-of-experiments)
  - [Keep the Scope Sustainable](#keep-the-scope-sustainable)
  - [Write It Down and Return When Ready](#write-it-down-and-return-when-ready)
- [Define What Is Out of Scope](#define-what-is-out-of-scope)
- [From Goals to Requirements](#from-goals-to-requirements)

---

## Design the Purpose

A homelab may be a disposable learning environment, a development sandbox, a self-hosted household platform, or a mixture. Learning Kubernetes on demand is different from hosting the only copy of household files. Define what must remain available when experiments fail.

Record learning topics, useful services, data importance, acceptable downtime/data loss, budget, electricity, space/noise, cabling and maintenance time. Decide whether remote access is actually needed and who administers the system.

| Option | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Disposable learning | Short-lived isolated exercises | Fast rebuilds, bounded scope and useful diagnostics | Remove/promote deliberately; not every exercise needs permanent hardware or HA. **Maintenance:** Little permanent upkeep if exercises are removed; refresh rebuild inputs and clean up resources/credentials after each exercise. | Explore tools/protocols without household dependencies |
| Development/testing | Reproducible environments for application integration | Realistic test paths and repeatable development | Test credentials/data must not share production trust; cleanup and capacity still matter. **Maintenance:** Keep test images, dependencies and fixtures current; reset environments and verify that stale credentials or data do not persist. | Build/test applications and infrastructure changes |
| Household services | Self-hosted useful services with ongoing users/data | Practical value and operational learning | Predictable maintenance, backups and usable recovery; household users may not accept lab-style downtime. **Maintenance:** Recurring patching, backup checks and restore exercises are essential; schedule changes around users and retain usable recovery instructions. | You can own the service's reliability/data lifecycle |
| Enterprise-platform practice | Targeted management, upgrade and failure exercises | Experience with specific enterprise concepts | Cost/resources and operational scope; copying enterprise topology is unnecessary for unrelated goals. **Maintenance:** Budget time for supported release paths and management components; retire exercises that no longer serve the learning goal. | Enterprise learning itself is the requirement |

Write requirements as outcomes: "restore important data within the accepted time" or "keep household connectivity working during compute maintenance," not "buy three nodes." Identify must-haves, useful learning goals and deferred ideas. A tool is a means of achieving those outcomes, not the goal unless learning that tool is explicitly the exercise.

### Why Build a Homelab?

A homelab brings many layers of engineering together for hands-on exploration: hardware, networking, storage, software, observability, automation, and much more. An idea does not stay abstract: you can build it, see how its parts depend on one another, find out what happens when something changes, and work through how to make it better. That mix of curiosity, problem-solving, and tangible progress is a large part of the appeal for me. I love engineering, enjoy learning how complex things fit together, and find that time spent experimenting in my lab gives me both a sense of purpose and a way to relax (see also [Relaxation and Enjoyment over Productivity and Pressure](#relaxation-and-enjoyment-over-productivity-and-pressure)).

It is also a personal playground: a place to try unfamiliar ideas, make mistakes safely, recover, and decide what is worth keeping without turning every experiment into a permanent service. A homelab is not only a way to host things; it is a self-directed space to create, learn, and enjoy the process. And there is always more to it than any short description can capture: every experiment can reveal another connection, another question, or a new direction to explore.

---

## Relaxation and Enjoyment over Productivity and Pressure

> **Central reference note:** This section is the shared home for guidance on homelab enjoyment, balance, focus and avoiding unnecessary pressure. Collect related explanations and personal lessons here rather than scattering them across different sections or maintaining duplicate versions. Other chapters and the private repository should link here for these principles, keeping only their topic-specific application and technical details in the sections that own them. For example, [Should You Self-Host It?](../05-services/self-hosted-services.md#should-you-self-host-it) still owns the comparison of self-hosted, hosted and local-first services; [Observability and Monitoring](../06-operations/observability.md#design-signals-and-placement) owns signal selection, alert routing and notification configuration; and optional project task tracking can capture a small next step without deadlines. My shared private backlog workflow is documented in [DevHub Task Tracking](https://github.com/CollinPoetoehena/DevHub/blob/main/docs/how-to/TASK-TRACKING.md); readers do not need that workflow or any tracker to use this guide. Each chapter can link back here for the shared reasoning about enjoyment, attention and pressure, rather than repeating that broader explanation.

The lab should fit around life, not make life fit around the lab. Find a balance that leaves room for work, everyday responsibilities, other interests and rest, while letting you enjoy the lab when you choose to spend time on it. Enjoyment, relaxation and inner peace matter more than a busy backlog or constant availability. Protecting time and attention, limiting operational responsibilities and allowing yourself to stop are parts of the same boundary: this is a hobby to choose, not a burden to carry around all day.

The following subsections explore each part of that balance in more detail: keeping the lab a hobby rather than a second job, timeboxing without deadlines, focusing on one context at a time, handling notifications without constant interruptions, keeping household dependencies out of experiments, limiting scope and using task tracking to return when ready. Together, these boundaries help keep the lab enjoyable without letting it take over the rest of life.

> **Personal note:** Finding this balance was difficult for me, especially when I first started the homelab, precisely because I enjoyed it so much. There was so much to discover, build and understand, and each new idea made me want to explore the next one. I wanted to keep working on it, improve everything and make it perfect. That excitement was part of what made starting so rewarding (see [A Hobby, Not a Second Job](#a-hobby-not-a-second-job)), but it also made it difficult to leave things for another day and accept that the lab did not need to be complete (see [Timebox Without Turning It into a Deadline](#timebox-without-turning-it-into-a-deadline)).
>
> That enthusiasm eventually turned into a lot of unnecessary pressure I was putting on myself. Wanting to learn and experiment started to feel like needing to make progress, resolve every open question and get everything right. None of that pressure was a requirement of the hobby; it came from the expectations I was setting for myself. Enjoying something deeply does not mean I need to spend every available moment on it (see [timeboxing](#timebox-without-turning-it-into-a-deadline)), and having more ideas does not mean I owe myself a finished result for each one.
>
> This is a reminder to myself, not a claim that balance is something I can solve once and never think about again: relax, enjoy experimenting and learning, and let the lab grow at a pace that fits the rest of life. What has helped me is giving lab time a gentle boundary (see [Timebox Without Turning It into a Deadline](#timebox-without-turning-it-into-a-deadline)): choose a possible time to start and stop, then leave the rest for another session. I can have a couple of likely windows in mind, but they remain optional; I can skip them if I do not feel like doing lab work. That is a limit on when I need to think about the lab, not a deadline to accomplish a certain amount. I can care about quality without demanding perfection, leave an experiment unfinished, change direction or take a break without needing to earn it by completing another task first. I do not need to perfect or finish everything for this hobby to be worthwhile. The point is to enjoy the process and find some peace in it, not turn something I love into something I feel I must do.
>
> My recommendation to others, based on my own homelab experience, is not to put that unnecessary pressure on yourselves either. Feeling that you must keep working, finish every idea or make everything perfect can take away the fun that made you want to build a homelab in the first place (see [A Hobby, Not a Second Job](#a-hobby-not-a-second-job)). Find a balance that works for you, give yourself permission to stop, and enjoy learning and experimenting without making the hobby a burden.

### A Hobby, Not a Second Job

A homelab is a hobby and a place to learn, not another productivity target. Its success is not measured by how quickly something is deployed, how many services stay online, how much of a backlog gets completed, or how closely the setup resembles a production environment. There is no required pace, deadline, or finish line. Taking time to understand an idea, following curiosity, repeating an experiment, or deciding not to keep a project can all be worthwhile outcomes.

For me, working on the lab is relaxing as well as interesting. A concrete, hands-on problem gives me something absorbing to focus on; I can see an idea take shape on real equipment, follow questions that arise naturally, and enjoy the satisfaction of making things work. The focus comes from curiosity, not a deadline, a customer, or pressure to deliver quickly. I can choose a small task, explore without knowing exactly where it will lead, and stop or return to it later. Some sessions are challenging or frustrating, of course; the freedom to experiment, learn from a mistake, and take a break is what keeps the overall experience enjoyable rather than an obligation.

Make room for the parts of the process that make the hobby rewarding: exploring, building, observing, troubleshooting, and learning from mistakes. It is fine to leave an experiment unfinished, simplify a design, turn the lab off, or take a break and return when it feels interesting again. Do not add automation, complexity, or extra work just to appear efficient or to match someone else's setup; choose the amount of structure that helps you learn and enjoy the work.

Relaxed does not mean careless. Keep experiments reversible and within the [scope you can sustain](#keep-the-scope-sustainable), and account for the responsibilities of any service that holds important data or affects other people. A deliberate boundary protects both the household and the enjoyment of experimenting; it is not a reason to turn the hobby into an uptime contest.

### Timebox Without Turning It into a Deadline

It can help to choose some times when you might work on the homelab, so it does not quietly expand into every spare moment. The schedule is a gentle boundary for your attention, not a promise, quota or deadline. It can be regular or flexible: for example, you might generally leave Friday afternoon and a couple of hours on Saturday morning available for lab work, while keeping the rest of the week optional. Someone else might choose one evening, a short weekend session, or decide week by week. Pick a pattern that fits your life—or do not set a recurring schedule at all, whatever fits your life best.

When a session starts, choose something interesting or a manageable next step. When the time you set aside ends, use that as a natural cue to stop, even if the task is unfinished or you feel there is more you could do. You can also stop earlier, skip the session, or use the time for something else whenever you do not feel like working on the lab. The point is to give yourself permission to say, "I have lab time again later; I do not need to squeeze this into every free moment." If an actual emergency affects essential services or safety, handle it appropriately (see [Notifications Are Not Automatic Interruptions](#notifications-are-not-automatic-interruptions)); ordinary lab work and unfinished experiments can wait.

Timeboxing is **not** a productivity technique for getting more done or finishing work within an estimate. Do not race the clock, measure success by completed tasks, or turn a missed session into debt. Keep the focus on curiosity, careful work and enjoyment described above. A short, satisfying experiment—or deciding to stop and rest—can be a good use of a session even when nothing is "finished."

Before stopping, write down where you got to, what you observed and the next useful step. Use whatever low-friction place you will find again: a Word document, a note in your own files, or a comment on the related task. See [Task Tracking](../README.md#task-tracking) for keeping follow-ups and context together, and [Recording Experiment and Workload Outcomes](../05-services/workloads.md#recording-experiment-and-workload-outcomes) for preserving experiment results. Then close the work and return to the rest of your day; you can continue in a later session if and when you want.

### Focus on One Context at a Time

**Do not work on the homelab during work.** When working, focus on work; when choosing to spend time on the homelab, focus on the homelab; when buying groceries, spending time with other people or resting, be present for that activity. A lab notification, an interesting idea or an AI response becoming available is not permission to interrupt the current activity. Learning something at home that later helps at work is valuable, but it does not make personal lab maintenance a work task.

Attention and working memory are limited. Two demanding activities usually mean switching between them rather than giving both full attention. Each switch requires reconstructing context: what was the goal, what changed, which assumptions apply, and what still needs checking? Unfinished thoughts can linger after switching, making it harder to settle into the next activity. Work and homelab tasks may use similar tools, but they still have different environments, responsibilities and goals; mixing them can also make it easier to run a command in the wrong place.

Repeated switching can make both activities take longer and reduce their quality. It encourages rushing to get back to the other task, choosing speed over understanding, skipping verification and feeling behind in both places. The point of focusing is not to squeeze more productivity out of every minute; it is to give the chosen activity enough attention to do it carefully and enjoy it without another task pulling at you.

This does not require rigidly doing one tiny step at a time. Within one larger task, related subtasks can share useful context. While an AI prepares a change, a build runs or a test completes, reviewing that task's documentation, preparing its test cases or checking a closely related requirement can be sensible. Keep those activities non-conflicting and review each result before relying on it. Waiting is also a valid choice; an idle minute does not need filling.

Sometimes another work task must be handled while waiting or because priorities change. Make that a deliberate switch, leave a brief resumption note, and return at a clear stopping point where possible. Staying within the work domain helps preserve the work/personal boundary, but unrelated work tasks still have switching costs. It is not a reason to open a personal homelab task alongside work.

### Notifications Are Not Automatic Interruptions

**Routine lab alerts do not need to be assessed when they arrive.** A lab component being down is normally an acceptable state: experiments can remain unavailable until a lab session you choose. Do not check, triage or troubleshoot ordinary lab alerts during work, errands, rest or other activities; in general, let routine alerts wait until you have chosen to work on the lab. Better still, configure notifications so ordinary experiment failures do not interrupt you at all. The lab should be designed so that losing experimental compute does not take household internet, Wi-Fi, essential name resolution or other important services with it. See [Keep Household Dependencies out of Experiments](#keep-household-dependencies-out-of-experiments) and the [household-independence design principle](./design-principles.md#keep-the-household-foundation-independent-of-the-lab).

Prompt attention is for a **credible emergency only**—for example, verified signs that an account or system is compromised, an immediate physical safety risk, imminent loss of important data, or an essential household service being affected. A generic warning or an experiment going offline is not by itself an emergency, and there is no need to investigate routine alerts just in case. Reduce the chance of emergencies with sensible security and recovery practices: keep systems patched, restrict access, separate experimental workloads from household services, and maintain recoverable backups. These controls lower risk but cannot guarantee incidents will never happen. Decide in advance which specific, trustworthy signals warrant an interruption and what safe first response is appropriate. The [observability chapter](../06-operations/observability.md) owns alert design and verification.

Avoid keeping dashboards, chat notifications or AI sessions open in the background of work, errands or rest just to watch for the next thing to do. Suppress or defer routine notifications outside chosen lab time, group related alerts, and use planned maintenance silences for expected shutdowns. This keeps ordinary lab downtime from entering your attention at all while leaving a deliberate notification path for the rare, credible emergency that truly needs a timely response.

The aim is not a hobby that feels compulsive or an expectation of personal 24/7 on-call coverage. If you repeatedly feel you must check, fix or continue, step back: reduce notifications, pause or retire the experiment, simplify the setup, or move an important responsibility to a service you do not have to operate yourself. Taking a break is a valid outcome, not a failure to keep up.

### Keep Household Dependencies out of Experiments

Technical independence supports personal freedom: it is much easier to leave the lab alone when nobody needs it working to get on with daily life. Household internet, Wi-Fi and essential name resolution should remain available when experimental compute is off, broken or being rebuilt. A stable shared network foundation can still need maintenance; independence means it does not depend on lab-hosted DNS, cluster workloads, storage or management VMs. See the [household-independence design principle](./design-principles.md#keep-the-household-foundation-independent-of-the-lab) and [network options](../02-architecture/network.md#design-options) for the technical approaches.

You do not need to self-host every useful service or move important household data into the lab. Hosted or local-first alternatives can leave more time and attention for the experiments you actually enjoy. If other people or important data do depend on a service, acknowledge its patching, backup and recovery obligations explicitly; calling it a hobby does not remove them. Choose those responsibilities consciously using the [self-hosting trade-offs](../05-services/self-hosted-services.md#should-you-self-host-it), rather than letting an experiment quietly become essential.

### Keep the Scope Sustainable

Prefer a small stable foundation and disposable experiments. One host can teach many concepts; extra machines are justified by capacity, physical-failure practice or actual availability requirements, not an enterprise-rack aesthetic.

Automation is useful when it saves work or supports learning, not when it makes every simple task a development project. Limit ongoing services to what you are willing to maintain without crowding out work, other interests or rest.

It is easy to overspend on equipment that looks impressive but is unnecessary for your goals; use the central [cost and scope guide](./cost.md) before expanding.

Include time and operational cost as well as purchase cost. Software that is free to download still needs updates and recovery work; repurposed hardware still consumes electricity. Prefer one well-understood service with tested recovery over a long list that nobody can operate after an outage.

Define a stable foundation and an experiment lifecycle: create, observe, learn, then remove or promote deliberately. Automate repeatable rebuilds where supported, and decide what evidence an experiment should produce. Promotion to a household dependency should add ownership, monitoring, access controls and data recovery, not merely a permanent restart policy.

Keep the learning even when the infrastructure goes away: [record experiment and workload outcomes](../05-services/workloads.md#recording-experiment-and-workload-outcomes) in short Markdown notes so you can revisit the evidence, improve your understanding and choose a useful next experiment.

### Write It Down and Return When Ready

Use [Task Tracking](../README.md#task-tracking) as a place to put ideas down, not as a demand to act on them now. A short note with the next action, relevant context and a link is enough to make returning easier. If an idea occurs outside lab time, capture it briefly when appropriate and safe, then return to the current activity instead of opening an investigation. Tasks can wait until you have time and feel interested; they can also be deferred or dropped.

Split a larger task into small, closely related subtasks with a clear outcome. Choose one manageable next step, keep the active set small, and record what was checked and where to resume when stopping. This supports focus, structure and organization without forcing you to remember unfinished details or finish everything in one sitting. The board helps protect the [one-context-at-a-time boundary](#focus-on-one-context-at-a-time); it is not a scoreboard or an invitation to start several unrelated tasks.

Find a balance that leaves you relaxed and able to leave the lab alone. There is no requirement to do something with it every day, clear every TODO, respond to every notification or maintain experiments around the clock. If the structure itself starts creating pressure, make it lighter. The purpose remains enjoyment, curiosity and peace of mind.

---

## Define What Is Out of Scope

A goal statement is incomplete until it also says what the homelab is **not**. Writing that down protects the budget, the household and spare time just as much as the goals themselves. Typical exclusions are:

- **Not a production data center.** Enterprise availability, redundant everything and vendor support contracts are not targets unless a specific requirement demands them; see [Learn Enterprise Patterns Without Enterprise Availability](./design-principles.md#learn-enterprise-patterns-without-enterprise-availability).
- **Not the only copy of important data.** If the lab stores something valuable, it also owns a [backup and restore process](../06-operations/backups.md); otherwise, keep that data elsewhere.
- **Not an accidental household obligation.** Apply the [household-dependency boundary](#keep-household-dependencies-out-of-experiments) so experiments can stop without disrupting daily life.
- **Not a collection of every interesting tool.** A tool earns its place by serving a goal; evaluating it briefly and removing it again is a successful experiment, not a failure.
- **Not a reason to buy equipment first.** Purchases follow measured needs and the [cost and scope test](./cost.md#the-most-important-constraint-buy-for-a-real-need).

Exclusions can change later—for example, a service may deliberately be promoted to a household dependency—but the change should be a conscious decision with its added obligations, not something that happens because an experiment kept running.

---

## From Goals to Requirements

Goals are intentionally broad. The [requirements chapter](./requirements.md) turns them into measurable outcomes, and the [design principles](./design-principles.md) describe how to choose between options that meet them. Together with the [cost boundary](./cost.md), these foundations are what the [architecture](../02-architecture/README.md) is designed against. Revisit the goals when a choice adds cost or operational work without supporting them.

Personal priorities and the actual decision record remain private under the [repository guidelines](../GUIDELINES.md#content-ownership).

For each major choice, record the requirement it satisfies and the cost/risk accepted. Review goals after changes in workload, available hardware or household reliance. This prevents expanding a cluster simply because a feature exists and helps explain the reasoning in a portfolio without publishing actual network inputs.

This book is not intended to reproduce general networking, security, systems or product theory, or to become an exhaustive knowledge base. It gives enough context to explain the homelab-specific choice and links to authoritative standards and official product documentation for deeper or release-specific detail. For a formal systems-engineering approach to translating goals into requirements, see NASA's [Systems Engineering Handbook](https://www.nasa.gov/reference/systems-engineering-handbook/).
