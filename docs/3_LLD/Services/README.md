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

| Option | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Disposable experiment | Short-lived workload with a create/observe/remove lifecycle | Safe learning and little permanent scope | Isolate trust/data; do not quietly promote it to an essential dependency. **Maintenance:** Remove resources and credentials after learning; refresh rebuild inputs rather than indefinitely patching an unused environment. | Tool evaluation and failure exercises |
| Long-lived stateless service | Useful workload rebuilt from artifacts/configuration | Repeatable replacement without application-data restore | Still needs patching, authentication, monitoring and dependency recovery. **Maintenance:** Patch images/dependencies and test redeployment; maintain authentication and recover external state/configuration dependencies. | Services with external or no persistent state |
| Long-lived stateful service | Workload owning persistent files/database state | Useful self-hosted capabilities | Data placement, migrations, backups and tested recovery; restart policy alone is insufficient. **Maintenance:** Coordinate data-aware upgrades/migrations, backup checks and restore tests; allocate time for storage growth and incident recovery. | Data importance and operational time justify ownership |
| Private / VPN access | Service available only through approved private paths | Less public attack exposure | Identity, VPN/routing and access policy still need maintenance. **Maintenance:** Update the entry point, rotate/revoke client access and verify routing/policy; private reachability does not remove application patching. | Household/admin services without a public audience |
| Public application access | Deliberately internet-reachable workload | External audience can use it | TLS, updates, authentication as needed, scoped exposure and recovery; ingress is not authentication. **Maintenance:** Maintain TLS renewal, security patches, authentication and exposure monitoring; review firewall scope and incident/recovery procedures continuously. | Public reachability supports the service's purpose |

Classify each workload by owner, purpose, dependencies, persistence and acceptable outage. Stateless experiments can often be recreated from images and configuration; stateful services need data lifecycle and migration plans. A household service promoted out of the experiment tier needs patching, access controls, monitoring and tested recovery.

A reverse proxy or ingress can route several applications behind one entry point, but does not automatically authenticate users or make an application safe to expose. Prefer private access when public reachability provides no benefit. Keep application code/build artifacts in the appropriate project repository and let the homelab consume releases, rather than mixing every application into the infrastructure repository.

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

## CI and Secret Delivery

