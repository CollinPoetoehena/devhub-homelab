# Homelab Goals and Constraints

A homelab can be a learning environment, a self-hosted household platform, an application-development sandbox, or a mixture. Choose the purpose before equipment: a lab for disposable Kubernetes experiments has different availability needs from the only place household documents are stored.

Record:

- Learning topics and useful workloads; which experiments can be temporary.
- Budget including electricity, accessories, replacement equipment and subscriptions.
- Space, noise, cooling, cabling, and acceptable maintenance time.
- Data that matters, recovery time and acceptable data loss.
- Household dependencies that must keep working when lab hosts are off.
- Security boundaries, remote-access requirements and who administers the system.

Prefer a small stable foundation with disposable experiments. A single host can teach virtualization, automation and Kubernetes; multiple virtual nodes teach distributed behavior without independent physical resilience. Add physical hardware for a demonstrated capacity or failure-domain requirement, not just to resemble an enterprise rack.

Self-hosting is valid, but introduces patching, backup, recovery and availability responsibilities. Be explicit about which services are essential. Keep disruptive router/DHCP experiments isolated from the household network. Choose tools that leave time for the learning or services the lab exists to support.

The public reference owns these selection criteria. Personal priorities and the decisions they produce belong in private design records, as described in the [guidelines](../Repository_Guidelines.md).
