# Self-Hosted Services

> **Part 5 · Services** — [Part overview](./README.md) · Previous: [Workloads](./workloads.md) · Next: [Part 6 · Operations](../06-operations/README.md)

The architecture, platform services and automation exist so that something useful can run. This chapter surveys the **kinds of services** a homelab commonly runs and what each kind demands from the layers below it. It is a map of options and their consequences, not a list of what is installed in any particular home; the actual service catalog belongs in the private record.

These are examples to help you explore possibilities—not a checklist, target state or requirement to self-host any of them. You can self-host none, a few, or many. A perfectly reasonable mix is to use OneDrive for file storage and Netflix for streaming while running Home Assistant in the homelab because home automation is the part you want to learn, customize and operate. You can also choose hosted alternatives, use local applications without running a server, or decide that a capability is not needed at all. The right mix depends on your interests, budget, privacy needs and the responsibilities you are willing to take on; there is no prize for self-hosting everything.

Self-hosting can be an absorbing learning project, but it also makes you the operator. You take responsibility for deployment, security updates, access control, monitoring, capacity, data protection and restoring the service when something goes wrong. Depending on the service, you may also need to manage certificates, migrations, hardware failures and changes in upstream software. A provider can take on much of the service's infrastructure and routine operation in exchange for a fee, but that does not remove every responsibility: you still need to consider account security, privacy, subscription terms, data export and what happens if the provider or your internet connection is unavailable. For data you care about, understand the provider's recovery options rather than assuming synchronization or hosting is a separate backup.

