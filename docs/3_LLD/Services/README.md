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

---

## GitOps and Continuous Delivery

GitOps reconciles runtime workloads from declared Git intent. Argo CD offers visual sync/diff workflows; Flux offers a controller/CLI-oriented model. Both require deliberate access, secret delivery and ownership.

CI-push deployment can be simpler but needs deployment credentials and drift handling. Manual Helm is packaging without ongoing reconciliation; it can also be used beneath a GitOps controller. Do not expect workload reconciliation to manage the host OS automatically.

---

## CI and Secret Delivery

CI validates/tests/builds; CD deploys. Hosted GitHub Actions/GitLab CI avoid some infrastructure; Jenkins/Woodpecker or other self-hosted systems add control and maintenance. Choose runner trust, billing, permissions and approval gates deliberately. Reuse [DevHub workflows](https://github.com/CollinPoetoehena/DevHub/tree/main/.github/workflows) for broadly shared practices.

SOPS + age can encrypt Git-native values; external managers such as Vault support other secret lifecycles with availability/recovery responsibilities. External Secrets Operator integrates stores; it is not itself a store. Ansible Vault suits encrypted host automation data. Kubernetes Secret base64 encoding alone is not encryption.

Protect identifying endpoints as well as credentials in private records under the [shared guideline](../../Repository_Guidelines.md#secrets-and-sensitive-artifacts). Choosing an encryption tool does not automatically wire decryption into a delivery controller.

---

## Implementation and Verification

Select delivery/secret ownership first, then define resources, health checks, least-privilege RBAC/network policy and persistent-data recovery. Validate reconciliation/drift, allowed/denied application access, secret delivery without leaked output, and rollback/restore.

Use disposable isolated experiments and publish only generalized lessons. Exact values, installed workloads and operational output remain private; this document is not the owner's deployment plan.
