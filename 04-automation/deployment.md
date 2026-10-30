# Deployment: Continuous Integration and Delivery

> **Part 4 · Automation** — [Part overview](./README.md) · Previous: [Configuration Management](./configuration-management.md) · Next: [Project Structure](./project-structure.md)

Infrastructure as code creates the machines and configuration management prepares them; **deployment** is how application code becomes a tested artifact and how approved intent reaches the [platform](../02-architecture/platform.md). It covers continuous integration (CI), which validates, tests and builds; continuous delivery or deployment (CD), which applies approved changes; and GitOps, which continuously reconciles the running platform with declared intent in Git.

This chapter compares those delivery models and tools. Which workloads are worth running at all is the subject of [Part 5 · Services](../05-services/README.md); how deployment credentials are stored and decrypted is part of [Secrets Management](../03-platform-services/secrets.md#where-decryption-happens).

---

## Table of Contents

- [Delivery Responsibilities](#delivery-responsibilities)
- [GitOps and Continuous Delivery](#gitops-and-continuous-delivery)
- [Continuous Integration](#continuous-integration)
- [Implementation and Verification](#implementation-and-verification)
- [Further Reading](#further-reading)

---

## Delivery Responsibilities

Define how code becomes an artifact, how approved intent reaches the platform and how credentials are delivered. Keep application repositories distinct from infrastructure integration where their release cycles differ: the homelab consumes released artifacts rather than containing every application's source. Reconciliation, packaging and infrastructure provisioning may cooperate, but must not fight over the same resource—this is the [one owner per responsibility](../01-foundations/design-principles.md#one-owner-per-responsibility) principle applied to delivery.

```text
Application repository --CI: lint, test, build--> versioned artifact (image, chart)
                                                         |
Homelab integration repository (declared intent) <-------+  reference by version
        |
        +--GitOps controller / approved CD job--> platform (Kubernetes) --> running workloads
```

The diagram is a generic pattern, not a map of an actual repository layout. A small homelab can start with manual or CI-push deployment and adopt GitOps when continuous reconciliation, drift visibility or learning GitOps becomes a goal.

---

## GitOps and Continuous Delivery

GitOps reconciles runtime workloads from declared Git intent. Deployment workflows and resource packaging are different responsibilities.

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| [Argo CD](https://argo-cd.readthedocs.io/en/stable/) | Kubernetes GitOps application reconciliation with sync/diff UI | Desired/actual visibility and application-oriented workflows | Repository/RBAC/tenancy, promotion, notifications and debugging; secret decryption and data-safe rollback are not automatic. **Maintenance:** patch controllers, renew repository credentials and investigate failed/stuck syncs; reconciliation does not review upgrades for you | Visual application reconciliation helps learning/operations |
| [Flux](https://fluxcd.io/flux/) | Controller-based GitOps toolkit | Git/Kubernetes-oriented composition and ongoing reconciliation | Controller/source access, promotion, notifications, secret integration and debugging. **Maintenance:** coordinate controller/API updates and source credentials, monitor failed reconciliation and review automated version proposals | Controller/CLI-oriented GitOps fits the workflow |
| CI-push deployment | Trusted job approves and applies an artifact/configuration | Direct, explicit and potentially simple for small environments | Runner has deployment reach; credentials, approvals and drift detection required. **Maintenance:** Maintain trusted runners, deployment credentials and approval checks; detect drift and test rollback when pipelines or target APIs change. | Bounded delivery without ongoing reconciliation |
| Manual deployment | Operator explicitly applies recorded inputs | Low initial tooling overhead | Drift, repeatability and forgotten steps; no ongoing reconciliation. **Maintenance:** Low controller upkeep but higher repeated operator effort; keep steps/inputs current and check drift after each change. | Short-lived exercises with documented steps |
| [Helm](https://helm.sh/docs/) | Kubernetes resource packaging/templating | Reusable charts and release inputs; works beneath GitOps or explicit deployment | Chart/value complexity and release compatibility; manual Helm is not continuous reconciliation. **Maintenance:** Review chart/dependency/value changes and rendered diffs; validate supported upgrade/rollback behavior, especially stateful resources. | Reusable application packaging |
| [Kustomize](https://kubectl.docs.kubernetes.io/references/kustomize/) | Resource composition and overlays | Reusable bases with environment-specific modifications | Overlay/patch complexity and ownership; does not itself deploy continuously. **Maintenance:** Keep bases and overlays compatible with API/tool changes; validate rendered output and retire stale patches as upstream resources evolve. | Manifest composition without chart templating |

Do not expect workload reconciliation to manage the host OS automatically. Compare repository access, tenancy, promotion, notification and debugging models. Keep bases reusable and environment values separate, avoid competing owners for the same object, and automate repeatable delivery as scope grows. A Git revert cannot undo a destructive schema migration or deleted data.

---

## Continuous Integration

CI validates/tests/builds; CD deploys. Choose runner trust, billing, permissions and approval gates deliberately. Check the [DevHub reusable-workflow guide](https://github.com/CollinPoetoehena/DevHub/blob/main/docs/GITHUB-ACTIONS.md) for current availability before adopting a shared workflow; its documented workflows are not yet callable.

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| [GitHub Actions](https://docs.github.com/en/actions) | Repository-integrated CI workflows with hosted/self-hosted runners | Hosted runners avoid operating a CI server; integrates project checks/builds | Billing, runner trust, permissions and privacy; private deployment reach needs deliberate design. **Maintenance:** Review workflow/action versions, permissions and secrets; monitor runner availability, billing and failed checks after project changes. | GitHub-based validation/build/delivery |
| [GitLab CI/CD](https://docs.gitlab.com/ci/) | GitLab pipeline and runner ecosystem | Integrated project pipelines with supported hosted/self-managed options | Runner, hosting/tier, billing and permission choices. **Maintenance:** Maintain pipeline includes, runner compatibility and credentials; self-managed deployments also need server upgrades and recovery. | GitLab-based project lifecycle |
| [Jenkins](https://www.jenkins.io/doc/) | Extensible automation server with agents/plugins | Control and broad workflow integration | Server/agent/plugin updates, credentials and recovery. **Maintenance:** Higher server/plugin/agent lifecycle work; test plugin upgrades, protect credentials and restore controller configuration/backups. | Self-managed CI and required integrations justify maintenance |
| [Woodpecker](https://woodpecker-ci.org/docs/intro) | Self-hosted pipeline system with agents | Control over pipeline execution and private integration | Server/agent lifecycle, isolation, credentials and recovery. **Maintenance:** Update server/agents and pipeline integrations; monitor execution isolation and retain protected configuration/database recovery. | A supported self-hosted CI model fits the project |
| Hosted runner | Provider-operated job environment | Avoids machine upkeep and separates jobs from household infrastructure | Billing, privacy, platform limits and private-path reachability. **Maintenance:** Provider handles machine upkeep; maintain job dependencies, quotas, permissions and compatibility with changed runner images. | Ordinary validation/build without local hardware requirements |
| Self-hosted runner | Job execution on an owned machine | Private resource reach and specialized hardware | Untrusted code can expose the host/network/secrets; patching and isolation are yours. **Maintenance:** Patch OS/runtime and runner agent, isolate jobs and clean workspaces; monitor resource use and remove stale credentials/artifacts. | Trusted restricted jobs requiring local reach/hardware |

Separate validation of untrusted pull requests from credentialed deployment, scope permissions and require approval for live changes. CI platform and runner placement are separate decisions; a hosted platform can use self-hosted runners.

---

## Implementation and Verification

Select delivery and secret ownership first, then build/test artifacts before deployment, pin compatible releases and promote them deliberately. Keep plan/apply or sync approval separate from untrusted validation, and restrict who and what can change the live platform.

Test a controlled drift event and confirm the chosen delivery workflow restores intent. Exercise secret rotation and data-aware rollback separately. Keep workload integration and planned GitOps ownership visible in the [project structure](./project-structure.md), without publishing manifests or environment values.

Verify the whole path once with a harmless change: a commit triggers validation, an artifact is produced with a traceable version, the change reaches the platform through the chosen workflow, and health checks confirm the result. Then verify the failure paths: a failing test blocks promotion, a manual change in the cluster is detected or reverted, and a rollback restores the previous version without touching persistent data unexpectedly. Exact pipelines, repositories and environment values belong in the private record.

## Further Reading

For tool-specific behavior, see [GitHub Actions](https://docs.github.com/en/actions), [Argo CD](https://argo-cd.readthedocs.io/en/stable/), [Flux](https://fluxcd.io/flux/) and Kubernetes' [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) documentation. These sources define the supported options; this chapter focuses on how their responsibilities fit together in a homelab. See [Goals and Scope](../01-foundations/goals-and-scope.md#define-what-is-out-of-scope).
