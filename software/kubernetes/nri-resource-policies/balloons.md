# Balloons Policy

The Balloons policy groups containers into CPU pools called
*balloons*. A container runs on the CPUs of its balloon, optionally on
free CPUs outside balloons, but never on CPUs of other balloons. Each
balloon can be tuned on its own: how many CPUs, at what frequency,
which C-states, how they are scheduled, and which devices they sit
close to.

Reach for Balloons when you need explicit control over how groups of workloads
share a node. For automatic alignment with no per-workload setup, see
[Topology-aware](topology-aware.md) instead.

A balloon can hold containers of any QoS class: BestEffort, Burstable,
or Guaranteed. Its tuning (frequency, C-states, Priority Core Turbo
(PCT), IRQ affinity, scheduling) applies to all of them. This is a key
difference from Topology-aware, whose CPU tuning, PCT, and IRQ
affinity apply only to Guaranteed containers with exclusive CPUs.

## What it solves

Each item below is a common host-level tuning task and where Balloons handles
it in Kubernetes. Follow the links for the configuration.

- **Isolate a latency-critical workload.** Give it dedicated CPUs, stop it
  sharing L2 cache with other workloads, and disable deep C-states so cores do
  not sleep. See the upstream
  [latency-critical recipe](https://containers.github.io/nri-plugins/stable/docs/resource-policy/policy/balloons.html#latency-critical-containers).

- **Run hot serial threads at top turbo.** Put them on Priority Core Turbo
  high-priority cores while the rest of the node stays at base frequency. See
  the [PCT guide](../../../hardware/priority_core_turbo/README.md) for the hardware
  side and the upstream
  [PCT quick start](https://containers.github.io/nri-plugins/stable/docs/resource-policy/policy/howto/balloons-pct-quickstart.html)
  for the Kubernetes side.

- **Get more memory bandwidth.** Spread a balloon's CPUs across NUMA nodes or
  sockets to use more memory channels, or keep it local for lowest latency. See
  the upstream
  [memory bandwidth recipe](https://containers.github.io/nri-plugins/stable/docs/resource-policy/policy/balloons.html#maximum-memory-bandwidth-containers)
  and the [NUMA guide](../../../hardware/NUMA/README.md).

- **Let bursty workloads use idle CPUs safely.** A balloon can borrow otherwise
  idle CPUs within a core, L2 domain, NUMA node, or package, and you control
  which workload types share a physical core. See the upstream
  [hyper-thread sharing recipe](https://containers.github.io/nri-plugins/stable/docs/resource-policy/policy/balloons.html#workload-aware-hyperthread-sharing).

- **Set frequency, governor, EPP, and C-states per group.** The host-wide knobs
  in the [common recommendations](../../common/README.md) become
  per-balloon CPU classes. See the upstream
  [CPU tuning options](https://containers.github.io/nri-plugins/stable/docs/resource-policy/policy/balloons.html#cpu-tuning).

- **Apply real-time scheduling and I/O priority.** Assign a balloon a scheduling
  class such as `SCHED_FIFO` with a priority, plus an I/O class and priority.
  Useful for control loops and other real-time work. See the upstream
  [CPU tuning options](https://containers.github.io/nri-plugins/stable/docs/resource-policy/policy/balloons.html#cpu-tuning).

- **Steer IRQs off critical cores.** Move device interrupts away from the CPUs
  running a latency-critical balloon.

- **Keep CPUs close to a device.** Place a balloon on the CPUs nearest a GPU,
  NIC, or accelerator to cut access latency. Useful for AI inference and packet
  processing.

## Try it

The upstream [PCT quick start](https://containers.github.io/nri-plugins/stable/docs/resource-policy/policy/howto/balloons-pct-quickstart.html)
is the fastest hands-on path and shows a measurable difference between tuned and
untuned pods. For a local isolation example, see the
[quick start](quickstart.md) in this section.

## Reference

Full configuration, all balloon and CPU-class fields, and more recipes are in
the upstream [Balloons documentation](https://containers.github.io/nri-plugins/stable/docs/resource-policy/policy/balloons.html).
