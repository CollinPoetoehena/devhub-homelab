# Service Scheduling and Power

Decide when each component needs to run rather than leaving every host and service powered on by default. Electricity is a recurring cost, and an always-on system that is only used at particular times may add expense without providing useful availability. Scheduling can reduce that cost, but shutdown and startup must preserve data and bring dependencies back in the right order.

---

## Table of Contents

- [Lifecycle Options](#lifecycle-options)
- [Schedule by Component](#schedule-by-component)
- [Safe Automatic Shutdown and Startup](#safe-automatic-shutdown-and-startup)
- [Measure the Result](#measure-the-result)

---

## Lifecycle Options

| Option | Example use | Benefits | Responsibilities and risks |
|---|---|---|---|
| 24/7 | Gateway, required switching/APs, or services with real continuous demand | Reachable at any time and no scheduled cold start | Ongoing energy use, patching, monitoring and exposure to failures; verify the service is truly needed continuously |
| Daytime schedule | Compute used for work or learning in defined daytime hours | Reduces idle runtime while keeping predictable availability | Must account for early/late sessions, update windows, backup completion and clean shutdown |
| Nighttime schedule | Workloads that run only overnight, such as planned batch jobs | Uses a defined window without keeping compute up all day | Respect job duration, data dependencies and recovery if a run misses its window |
| On demand | Disposable experiments or occasional workloads | Little idle consumption and clear experiment lifecycle | Startup delay and someone/system must request start; ensure required management path stays available |
| Always-on service with scheduled workload | Monitoring/control plane stays available while selected workers/experiments stop | Keeps a small useful baseline while reducing larger host draw | Persistent service/resource placement and alert behavior must be designed around missing hosts |

---

## Schedule by Component

Do not apply one schedule to the entire homelab. Essential network equipment—modem/ONT, router, necessary switches and Wi-Fi access points—normally remains on when household connectivity is expected. Compute hosts can be scheduled or started on demand if no workload needs them overnight. IoT devices often draw little power or have no safe shutdown control; decide whether to leave them on rather than adding unreliable automation.

Monitoring may need to remain available to supervise shutdown/startup and detect failures. Silence or route alerts for intentionally offline nodes during their planned window, but keep alerts for unexpected outages outside the window. Storage and backup components must be on for their jobs; complete backups and confirm success before taking a required host or disk offline. Check that DNS, DHCP, identity, time, automation and wake mechanisms do not depend solely on the host being shut down.

An energy-monitoring plug or UPS is an optional accessory; buy it only when measurement or shutdown requirements justify it. See the HLD [power accessories guidance](../../2_HLD/Placement.md#power-accessories) and the central [cost and scope guide](../../1_Goals/Cost_and_Scope.md).

---

## Safe Automatic Shutdown and Startup

Use an automatic, documented schedule rather than relying on someone to remember every manual step. Manual shutdowns are easy to forget and can interrupt writes, databases, backups or cluster membership; automation can still fail, so retain an accessible manual override and recovery path.

A safe design should:

1. Calculate expected savings from measured wall power and scheduled hours.
2. Select a maintenance window and block shutdown while critical jobs, backups, migrations or experiments are active.
3. Drain/cordon workers where appropriate, stop workloads cleanly, then request graceful guest and hypervisor shutdown through supported APIs/ACPI.
4. Verify that dependent state is flushed and services have exited before removing power. Never use a smart plug to abruptly cut power to a running host.
5. Start required infrastructure first, then hypervisors/VMs, cluster control-plane components and workloads in dependency order. Use supported automatic power-on after AC recovery, Wake-on-LAN, a scheduled management controller or a person-triggered on-demand path where available.
6. Wait for health checks and confirm critical services recover. If startup or shutdown fails, notify an always-available channel and leave the component in a safe state rather than retrying destructive actions indefinitely.

Kubernetes control-plane quorum and stateful services need special care. Do not power off voting members together without understanding the supported shutdown sequence; do not assume a cold start automatically restores databases or volumes. Test the workflow with disposable workloads first and document observed recovery, not just the schedule configuration.

---

## Measure the Result

Compare measured idle and active watts before/after scheduling and compute actual active hours per day. Include the scheduler/controller's own power draw and any energy use during startup, backup or maintenance. Review energy trends and unexpected runtime; re-evaluate schedules when workload patterns change.

State the accepted availability clearly: scheduled compute is intentionally unavailable outside its window, not an always-on service that happens to be powered off. If a component must be reachable around the clock, either keep it on or redesign dependencies so a smaller independent component provides that availability.
