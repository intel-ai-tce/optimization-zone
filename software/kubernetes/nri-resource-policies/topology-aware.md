# Topology-Aware Policy

The Topology-aware policy aligns each workload with the hardware. It reads the
node's topology (sockets, dies, NUMA nodes, L3 caches) and places a container's
CPU and memory together, as close as they fit. You get NUMA- and cache-aligned
placement without configuring anything per workload.

This is the policy to start with. If you need explicit CPU pools or per-group
power and scheduling tuning, use [Balloons](balloons.md) instead.

## What it solves

- **NUMA and cache alignment, automatically.** The [NUMA guide](../../../hardware/NUMA/README.md)
  explains why aligning CPU and memory matters and how to do it by hand.
  Topology-aware does the same alignment for every pod as it starts, and picks
  the tightest fit that has room. Its
  [case study](../../../hardware/NUMA/case_studies/java_server_side_workload.md)
  shows the kind of gain at stake.

- **Scales to large NUMA counts.** It builds a pool tree from the real topology
  and scores candidates, so it keeps working on systems with many NUMA nodes
  where simpler pinning breaks down.

- **Exclusive, shared, or mixed CPUs.** Give a workload its own cores, a shared
  pool, or a mix. It can also use kernel-isolated (`isolcpus`) CPUs for the
  exclusive part.

- **Multi-tier memory.** Assign workloads to the memory type they prefer across
  DRAM, HBM, and PMEM, with an optional cold-start phase pinned to PMEM.

- **Device-aligned placement.** Pick the pool nearest the devices a workload
  uses, so CPU, memory, and device stay local.

- **CPU tuning, Priority Core Turbo (PCT), and IRQ affinity
  (Guaranteed containers only).**  Topology-aware can also set CPU
  frequency, C-states, PCT, and IRQ affinity per CPU class. These
  apply only to **Guaranteed** containers that hold exclusive CPUs. To
  tune Burstable or BestEffort workloads, or shared CPU pools, use
  [Balloons](balloons.md), which applies per-pool tuning to any QoS
  class.

## How it decides

For each container it filters out pools that lack free capacity, scores the
rest, and picks the best. Scoring prefers tighter alignment (lower latency),
more free capacity, and better device locality. Pools lower in the tree mean
stricter alignment; higher pools fit more but relax alignment.

## Reference

For installation, configuration, and cookbooks, see the upstream
[Topology-aware documentation](https://containers.github.io/nri-plugins/stable/docs/resource-policy/policy/topology-aware.html).