This is also, first and foremost, a **cost-and-scope choice**, not a self-hosting purity test. Compare recurring subscriptions with the full cost of owning a service: hardware, electricity, replacements, backups and the time needed to maintain and recover it. Neither self-hosting nor paying a provider is automatically cheaper or better value; decide which responsibilities you want to own and which you would rather pay a provider to take on. Use the [cost and scope chapter](../01-foundations/cost.md)—especially [Count Purchase and Operating Costs](../01-foundations/cost.md#count-purchase-and-operating-costs)—before expanding the lab or taking on another service.

Personally, I am comfortable paying for services such as OneDrive or Netflix when that lets me leave their ongoing operation to a provider and focus my time, money and attention on areas I care more about—such as home automation. Someone else may prefer to self-host those services for control, privacy or learning. Both choices are valid. [Should You Self-Host It?](#should-you-self-host-it) compares the trade-offs.

Every service in this chapter has the same hidden cost: once someone relies on it, it moves up the [service tiers](../01-foundations/requirements.md#service-tiers-how-much-does-it-matter) and brings patching, access control, backups and recovery obligations with it. Read the [workload chapter](./workloads.md) first for how to classify, record, promote and retire services; this chapter then helps decide *which* services are worth that effort.

---

## Table of Contents

- [Should You Self-Host It?](#should-you-self-host-it)
- [Learning and Platform-Practice Workloads](#learning-and-platform-practice-workloads)
- [Media and Photos](#media-and-photos)
- [Files, Documents and Synchronization](#files-documents-and-synchronization)
- [Home Automation](#home-automation)
- [Credentials and Password Management](#credentials-and-password-management)
- [Developer Services](#developer-services)
- [Dashboards and Status Pages](#dashboards-and-status-pages)
- [Choosing a First Service](#choosing-a-first-service)
- [Implementation and Verification](#implementation-and-verification)
- [Further Reading](#further-reading)

---

## Should You Self-Host It?

Self-hosting is a means, not a goal. A hosted service is often the cheaper and more reliable choice for something the household depends on; self-hosting is worth it when it teaches something you want to learn, gives control you actually need, or provides a capability that hosted options do not.

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Self-hosted in the homelab | The service runs on owned hardware and platform | Control over data location, configuration and integration; strong learning value across the whole stack | Availability, security, updates, backups and recovery are entirely yours; outages happen when the lab is down. **Maintenance:** Regular patching, backup checks and restore tests; monitor exposure and resource use; budget time for upgrades and data migrations | Learning goals, privacy or integration needs, and services whose outages the users accept |
| Hosted service (SaaS) | A provider operates the service | Provider handles availability, updates and much of security; nothing to run at home | Subscription cost, provider dependence, privacy and export limitations. **Maintenance:** Little technical upkeep; review account security, billing, data exports and provider changes; keep an exit path for important data | Household-critical services where availability matters more than control or learning |
| Hybrid / local-first | Data stays on devices and synchronizes between them or to a backup, with the homelab as an optional participant | Works when the lab is down; the lab adds convenience or an extra copy rather than being a single point of failure | Synchronization conflicts and multiple copies to reason about; not every application supports it. **Maintenance:** Monitor synchronization health and storage on every participant; back up at least one complete copy independently | Files and notes that must remain available on devices even during lab maintenance |

A useful test: *if the homelab were off for a week, who would notice, and would anything be lost?* If the answer is "everyone" or "irreplaceable data," either operate the service at the household-service tier with tested recovery, or choose a hosted or local-first alternative.

This is also a personal boundary, not just an availability decision. Keep other people's dependencies out of experiments, following the [shared personal-project principles](https://github.com/CollinPoetoehena/DevHub/blob/main/docs/GUIDELINES.md#relaxation-and-enjoyment-over-productivity-and-pressure), so you can pause the hobby without feeling obliged to fix it during work or other activities. Taking on another service should preserve enjoyment, not create an accidental on-call role.

---

## Learning and Platform-Practice Workloads

Not every workload needs to be useful to the household. Workloads designed to exercise the platform are the safest way to learn deployment, scaling, networking and observability because they hold no important data and can be destroyed at any time.

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Sample microservice applications, e.g. Google's [Online Boutique](https://github.com/GoogleCloudPlatform/microservices-demo#readme) | Multi-service demo applications built to showcase cloud-native patterns | Realistic service-to-service traffic for practising ingress, NetworkPolicy, tracing, scaling and rollouts without household data | Resource usage can be significant for a small host; demo security and maturity are not production-grade. **Maintenance:** Treat as disposable; redeploy from upstream manifests rather than patching long-term, and remove when the exercise ends | Platform and observability learning in the experiment tier |
| Self-written small applications | Your own simple web service, API or job packaged as a container | Exercises the full path from source to [CI](../04-automation/deployment.md#continuous-integration) to running workload; easy to break deliberately | You own the code, dependencies and image updates. **Maintenance:** Keep base images and dependencies current, or delete the workload when it has served its purpose | Learning CI/CD, packaging, health checks and failure handling end to end |
| Load and chaos tools | Tools that generate traffic or inject failures into test workloads | Make scaling, alerting and recovery behavior observable | Can affect the whole host or network if not bounded. **Maintenance:** Run only against disposable targets in the lab segment, with limits, and remove after the exercise | Testing alerts, disruption budgets and recovery in a controlled way |

---

## Media and Photos

Media and photo services are popular because they are useful every day. Photo libraries in particular often contain irreplaceable data, which makes them household services with strict backup requirements.

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| [Jellyfin](https://jellyfin.org/docs/) | Open-source media server for personal video, music and photo libraries | No license fees or mandatory accounts; runs fully locally | Transcoding can need significant CPU or a supported GPU; client support varies. **Maintenance:** Update server and plugins, monitor storage growth and transcoding load, back up configuration and metadata; media files need their own backup decision | Household media streaming with local control |
| [Plex Media Server](https://support.plex.tv/articles/categories/plex-media-server/) | Media server with a commercial account ecosystem and broad client support | Polished clients on many devices | Account and online-service dependencies, and some features require a paid plan; check current terms. **Maintenance:** Update server, review account security and remote-access settings, back up configuration | Households valuing client polish over full independence |
| [Immich](https://immich.app/docs/overview/introduction) | Self-hosted photo and video library with mobile backup | Phone upload, search and sharing similar to hosted photo services | Several components (database, machine-learning service, storage); the project is evolving quickly—follow its current upgrade and backup guidance. **Maintenance:** Read release notes before upgrades, back up both the database and the media files, and test a restore; do not let it be the only copy of photos | Learning a multi-component stateful service that the household also values |
| [PhotoPrism](https://docs.photoprism.app/) | Self-hosted photo management with indexing and search | Browses and organizes existing photo folders | Indexing is resource-intensive; database and originals both need protection. **Maintenance:** Update the application and database, re-index after major changes, back up originals, sidecar files and database | Organizing an existing photo archive |

Whatever the tool, the photos themselves need a [backup](../06-operations/backups.md) outside the server—ideally including an off-site copy—because a media server is not a backup system.

---

## Files, Documents and Synchronization

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| [Nextcloud](https://docs.nextcloud.com/) | File sync and share platform with many optional apps (calendar, contacts, office integration) | Broad hosted-cloud replacement in one platform | Large surface: web application, database, background jobs, apps and integrations to keep compatible and secure. **Maintenance:** Follow supported upgrade paths one major version at a time, update apps, run background jobs, monitor security warnings and back up files plus database | A deliberate self-hosted collaboration platform with time to maintain it |
| [Syncthing](https://docs.syncthing.net/) | Peer-to-peer continuous file synchronization between devices | No central server required; data stays on participating devices | Synchronization is not a backup—deletions and corruption replicate; conflict handling needs attention. **Maintenance:** Monitor device connectivity and folder status, keep versioning settings deliberate and back up at least one copy independently | Local-first synchronization where the lab is one participant |
| [Paperless-ngx](https://docs.paperless-ngx.com/) | Document management that scans, OCRs and indexes documents | Searchable archive of household paperwork | Contains sensitive personal documents; access control and backups are critical. **Maintenance:** Update application and dependencies, back up documents, database and the export, and restrict access tightly | Digitizing household paperwork with private access only |

Shared file services often trigger the question of a NAS. Check the [storage chapter](../02-architecture/storage.md#start-simple-and-expand-only-for-a-need) first: a file service on local storage with independent backups is frequently enough.

---

## Home Automation

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| [Home Assistant](https://www.home-assistant.io/docs/) | Local home-automation platform integrating many devices and protocols | Local control and automation across vendors; reduces dependence on many cloud apps | Quickly becomes a household dependency (lights, heating, alarms); integrations break with device or vendor changes; often needs access to the IoT segment. **Maintenance:** Review release notes before updates, back up configuration, test automations after upgrades, and keep a manual fallback for anything safety-relevant | Home automation where local control matters; place it deliberately relative to the [IoT trust role](../02-architecture/network.md#trust-roles-and-minimum-access) |

Home automation crosses network boundaries by nature. Allow only the specific controller-to-device paths that are needed instead of merging segments, and decide whether the controller belongs to the foundation tier (always on, outside lab experiments) or the lab tier (allowed to be down).

---

## Credentials and Password Management

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| [Vaultwarden](https://github.com/dani-garcia/vaultwarden/wiki) | Unofficial, community-maintained server implementation compatible with Bitwarden clients | Self-hosted password vault with familiar clients | A very high-value target; an outage or data loss can lock the household out of everything; not the official Bitwarden server. **Maintenance:** Patch promptly, restrict exposure, enforce MFA, back up and test restores frequently, and keep clients' offline caches and an independent export for recovery | Only when the owner accepts operating a critical security service; otherwise prefer a hosted or local-file password manager |

A password vault is a poor first self-hosted service: the cost of a mistake is high and the recovery path must work when the lab is down. See [human secrets and break-glass access](../03-platform-services/secrets.md#human-secrets-and-break-glass-access).

---

## Developer Services

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| [Forgejo](https://forgejo.org/docs/latest/) / [Gitea](https://docs.gitea.com/) | Lightweight self-hosted Git forges with issues, pull requests and CI integrations | Private repositories and CI runners inside the lab; learning forge administration | Another stateful service holding valuable code; if automation depends on it, it must be recovered first. **Maintenance:** Update, back up repositories and database, protect runner credentials and test restore | Learning or isolated lab development; keep authoritative copies of important repositories elsewhere too |
| Container registry, e.g. [Harbor](https://goharbor.io/docs/) | Self-hosted image registry with access control and optional scanning | Local image caching and private images for lab workloads | Storage growth, certificates and authentication; the platform may depend on it during recovery. **Maintenance:** Garbage-collect, monitor storage, update and back up; plan how workloads pull images when the registry is down | Private images or learning registry operations |

Hosted platforms such as GitHub remain a good default for authoritative repositories and CI; a self-hosted forge is most useful as a learning exercise or for lab-only work. Avoid circular dependencies: the automation that rebuilds the lab should not depend on a service that only exists inside the lab.

---

## Dashboards and Status Pages

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| [Homepage](https://gethomepage.dev/) | Configurable start page linking to services, with optional status widgets | One entry point for household users and the administrator | Widgets may need API tokens for other services; another web application to expose and update. **Maintenance:** Update, scope widget tokens read-only, and keep the configuration versioned | A friendly entry point for several services |
| [Uptime Kuma](https://github.com/louislam/uptime-kuma/wiki) | Self-hosted uptime and status checks with notifications | Simple availability checks and status pages without a full monitoring stack | Shares the lab's failure domain unless placed elsewhere. **Maintenance:** Update, keep notification channels tested, and place it where it can observe the failures you care about | Lightweight checks before or alongside a fuller [observability stack](../06-operations/observability.md) |

---

## Choosing a First Service

Start with something that teaches the platform without putting the household at risk, then add one useful service at a time:

1. **A disposable learning workload** to exercise deployment, ingress, [DNS](../03-platform-services/dns.md) and [certificates](../03-platform-services/certificates.md) end to end.
2. **A low-risk useful service**, such as a start page or a media server whose files are also stored elsewhere, to practise the [service record](./workloads.md#the-service-record), updates and monitoring.
3. **A stateful household service** only after backups and restore tests are part of the routine—for example, a photo library with an independent, tested backup.
4. **Critical services**—password vaults, home automation that controls important devices—last, and only when the operational routine has proven reliable.

Resist installing many services at once. Each one adds updates, exposure and recovery work; a few well-operated services teach more about operations than a long list nobody can maintain.

---

## Implementation and Verification

For each selected service, fill in the [service record](./workloads.md#the-service-record), deliver it through the chosen [deployment workflow](../04-automation/deployment.md), and verify it as described in the [workload checks](./workloads.md#implementation-and-verification): health, allowed and denied access, authentication, backup and restore, and behavior when its host or dependencies restart. Check the official documentation for current installation, upgrade and backup guidance; product capabilities, licensing and maturity change over time, and this survey is not a substitute for verifying them.

The services actually installed, their versions, exposure and data locations are private implementation details and are recorded only in the private repository.

## Further Reading

Product-specific documentation is linked alongside each service option above. For broader application-security context, see OWASP's [Top 10](https://owasp.org/www-project-top-ten/). This chapter is a selection guide, not an exhaustive directory or a replacement for each project's deployment and security documentation; see [Goals and Scope](../01-foundations/goals-and-scope.md#define-what-is-out-of-scope).
