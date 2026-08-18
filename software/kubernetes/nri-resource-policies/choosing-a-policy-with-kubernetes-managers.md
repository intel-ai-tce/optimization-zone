# Choosing a Policy, Including the Built-in Kubernetes Managers

Kubernetes ships its own CPU manager, Memory manager, Device manager, and
Topology manager. They pin CPUs and memory and try to keep a pod's resources on
one NUMA node. If you already use them, this page shows where the NRI policies
fit and what they add.

For the choice between the two NRI policies alone, see
[choosing a policy](choosing-a-policy.md).

## Three options

- **Built-in Kubernetes managers** — CPU manager (`static` policy), Memory
  manager, Device manager, and Topology manager working together.
- **[Topology-aware](topology-aware.md)** — an NRI policy that aligns CPU and
  memory automatically.
- **[Balloons](balloons.md)** — an NRI policy built around CPU pools you define.

## Topology-aware vs. the built-in managers

Topology-aware is close to a drop-in replacement for the built-in CPU and
Memory managers. The intent is the same: place a workload's CPU and memory
together on aligned hardware. The differences are in the allocation:

- **Smarter allocation.** It builds a pool tree from the real topology
  and scores candidates, rather than following the managers' fixed rules.
- **Scales further.** The built-in managers already struggle to align well
  around 8 NUMA nodes. Topology-aware keeps working at 8 nodes and beyond.
- **Burstable container locality.** CPU, memory and device locality of
  Burstable containers is optimized as long as there are enough
  resources in hardware topology branches.
- **More tuning for Guaranteed containers.** It can also set CPU
  frequency, C-states, Priority Core Turbo (PCT), and IRQ affinity for
  Guaranteed containers with exclusive CPUs. The built-in managers do
  not offer these.

If your goal is aligned placement and you find the built-in managers limiting on
larger systems, Topology-aware is the natural step up.

## Balloons vs. the built-in managers

The balloons policy has different semantics. Instead of per-pod
pinning rules, you partition the node into CPU pools, pre-allocated
and dynamically created, and assign containers and pods to them:

- **Group and mix freely.** Put containers from the same or different pods,
  applications, or namespaces into the same pool, or keep them apart. This
  grouping is more flexible than the built-in managers allow.
- **More to tune, for any QoS class.** Each pool can set CPU
  frequency, C-states, Energy Performance Preference (EPP), PCT,
  real-time scheduling, I/O priority, IRQ affinity, and device
  locality. This tuning applies to any container in a pool, not just
  Guaranteed ones. The built-in managers do not offer these, and
  Topology-aware applies the CPU, PCT, and IRQ tuning only to
  Guaranteed containers.

Choose Balloons when you need that grouping flexibility or the extra tuning.

## Which to use

| Situation | Use |
| --- | --- |
| Aligned CPU/memory, small NUMA count, happy with built-ins | Built-in managers |
| Aligned CPU/memory, larger NUMA count, smarter allocation, more controls, Guaranteed tuning | Topology-aware |
| Aligned CPU/memory, CPU pools, container/pod grouping, tuning all QoS classes | Balloons |

The NRI policies are alternatives to the built-in managers for these tasks, not
additions on top. Run one approach per node.
