# Services and Delivery

Choose useful workloads and their dependencies before picking delivery tools. Not every service needs Kubernetes or internet exposure.

---

## Table of Contents

- [Workload Design](#workload-design)
- [GitOps and Continuous Delivery](#gitops-and-continuous-delivery)
- [CI and Secret Delivery](#ci-and-secret-delivery)
- [Implementation and Verification](#implementation-and-verification)

---

## Workload Design

Distinguish disposable experiments from household dependencies. Consider data ownership, capacity, authentication, persistence and recovery. Application projects can have their own repositories/lifecycle; the homelab integrates their artifacts.

Choose single-host services versus [orchestration](../Cluster/README.md), private access/VPN versus deliberate public exposure, and appropriate TLS/access controls. Ingress routing, load-balancer address allocation and authentication solve different problems.

Classify each workload by owner, purpose, dependencies, persistence and acceptable outage. Stateless experiments can often be recreated from images and configuration; stateful services need data lifecycle and migration plans. A household service promoted out of the experiment tier needs patching, access controls, monitoring and tested recovery.

A reverse proxy or ingress can route several applications behind one entry point, but does not automatically authenticate users or make an application safe to expose. Prefer private access when public reachability provides no benefit. Keep application code/build artifacts in the appropriate project repository and let the homelab consume releases, rather than mixing every application into the infrastructure repository.

---

## GitOps and Continuous Delivery

GitOps reconciles runtime workloads from declared Git intent. Argo CD offers visual sync/diff workflows; Flux offers a controller/CLI-oriented model. Both require deliberate access, secret delivery and ownership.

CI-push deployment can be simpler but needs deployment credentials and drift handling. Manual Helm is packaging without ongoing reconciliation; it can also be used beneath a GitOps controller. Do not expect workload reconciliation to manage the host OS automatically.

**Argo CD** centers application sync and desired/actual comparisons, with a UI that can help make reconciliation visible. **Flux** is a controller-based GitOps toolkit with Git/Kubernetes-oriented workflows. Compare repository access, tenancy, promotion, notification and debugging models. Both require explicit configuration and neither inherently supplies secret decryption or safe application rollback.

**CI-driven deployment** can be direct and understandable for a small environment: a trusted job approves and applies changes. It also gives the runner deployment reach and needs deliberate drift detection. **Manual deployment** can fit short-lived experiments, but record the steps and inputs so it is not an undocumented one-off. As scope grows, automate reconciliation or another repeatable delivery path rather than relying on memory.

Helm packages and templates resources; Kustomize overlays compose/modify resources. They can be used by reconciliation controllers or explicit deployment stages. Keep bases reusable and environment values separate, and avoid competing controllers/manual changes to the same object. A Git revert may restore manifests but cannot undo a destructive schema migration or deleted data.

---

## CI and Secret Delivery

CI validates/tests/builds; CD deploys. Hosted GitHub Actions/GitLab CI avoid some infrastructure; Jenkins/Woodpecker or other self-hosted systems add control and maintenance. Choose runner trust, billing, permissions and approval gates deliberately. Reuse [DevHub workflows](https://github.com/CollinPoetoehena/DevHub/tree/main/.github/workflows) for broadly shared practices.

SOPS + age can encrypt Git-native values; external managers such as Vault support other secret lifecycles with availability/recovery responsibilities. External Secrets Operator integrates stores; it is not itself a store. Ansible Vault suits encrypted host automation data. Kubernetes Secret base64 encoding alone is not encryption.

Protect identifying endpoints as well as credentials in private records under the [shared guideline](../../Repository_Guidelines.md#secrets-and-sensitive-artifacts). Choosing an encryption tool does not automatically wire decryption into a delivery controller.

Hosted CI avoids operating a server but has runner, billing and privacy constraints. Self-hosted runners can reach private resources and use specialized hardware, yet expose those resources if untrusted code executes there. Separate validation of untrusted pull requests from credentialed deployment, scope permissions and require approval for live changes.

SOPS encrypts selected values with supported key mechanisms; age is one encryption/key option, not the delivery controller. Ansible Vault encrypts host-automation files/variables. External stores such as Vault add centralized secret lifecycle and potentially dynamic credentials, with their own availability/recovery work. External Secrets Operator fetches from a supported store; its credentials and resulting Kubernetes Secrets still need protection.

Choose where decryption occurs and who can access the resulting plaintext. Protect decryption keys outside Git, constrain controller/runner identity and avoid plaintext in rendered diffs, logs or artifacts. A private repository or a base64-encoded Kubernetes Secret is not a substitute for that design.

---

## Implementation and Verification

Select delivery/secret ownership first, then define resources, health checks, least-privilege RBAC/network policy and persistent-data recovery. Validate reconciliation/drift, allowed/denied application access, secret delivery without leaked output, and rollback/restore.

Use disposable isolated experiments and publish only generalized lessons. Exact values, installed workloads and operational output remain private; this document is not the owner's deployment plan.

Build/test artifacts before deployment, pin compatible releases, then validate health and actual allowed/denied access. Readiness, liveness and startup checks answer different questions; a process running is not necessarily ready to serve. Define resource budgets and disruption behavior according to the service, not generic percentages.

Test a controlled drift event and confirm the chosen delivery workflow restores intent. Exercise secret rotation and data-aware rollback separately. Keep workload integration and planned GitOps ownership visible in the [project structure](../../Project_Structure.md), without publishing manifests or environment values.