CI validates/tests/builds; CD deploys. Choose runner trust, billing, permissions and approval gates deliberately. Reuse [DevHub workflows](https://github.com/CollinPoetoehena/DevHub/tree/main/.github/workflows) for broadly shared practices.

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| [GitHub Actions](https://docs.github.com/en/actions) | Repository-integrated CI workflows with hosted/self-hosted runners | Hosted runners avoid operating a CI server; integrates project checks/builds | Billing, runner trust, permissions and privacy; private deployment reach needs deliberate design. **Maintenance:** Review workflow/action versions, permissions and secrets; monitor runner availability, billing and failed checks after project changes. | GitHub-based validation/build/delivery |
| [GitLab CI/CD](https://docs.gitlab.com/ci/) | GitLab pipeline and runner ecosystem | Integrated project pipelines with supported hosted/self-managed options | Runner, hosting/tier, billing and permission choices. **Maintenance:** Maintain pipeline includes, runner compatibility and credentials; self-managed deployments also need server upgrades and recovery. | GitLab-based project lifecycle |
| [Jenkins](https://www.jenkins.io/doc/) | Extensible automation server with agents/plugins | Control and broad workflow integration | Server/agent/plugin updates, credentials and recovery. **Maintenance:** Higher server/plugin/agent lifecycle work; test plugin upgrades, protect credentials and restore controller configuration/backups. | Self-managed CI and required integrations justify maintenance |
| [Woodpecker](https://woodpecker-ci.org/docs/intro) | Self-hosted pipeline system with agents | Control over pipeline execution and private integration | Server/agent lifecycle, isolation, credentials and recovery. **Maintenance:** Update server/agents and pipeline integrations; monitor execution isolation and retain protected configuration/database recovery. | A supported self-hosted CI model fits the project |
| Hosted runner | Provider-operated job environment | Avoids machine upkeep and separates jobs from household infrastructure | Billing, privacy, platform limits and private-path reachability. **Maintenance:** Provider handles machine upkeep; maintain job dependencies, quotas, permissions and compatibility with changed runner images. | Ordinary validation/build without local hardware requirements |
| Self-hosted runner | Job execution on an owned machine | Private resource reach and specialized hardware | Untrusted code can expose the host/network/secrets; patching and isolation are yours. **Maintenance:** Patch OS/runtime and runner agent, isolate jobs and clean workspaces; monitor resource use and remove stale credentials/artifacts. | Trusted restricted jobs requiring local reach/hardware |

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| [SOPS](https://getsops.io/docs/) | Encrypts selected structured-file values using supported key mechanisms | Git-native review/versioning of encrypted configuration | Protect keys outside Git; configure explicit decryption and prevent plaintext diffs/logs/artifacts. **Maintenance:** update recipient/access rules, rotate compromised material and test decryption/recovery separately from controller health | Encrypted configuration in an owned delivery workflow |
| [age](https://github.com/FiloSottile/age#readme) | Encryption tool/key option, including use with SOPS | Simple key-based encryption workflow | Key custody, rotation and recovery; not a delivery controller. **Maintenance:** Protect and rotate keys/recipients when access changes; test recovery with separately stored keys and do not confuse encryption with secret rotation. | A supported encryption mechanism for protected values |
| [Vault](https://developer.hashicorp.com/vault/docs) | External secret-management service | Centralized secret lifecycle and supported dynamic credentials | Store availability, identity, policy, keys and tested recovery. **Maintenance:** Substantial service/identity/policy lifecycle; monitor availability, renew credentials and test storage/unseal/recovery procedures for the chosen mode. | Centralized/runtime secret needs justify another service |
| [External Secrets Operator](https://external-secrets.io/latest/) | Synchronizes supported external stores into Kubernetes | Connects workload secret delivery to a store | Not itself a store; protect store credentials and resulting Secrets. **Maintenance:** Update controllers/store integrations and access identities; monitor failed synchronization and verify rotation reaches consumers safely. | A supported external manager owns secret lifecycle |
| [Ansible Vault](https://docs.ansible.com/ansible/latest/vault_guide/index.html) | Encryption for host-automation files/variables | Protects Ansible inputs in tracked encrypted form | Vault-key protection and plaintext handling during runs. **Maintenance:** Protect vault passwords, review authorized decryptors and rotate exposed secrets; prevent plaintext in run logs or temporary outputs. | Host configuration consumes encrypted variables |
| [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) | Kubernetes objects consumed by workloads | Native application integration and access controls | Base64 is not encryption; configure RBAC, storage encryption and safe delivery. **Maintenance:** Maintain RBAC, supported storage encryption and consumer rotation/reload behavior; review access and avoid plaintext rendered artifacts. | Runtime consumption, not a substitute for protected source/key storage |

Protect identifying endpoints as well as credentials in private records under the [shared guideline](../../Repository_Guidelines.md#secrets-and-sensitive-artifacts). Choosing an encryption tool does not automatically wire decryption into a delivery controller.

Separate validation of untrusted pull requests from credentialed deployment, scope permissions and require approval for live changes. CI platform and runner placement are separate decisions; a hosted platform can use self-hosted runners.

Choose where decryption occurs and who can access the resulting plaintext. Protect decryption keys outside Git, constrain controller/runner identity and avoid plaintext in rendered diffs, logs or artifacts. A private repository or a base64-encoded Kubernetes Secret is not a substitute for that design.

---

## Implementation and Verification

Select delivery/secret ownership first, then define resources, health checks, least-privilege RBAC/network policy and persistent-data recovery. Validate reconciliation/drift, allowed/denied application access, secret delivery without leaked output, and rollback/restore.

Use disposable isolated experiments and publish only generalized lessons. Exact values, installed workloads and operational output remain private; this document is not the owner's deployment plan.

Build/test artifacts before deployment, pin compatible releases, then validate health and actual allowed/denied access. Readiness, liveness and startup checks answer different questions; a process running is not necessarily ready to serve. Define resource budgets and disruption behavior according to the service, not generic percentages.

Test a controlled drift event and confirm the chosen delivery workflow restores intent. Exercise secret rotation and data-aware rollback separately. Keep workload integration and planned GitOps ownership visible in the [project structure](../../Project_Structure.md), without publishing manifests or environment values.
