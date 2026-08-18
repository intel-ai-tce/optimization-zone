# Choosing a Policy

Balloons and Topology-aware are alternatives. You run one policy per node, so
pick the one that matches how much control you need. If you already use the
built-in Kubernetes CPU and Memory managers, see
[choosing a policy with the Kubernetes managers](choosing-a-policy-with-kubernetes-managers.md).

## Start here

Use **[Topology-aware](topology-aware.md)** unless you have a specific reason
not to. It aligns CPU and memory to the hardware for every pod with no
per-workload configuration.

Switch to **[Balloons](balloons.md)** when you need any of these:

- dedicated CPU pools for groups of containers or pods,
- CPU frequency, C-states, Energy Performance Preference (EPP),
  Priority Core Turbo (PCT), or IRQ tuning for **any** QoS class, not
  just Guaranteed containers,
- real-time scheduling or I/O priority,
- grouping containers from different pods or namespaces into one pool,
- explicit control over which workloads share physical cores.

Topology-aware also supports CPU power, PCT, and IRQ tuning, but only for
Guaranteed containers with exclusive CPUs. Balloons applies the same tuning to
any container in a balloon.

## Side by side

| | Topology-aware | Balloons |
| --- | --- | --- |
| Main idea | Automatic hardware-aligned placement | CPU pools you define and tune |
| Setup effort | None per workload | You define balloon types |
| CPU/memory alignment | Yes, automatic | Yes, configurable |
| CPU power, PCT, IRQ tuning | Guaranteed (exclusive-CPU) containers only | Any container in a balloon |
| Grouping / mixing workloads | Per-pod alignment | Group any pods or containers into shared pools |
| Best for | Most workloads, the default | Explicit control and mixed workloads on a node |

## Rule of thumb

Do you need per-pool tuning for non-Guaranteed workloads, explicit pool shapes,
or grouping across pods? Use Balloons. Otherwise start with Topology-aware.
