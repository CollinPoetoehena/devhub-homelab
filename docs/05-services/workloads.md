# Workloads: Designing and Running Services

> **Part 5 · Services** — [Part overview](./README.md) · Next: [Self-Hosted Services](./self-hosted-services.md)

Choose useful workloads and their dependencies before picking delivery tools. Not every service needs Kubernetes or internet exposure.

This chapter is about *how to decide* what runs on the homelab and what each workload needs before it is deployed: its tier, data, exposure, dependencies and lifecycle. The [next chapter](./self-hosted-services.md) surveys common kinds of self-hosted services; the [deployment chapter](../04-automation/deployment.md) explains how workloads are delivered, and [scheduling](../03-platform-services/scheduling.md) decides when they run.

---

## Table of Contents

- [Workload Design](#workload-design)
- [Service Tiers Applied to Workloads](#service-tiers-applied-to-workloads)
- [The Service Record](#the-service-record)
- [Recording Experiment and Workload Outcomes](#recording-experiment-and-workload-outcomes)
  - [Why Keep the Results?](#why-keep-the-results)
  - [What to Record](#what-to-record)
  - [Where to Keep It and When to Update It](#where-to-keep-it-and-when-to-update-it)
- [Promoting an Experiment](#promoting-an-experiment)
- [Retiring a Service](#retiring-a-service)
- [Implementation and Verification](#implementation-and-verification)

---

## Workload Design

Distinguish disposable experiments from household dependencies. Consider data ownership, capacity, authentication, persistence and recovery. Application projects can have their own repositories/lifecycle; the homelab integrates their artifacts.

Choose single-host services versus [orchestration](../02-architecture/platform.md), private access/VPN versus deliberate public exposure, and appropriate TLS/access controls. Ingress routing, load-balancer address allocation and authentication solve different problems.

| Option | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Disposable experiment | Short-lived workload with a create/observe/remove lifecycle | Safe learning and little permanent scope | Isolate trust/data; do not quietly promote it to an essential dependency. **Maintenance:** Remove resources and credentials after learning; refresh rebuild inputs rather than indefinitely patching an unused environment. | Tool evaluation and failure exercises |
| Long-lived stateless service | Useful workload rebuilt from artifacts/configuration | Repeatable replacement without application-data restore | Still needs patching, authentication, monitoring and dependency recovery. **Maintenance:** Patch images/dependencies and test redeployment; maintain authentication and recover external state/configuration dependencies. | Services with external or no persistent state |
| Long-lived stateful service | Workload owning persistent files/database state | Useful self-hosted capabilities | Data placement, migrations, backups and tested recovery; restart policy alone is insufficient. **Maintenance:** Coordinate data-aware upgrades/migrations, backup checks and restore tests; allocate time for storage growth and incident recovery. | Data importance and operational time justify ownership |
| Private / VPN access | Service available only through approved private paths | Less public attack exposure | Identity, VPN/routing and access policy still need maintenance. **Maintenance:** Update the entry point, rotate/revoke client access and verify routing/policy; private reachability does not remove application patching. | Household/admin services without a public audience |
| Public application access | Deliberately internet-reachable workload | External audience can use it | TLS, updates, authentication as needed, scoped exposure and recovery; ingress is not authentication. **Maintenance:** Maintain TLS renewal, security patches, authentication and exposure monitoring; review firewall scope and incident/recovery procedures continuously. | Public reachability supports the service's purpose |

Classify each workload by owner, purpose, dependencies, persistence and acceptable outage. Stateless experiments can often be recreated from images and configuration; stateful services need data lifecycle and migration plans. A household service promoted out of the experiment tier needs patching, access controls, monitoring and tested recovery.

A reverse proxy or ingress can route several applications behind one entry point, but does not automatically authenticate users or make an application safe to expose. Prefer private access when public reachability provides no benefit. Keep application code/build artifacts in the appropriate project repository and let the homelab consume releases, rather than mixing every application into the infrastructure repository.

Use [Service Scheduling](../03-platform-services/scheduling.md) to decide whether each component should run 24/7, on a defined schedule or only on demand. Plan automatic graceful lifecycle management before buying hardware that must remain on.

---

## Service Tiers Applied to Workloads

The [service tiers](../01-foundations/requirements.md#service-tiers-how-much-does-it-matter) from the foundations decide how much care each workload needs. Applied to services, they translate into concrete obligations:

| Tier | Placement | Access | Data | Observability | Updates |
|---|---|---|---|---|---|
| Foundation | Outside lab compute; never inside the workload cluster | Household clients; administration restricted | Configuration exports protected and offline | Independent checks where practical | Staged and scheduled, with rollback |
| Household service | Lab platform, with known failure domains and accepted downtime | Private/VPN by default; public only by explicit decision | Independent, tested [backups](../06-operations/backups.md) with an agreed RPO | Health and backup alerts that someone acts on | Regular patching window; data-aware upgrades |
| Lab platform | Lab platform or management VMs | Administrators and automation only | Reproducible from code; important state backed up | Platform health and capacity | Supported upgrade paths, rehearsed rebuilds |
| Experiment | Lab segment, disposable resources | Owner only | None that matters | Optional, ad hoc | Not patched long-term; removed instead |

The most common mistake is a workload that behaves like a household service—people rely on it, it holds unique data—while still being operated like an experiment. Classify each workload explicitly and review the classification when its use changes.

---

## The Service Record

Before deploying a long-lived service, write a short record of it. This takes a few minutes and answers most questions that would otherwise come up during an outage, an upgrade or a removal. A generic template:

| Field | Question |
|---|---|
| Purpose and requirement | Which [requirement](../01-foundations/requirements.md) or learning goal does it serve? |
| Owner and users | Who maintains it, and who depends on it? |
| Tier | Experiment, lab platform or household service? |
| Dependencies | Which platform services—[DNS](../03-platform-services/dns.md), [certificates](../03-platform-services/certificates.md), [identity](../03-platform-services/identity.md), [secrets](../03-platform-services/secrets.md), storage—does it need? |
| Data | What persistent data does it own, where does it live, and what are its RPO and RTO? |
| Exposure | Home-only, private/VPN or deliberately public? Which [network path](../02-architecture/network.md#reaching-applications-from-home) reaches it? |
| Authentication | How do users and administrators sign in? |
| Delivery and updates | How is it deployed and how are new versions adopted? |
| Observability | Which health checks and alerts show that it works? |
| Backup and recovery | How is its data backed up, and when was a restore last tested? |
| Schedule | Always on, scheduled or on demand? See [scheduling](../03-platform-services/scheduling.md) |
| Removal | What must be deleted (data, credentials, DNS records, firewall exceptions) when it is retired? |

The public book provides the template; the actual records for installed services belong in the private documentation.

---

## Recording Experiment and Workload Outcomes

**Document what you tried, what happened and what you learned**, not only how to deploy the workload. This applies to disposable experiments as well as tests of long-lived services: a failed deployment, access-policy check, restore exercise or upgrade can all produce useful learning. Keep a short Markdown (`.md`) record of meaningful tests and outcomes in the owning repository, even if the experiment will be removed. A running workload, a configuration file or a completed task does not by itself explain what was actually verified.

### Why Keep the Results?

Notes let you return to an experiment after its details have faded, without reconstructing the whole setup or repeating the same investigation. Explaining the result in your own words helps distinguish what you understand from what merely appeared to work: why did the system behave that way, which dependency mattered, and what evidence supports the explanation? On a later read, you can revisit assumptions, connect the result to another concept and improve your understanding rather than relying on a vague memory of success.

Recording failures and unexpected outcomes is just as useful. It preserves troubleshooting paths, prevents repeatedly trying the same unsuccessful fix and makes unanswered questions visible. A dated result with relevant versions and conditions also gives a comparison point for a later change: if behavior differs after an upgrade or configuration change, you have a basis for investigating why. It is evidence for those recorded conditions, not a guarantee that another version or environment behaves identically.

The notes help make later decisions deliberate: whether a tool serves the goal, whether a workload is ready for promotion, which recovery step needs improving and what to test next. They also let you delete disposable infrastructure without losing the value of the exercise. Retaining the learning is more useful than keeping an unused service online just to remember how it worked.

### What to Record

Use the following prompts as a lightweight structure, not a requirement for a lengthy report after every command:

| Item | What to capture |
|---|---|
| Question and expected outcome | What were you trying to learn or verify? What result would support or contradict the expectation? |
| Context and starting state | Date, relevant versions, prerequisites, test data and dependencies needed to interpret or repeat the exercise. Keep actual environment details private. |
| Method | The meaningful steps, safe change or fault introduced, and links to the versioned configuration, test code or runbook used. Avoid copying a second configuration into prose. |
| Observed result and evidence | What actually happened, including failures and unexpected behavior; relevant measurements, assertions or protected evidence references. Distinguish passed, failed, inconclusive and not tested. |
| Explanation and limits | Why you think it happened, what the result demonstrates and what it does not. Separate observations from hypotheses; for example, guest failover does not prove physical-host resilience. |
| Outcome and next step | What you learned, what remains unclear, whether to repeat, improve, remove or promote the workload, and any linked follow-up task. |
| Recovery and cleanup | Whether the starting state was restored, temporary resources and credentials removed, or anything remains to be cleaned up. |

For example, a synthetic restore-test note could say that a disposable dataset was backed up, deleted and restored, that its contents were compared successfully, and what recovery time was measured. It should also say whether off-site recovery or loss of the backup host was **not** exercised. That is more informative than simply writing "backups work." This is an illustration of a record, not a claim that a test has been performed.

### Where to Keep It and When to Update It

Keep a small exercise as a subsection of the owning workload or subject chapter. If a set of tests needs its own Markdown record, keep it alongside the owning project documentation and link it from the subject chapter or service record; do not create a parallel design/setup documentation tree. Application-specific tests belong with the application, while the homelab record links them and captures integration outcomes. Add a short result after a meaningful test or before cleanup, and leave a resumption note if the investigation is unfinished.

Actual installation details and operational results belong in the private record; sensitive logs, screenshots, exports and secrets belong in protected storage, not pasted into Markdown or committed merely because a repository is private. Link to an access-controlled evidence location when needed. The public book owns this method and independently generalized lessons, never raw private test records; follow the [publication boundary](../repository-guidelines.md#publication-boundary).

Keep the effort proportionate to the learning. A few clear sentences may be enough for a small experiment; a recovery exercise may need more evidence and context. The goal is to make returning and understanding easier, not to produce a perfect report or create another obligation. Use [task tracking](../../README.md#task-tracking) for follow-ups and preserve the [enjoyment-over-pressure boundary](../01-foundations/goals-and-scope.md#relaxation-and-enjoyment-over-productivity-and-pressure).

---

## Promoting an Experiment

Experiments are meant to be created, observed and removed. Sometimes one proves useful enough to keep. Promotion is a deliberate step, not a side effect of an experiment that kept running:

1. Confirm that a requirement justifies keeping it and that the [cost](../01-foundations/cost.md)—including maintenance time—is acceptable.
2. Rebuild it from versioned configuration instead of keeping the hand-modified experiment; this also tests that it is reproducible.
3. Replace experiment credentials with properly scoped, stored and rotatable [secrets](../03-platform-services/secrets.md).
4. Add [backups](../06-operations/backups.md) for its data and test a restore.
5. Add [observability](../06-operations/observability.md): at least a health check and alerts for its important failure modes.
6. Review exposure and [authentication](../03-platform-services/identity.md); prefer private access.
7. Write the [service record](#the-service-record) and inform the people who will rely on it about its expected availability.

---

## Retiring a Service

Services that no longer serve a requirement should be removed, not left running "just in case." Retirement frees capacity and electricity and reduces patching and exposure. Before removing a service, export or migrate any data that must be kept, inform its users, then remove the workload, its persistent volumes, credentials and tokens, DNS records, certificates, firewall exceptions and monitoring targets. Update the service record and the documentation that referred to it. A forgotten DNS record or firewall exception for a removed service is a small but avoidable risk.

---

## Implementation and Verification

Select delivery/secret ownership first, then define resources, health checks, least-privilege RBAC/network policy and persistent-data recovery. Validate reconciliation/drift, allowed/denied application access, secret delivery without leaked output, and rollback/restore.

Use disposable isolated experiments and publish only generalized lessons. Exact values, installed workloads and operational output remain private; this document is not the owner's deployment plan.

Build/test artifacts before deployment, pin compatible releases, then validate health and actual allowed/denied access. Readiness, liveness and startup checks answer different questions; a process running is not necessarily ready to serve. Define resource budgets and disruption behavior according to the service, not generic percentages.

Record the observed results and any untested cases using [Recording Experiment and Workload Outcomes](#recording-experiment-and-workload-outcomes). Link the evidence from the service record so later changes and recovery exercises can build on it rather than relying on memory.

Delivery-specific checks—controlled drift, data-aware rollback and the end-to-end pipeline—are described in [Deployment](../04-automation/deployment.md#implementation-and-verification); secret rotation in [Secrets Management](../03-platform-services/secrets.md#implementation-and-verification).
