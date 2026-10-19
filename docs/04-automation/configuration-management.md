# Configuration Management

> **Part 4 · Automation** — [Part overview](./README.md) · Previous: [Infrastructure as Code](./infrastructure-as-code.md) · Next: [Deployment](./deployment.md)

Configuration management brings existing machines to a desired state—users, SSH settings, packages, services, firewall rules, platform settings—and can bring them back when they drift. Images and first-boot tools prepare the starting point so that configuration management has less to do. Together they answer the question every rebuilt host eventually raises: *how do I get this machine back to exactly how it was?*

This chapter compares configuration tools and image/bootstrap approaches. Creating the machines in the first place is [Infrastructure as Code](./infrastructure-as-code.md); delivering applications to the platform is [Deployment](./deployment.md).

---

## Table of Contents

- [What Configuration Management Owns](#what-configuration-management-owns)
- [Tool Options](#tool-options)
- [Images and Bootstrap](#images-and-bootstrap)
- [Scripts and Glue](#scripts-and-glue)
- [Using Configuration Tools Well](#using-configuration-tools-well)
- [Implementation and Verification](#implementation-and-verification)

---

## What Configuration Management Owns

Configuration management owns the **inside of conventional hosts and guests**: accounts and SSH access (see [Identity and Access](../03-platform-services/identity.md)), packages, service configuration, host firewalls, logging and supported platform settings such as hypervisor storage or API tokens. It consumes resources created by IaC and hands workloads over to the deployment workflow.

Two exceptions are common in a homelab. API-managed node operating systems, such as Talos for Kubernetes nodes, are configured through their own supported API rather than SSH roles. Network appliances managed through a controller or GUI may stay outside configuration management when no supported automation interface exists; document those settings and back them up instead.

---

## Tool Options

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| [Ansible](https://docs.ansible.com/ansible/latest/) | Usually agentless, explicit configuration runs on supported hosts | Roles/modules, handlers and repeatable convergence without a resident agent | Idempotency/check/diff depend on task quality; check mode is not a resource-state plan; API-managed OSes may not support SSH roles. **Maintenance:** keep collections/roles compatible with host releases, test first convergence and reruns, and review drift/credential access | Small fleets and explicit or scheduled reviewed host configuration |
| [Puppet](https://www.puppet.com/docs/) | Desired-state configuration, commonly with agent-based enforcement | Continuous convergence and drift reduction | Agent/controller trust, lifecycle and learning overhead. **Maintenance:** Update agents, server and modules together; monitor failed convergence, certificate expiry and policy regressions. | Fleets needing ongoing enforcement |
| [Chef](https://docs.chef.io/) | Code-driven host configuration and convergence workflows | Reusable policy and continuous configuration management | Client/server or chosen execution-model lifecycle and testing effort. **Maintenance:** Maintain clients, cookbooks and the chosen server/execution model; test convergence and credential renewal before rollout. | Teams already using its policy/tool ecosystem |
| [Salt](https://docs.saltproject.io/) | Configuration and remote-execution platform with several execution models | Flexible orchestration/enforcement across hosts | Choose and secure the controller/agent or agentless model; another operational lifecycle. **Maintenance:** Keep execution components and state definitions compatible; secure keys and monitor failed jobs or disconnected minions. | Fleet execution and convergence needs justify its scope |

Ansible may provision too, but its check mode is not equivalent to a resource-state plan. An immutable/API-managed node OS does not automatically accept conventional SSH/package-management roles.

---

## Images and Bootstrap

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| [Packer](https://developer.hashicorp.com/packer/docs) | Automated machine-image build workflows | Consistent, reproducible baselines reduce repeated installation | Images need refresh, testing and compatible builders; not ongoing configuration management. **Maintenance:** rebuild after base-image/security changes and verify boot/integration before promoting templates | Repeated guest/template creation |
| [cloud-init](https://cloudinit.readthedocs.io/en/latest/) | Supported first-boot guest initialization | Establishes identity, networking and bootstrap configuration | Image/platform support varies; not a replacement for all ongoing maintenance. **Maintenance:** Refresh base images and bootstrap data; test a clean first boot after image/platform changes rather than assuming reruns repair existing guests. | Bootstrapping supported cloud/VM images |
| Supported node-OS API | The purpose-built OS's configuration/bootstrap/upgrade interface | Matches immutable/API-managed lifecycle rather than mutable-host assumptions | Explicit credential, compatibility and apply ownership; use the platform's supported tooling. **Maintenance:** Track supported image/API/provider versions, rotate access credentials and test configuration apply, upgrade and rebuild paths. | Dedicated node systems such as [Talos](../02-architecture/platform.md#kubernetes-node-lifecycle) |

Images reduce repeated installation work and make clones consistent, but they age: an image built months ago reproduces months-old packages and vulnerabilities. Rebuild and retest images on a schedule and after security updates, and let configuration management handle the ongoing state.

---

## Scripts and Glue

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Bounded scripts | Explicit glue or diagnostic programs | Simple for small tasks with clear inputs/outputs | Error handling, exit statuses, safe reruns and idempotency are yours. **Maintenance:** Maintain dependencies, error handling and tests; verify safe reruns when platform commands or output formats change. | Tasks not better served by a supported module/provider |

Scripts are appropriate for bounded tasks with clear inputs and outputs—preparing a local workstation, collecting diagnostics, or calling an API that has no module. They become a liability when they quietly replace a supported module or provider, because error handling, idempotency and safe reruns then become the script author's responsibility.

---

## Using Configuration Tools Well

For configuration tools, prefer reusable roles and supported modules over shell commands. A successful check run is not proof every change was simulated. Choose explicit/scheduled convergence versus continuous enforcement deliberately, and keep images refreshed so cloning does not reproduce an obsolete insecure baseline. Reusable roles/modules should follow [DevHub package conventions](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/README.md); site-specific roots consume them, rather than embedding another divergent copy.

---

## Implementation and Verification

Pin compatible versions of the tool, collections, roles and modules. Test locally with linting and syntax checks, then against an isolated or disposable target. For configuration, test both the **first convergence** on a fresh machine and a **rerun** on a configured one: the second run should report no changes, which is the practical test of idempotency. A successful check-mode run is useful but not proof that every change was simulated correctly.

Protect the credentials that configuration runs use—SSH keys, vault passwords, API tokens—and keep decrypted values out of task output and logs, as described in [Secrets Management](../03-platform-services/secrets.md#where-decryption-happens). Keep a documented bootstrap path: the machine that runs configuration management must itself be rebuildable without the lab. Actual inventories, variables and run results belong in the private repository.
