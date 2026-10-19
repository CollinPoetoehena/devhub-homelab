# Disaster Recovery

> **Part 6 · Operations** — [Part overview](./README.md) · Previous: [Backups](./backups.md) · Next: [Maintenance](./maintenance.md)

Backups make recovery *possible*; disaster recovery makes it *work*. A homelab that has backups but no plan for using them often discovers during an outage that the backup server ran on the failed host, the decryption key was on the broken laptop, or nobody remembers the order in which things must start. This chapter turns the [recovery objectives](../01-foundations/requirements.md#availability-and-recovery-requirements) into a plan: which failures are expected, what the recovery order is, what must be available when everything else is down, and how the plan is rehearsed.

The individual subjects own their specific recovery options—gateway recovery in the [network chapter](../02-architecture/network.md#availability-backups-and-recovery), backup tools and copies in [Backups](./backups.md), quorum and host failure in [Compute](../02-architecture/compute.md#quorum-and-failure-domains). This chapter connects them into one recovery story and links to them rather than repeating them.

---

## Table of Contents

- [Purpose and Requirements](#purpose-and-requirements)
- [Failure Scenarios](#failure-scenarios)
- [Recovery Order](#recovery-order)
- [The Recovery Kit](#the-recovery-kit)
- [Rebuild or Restore?](#rebuild-or-restore)
- [Rehearsing Recovery](#rehearsing-recovery)
- [After an Incident](#after-an-incident)
- [Implementation and Verification](#implementation-and-verification)

---

## Purpose and Requirements

Disaster recovery for a homelab does not mean a second data center. It means knowing, before something fails, how each important component returns within its accepted [RTO](../01-foundations/requirements.md#availability-and-recovery-requirements) and with no more data loss than its RPO allows. The [design principle](../01-foundations/design-principles.md#recoverability-before-redundancy) is recoverability before redundancy: a tested plan with modest hardware is usually worth more than redundant hardware without a plan.

Typical requirements:

- The household connection can be restored within an agreed time, without depending on lab compute.
- Unique household data can be recovered after loss of a disk, a host or the whole site.
- The lab platform can be rebuilt from versioned configuration and backups.
- Recovery works with only the material that survives the failure: offline credentials, protected backups and documentation that is readable without the lab.

---

## Failure Scenarios

Plan for a small set of realistic scenarios rather than every conceivable disaster. For each, know what is lost, what recovers it and where the subject-specific details live.

| Scenario | What is affected | Recovery approach | Details |
|---|---|---|---|
| Single disk failure | Data and VMs on that disk | Replace the disk; restore from redundancy if present, otherwise from backup | [Storage](../02-architecture/storage.md), [Backups](./backups.md) |
| Hypervisor host failure | Every guest on the host, including several cluster nodes | Repair or replace the host, reinstall the hypervisor from documented/automated steps, restore guests or rebuild them from code | [Compute](../02-architecture/compute.md#quorum-and-failure-domains), [Infrastructure as Code](../04-automation/infrastructure-as-code.md) |
| Gateway failure | Internet access and routing for the whole home | Restore configuration to repaired or replacement hardware, cold spare, temporary virtual gateway or alternate uplink | [Network availability and recovery](../02-architecture/network.md#availability-backups-and-recovery) |
| Workload platform failure | All platform-hosted services | Rebuild the cluster from configuration, redeploy workloads through the delivery workflow, restore persistent data | [Platform](../02-architecture/platform.md#implementation-and-verification), [Deployment](../04-automation/deployment.md) |
| Accidental deletion or corruption | Specific data or configuration | Restore from a point-in-time backup that predates the change | [Independent copies and retention](./backups.md#independent-copies-and-retention) |
| Credential compromise | Anything the credential could reach | Revoke and rotate, assess what was accessed, restore from known-good backups if needed | [Secrets](../03-platform-services/secrets.md#rotation-and-exposure), [exposure procedure](../repository-guidelines.md#if-information-is-exposed) |
| Power outage | Everything without battery backup | Orderly shutdown if a UPS signals it; ordered startup afterwards | [Scheduling](../03-platform-services/scheduling.md#safe-automatic-shutdown-and-startup), [Physical](../02-architecture/physical.md#power-accessories) |
| Loss of the site | All local equipment and local copies | Rebuild on new hardware from off-site backups, documentation and code | [Off-site copies](./backups.md#independent-copies-and-retention), [the recovery kit](#the-recovery-kit) |

Some scenarios are deliberately accepted with a long recovery time—for example, waiting for replacement hardware after a host failure. Accepting downtime is fine; being surprised by it is not.

---

## Recovery Order

Dependencies decide the order. Restoring an application before its storage, name resolution or credentials are available wastes time and can corrupt data. A generic order for a homelab is:

1. **Access and safety:** recovery credentials, decryption keys and documentation from the [recovery kit](#the-recovery-kit); power and physical connectivity.
2. **Network foundation:** gateway, switching, Wi-Fi and household DNS/DHCP, so the household is online and administration paths exist.
3. **Management path:** an administrator workstation or management VM with the automation tooling and access to backups.
4. **Compute and storage:** hypervisor hosts and storage, reinstalled or repaired.
5. **Backup system:** if the backup server runs inside the environment it protects, recover it first—or restore directly from its independent copies—so that everything else can be restored.
6. **Platform services:** DNS for the lab, secret stores or keys, certificate issuance and identity, in their [dependency order](../03-platform-services/README.md#how-the-capabilities-depend-on-each-other).
7. **Workload platform:** cluster nodes and control plane, rebuilt from configuration.
8. **Data, then applications:** restore persistent data before starting the applications that use it, then validate application behavior.
9. **Observability:** confirm monitoring sees the recovered components and that alerts work again.

Write the actual order for the installed components in the private record, including the specific dependencies between them, and keep it readable without the lab.

---

## The Recovery Kit

The recovery kit is the minimal set of material needed to start recovery when nothing in the homelab works. It must not live only inside the homelab:

- Administrator and break-glass credentials, MFA recovery codes and backup decryption keys—see [human secrets and break-glass access](../03-platform-services/secrets.md#human-secrets-and-break-glass-access).
- Gateway and network device configuration exports, plus provider handoff details, stored as described in [backups that survive the outage](../02-architecture/network.md#backups-that-survive-the-outage).
- Access to the backup copies themselves: where they are, how to reach them and how to decrypt them.
- The recovery order and the key procedures, readable offline—for example, a local clone of the private documentation or an encrypted export.
- Installation media or download sources and the versions needed to reinstall the hypervisor and essential tools.

Keep an encrypted offline copy and an off-site copy, and review the kit whenever credentials, devices or procedures change. A kit that was correct a year ago may be useless today.

---

## Rebuild or Restore?

Not everything needs to be restored from a backup. Components fall into three groups:

| Approach | What it means | Suits |
|---|---|---|
| Rebuild from code | Recreate the component from versioned infrastructure, configuration and deployment code | Stateless hosts, cluster nodes, stateless workloads, images and templates |
| Restore from backup | Recover state that cannot be recreated | Databases, files, application data, device configuration exports, state for tools that need it |
| Accept loss | Recreate manually or not at all | Experiments and anything without unique value |

The more of the homelab that can be rebuilt from code, the smaller and simpler the restore scope becomes—this is one of the strongest arguments for the [automation part](../04-automation/README.md). Git records intent, not data: rebuilding a stateful service from its manifests without restoring its data produces an empty service.

---

## Rehearsing Recovery

An untested recovery plan is an assumption. Rehearse at a scale that is safe:

- **Restore tests:** restore a representative VM, database or file set into an isolated target on a regular schedule and check that it works, not only that files exist. See [backup verification](./backups.md#implementation-and-verification).
- **Rebuild tests:** destroy and recreate a disposable cluster node or VM from code.
- **Foundation tests:** turn lab compute off and confirm household connectivity, DNS and administration still work.
- **Kit tests:** using only the recovery kit, prove that backups can be found and decrypted and that break-glass access works.
- **Tabletop exercises:** walk through a scenario on paper—"the host is dead, what do I do first?"—to find missing steps without touching anything.

Measure how long each recovery takes and compare it with the RTO. Record what was tested, when and with what result, and fix the plan when a rehearsal finds a gap. Never test a restore by overwriting the only live copy.

---

## After an Incident

After a real outage or a rehearsal, record briefly what happened, how long recovery took, what worked and what did not, and what changes follow—in the plan, the kit, the automation or the requirements. Treat this as learning, not blame. Many valuable homelab improvements come from recovering from a self-inflicted failure.

---

## Implementation and Verification

Start with the scenarios that matter most for the household and the unique data, write the recovery order for the actual components, assemble the recovery kit and schedule the first rehearsals. Track the date and result of the last successful test per scenario; an RTO or RPO that has never been measured is a hope, not a property of the system.

The public book describes the method. The actual scenarios, recovery order, kit contents and test results belong in the private record—with credentials, keys and sensitive endpoints only in protected storage.
