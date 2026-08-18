# Kubernetes Optimization

Most tuning guides in the Optimization Zone assume you control the host: you
pin threads, set the CPU governor, disable deep C-states, or align a workload
to a NUMA node. In Kubernetes you usually cannot do that by hand. The scheduler
places pods on nodes, and the built-in CPU and Memory managers know little
about NUMA distance, cache domains, device locality, Priority Core Turbo (PCT),
uncore and C-states, or real-time scheduling.

This section is about closing that gap. It shows how to apply the same
hardware-aware optimizations to containerized workloads, on both cloud and edge
nodes, without editing each node by hand.

## The mechanism: NRI resource-policy plugins

The [NRI (Node Resource Interface) resource-policy plugins](https://github.com/containers/nri-plugins)
run as a DaemonSet on each node. They hook into the container runtime
(containerd or CRI-O) and set CPU, memory, and device affinity for each
container as it starts. They can also tune CPU frequency, C-states, IRQ
affinity, and process scheduling per group of containers.

Two policies cover most needs:

- **[Topology-aware](nri-resource-policies/topology-aware.md)** — automatic
  NUMA-, cache-, and device-aligned placement. No per-workload configuration
  needed. Start here if you want the wins from the [NUMA guide](../../hardware/NUMA/README.md)
  without hand-pinning.
- **[Balloons](nri-resource-policies/balloons.md)** — group containers into CPU
  pools ("balloons") and tune each pool: dedicated CPUs, frequency, C-states,
  PCT high-priority cores, real-time scheduling, device locality. Choose this
  when you need explicit control per group of workloads.

Not sure which fits? See [choosing a policy](nri-resource-policies/choosing-a-policy.md),
or, if you already use the built-in Kubernetes managers,
[choosing a policy with the Kubernetes managers](nri-resource-policies/choosing-a-policy-with-kubernetes-managers.md).

New here? Start with the [foundation page](nri-resource-policies/README.md), or
jump to the [quick start](nri-resource-policies/quickstart.md).

## What maps to what

If you came from a hardware or software guide, this is where its host-level
lever shows up in Kubernetes:

| Optimization Zone topic | NRI policy feature |
| --- | --- |
| [NUMA alignment](../../hardware/NUMA/README.md) | Topology-aware automatic alignment; Balloons topology balancing |
| Memory bandwidth vs. latency trade-off | Balloons spreading across NUMA nodes and sockets, or local-only |
| Multi-tier memory (DRAM, HBM, PMEM) | Topology-aware multi-tier allocation |
| [Priority Core Turbo](../../hardware/priority_core_turbo/README.md) | Balloons CPU classes, any QoS; Topology-aware CPU classes, Guaranteed containers only |
| [CPU frequency, Energy Performance Preference (EPP), governor](../common/README.md) | Balloons CPU classes: min/max frequency, governor, EPP, uncore |
| C-states and wakeup latency | Balloons CPU classes: disabled C-states |
| [Hyper-threading choices](../common/README.md) | Topology-aware HT control; Balloons hyper-thread-aware sharing |
| Dedicated cores / isolation | Balloons dedicated pools; Topology-aware exclusive CPUs and isolcpus |
| Burst headroom without noisy neighbors | Balloons idle-CPU sharing |
| Real-time scheduling and I/O priority | Balloons scheduling classes (SCHED_FIFO, ioClass, ioPriority) |
| IRQ affinity | Balloons IRQ affinity, any QoS; Topology-aware IRQ affinity, Guaranteed containers only |
| PCI / GPU / NIC / accelerator locality | Balloons device locality; Topology-aware device alignment |

Each feature is documented upstream. The pages in this section explain when to
reach for it and link to the details.

## Who this is for

- **AI and inference** — put host-side serial work (tokenization, dispatch) on
  PCT high-priority cores and keep it local to the GPU or NIC.
- **Databases** — align CPU and memory to cut cross-NUMA traffic and tail
  latency; isolate from noisy neighbors.
- **Web services and proxies** — let bursty handlers use idle CPUs without
  hurting a latency-critical neighbor.
- **Real-time and industrial / edge** — dedicated cores, no deep C-states,
  real-time scheduling, and IRQs steered off the control loop.

## Measure your results

Improvements depend on the workload. Use a profiler such as
[VTune Profiler](../../tools/vtune/README.md) or [PerfSpect](../../tools/perfspect/README.md)
to see where time goes and to compare before and after.

> Performance varies by use, configuration, and other factors. See the
> [disclaimer](../../README.md#disclaimer).
