# NRI Resource-Policy Plugins

The NRI resource-policy plugins apply hardware-aware placement to containers in
Kubernetes. This page covers what they are and how to get one running. The
policy pages ([Balloons](balloons.md), [Topology-aware](topology-aware.md))
cover when and why to use each.

## What is NRI

[NRI (Node Resource Interface)](https://containers.github.io/nri-plugins/stable/docs/resource-policy/introduction.html)
is a plugin interface in the container runtime. A resource-policy plugin
receives each container's lifecycle events and decides its CPU, memory, and
device affinity before it starts. This happens on the node, below the
Kubernetes scheduler.

## How it runs

- One plugin runs as a **DaemonSet**, one pod per node.
- It talks to the container runtime over NRI: **containerd 1.7+** or
  **CRI-O 1.26+**, with NRI enabled (the default in current containerd).
- You run **one policy per node**. Balloons and Topology-aware are
  alternatives, not layers.

## Prerequisites

- Kubernetes 1.24 or newer (1.27+ for the PCT flow).
- A container runtime with NRI enabled: containerd 1.7+ or CRI-O 1.26+.
- Helm 3+.

See the upstream [setup guide](https://containers.github.io/nri-plugins/stable/docs/resource-policy/setup.html)
for enabling NRI and for the full install procedure.

## Install

Add the Helm repository:

```bash
helm repo add nri-plugins https://containers.github.io/nri-plugins
helm repo update
```

Install one policy. For Topology-aware:

```bash
helm install nri-resource-policy-topology-aware \
  nri-plugins/nri-resource-policy-topology-aware --namespace kube-system
```

For Balloons:

```bash
helm install nri-resource-policy-balloons \
  nri-plugins/nri-resource-policy-balloons --namespace kube-system
```

Check the DaemonSet is ready:

```bash
# Topology-aware
kubectl -n kube-system rollout status ds/nri-resource-policy-topology-aware
# Balloons
kubectl -n kube-system rollout status ds/nri-resource-policy-balloons
```

## Configuration

You configure a policy with a custom resource, not Helm values, so changes
apply without reinstalling. Each policy has its own kind
(`BalloonsPolicy`, `TopologyAwarePolicy`). Configuration works at three scopes,
most specific wins:

- **default** — all nodes without a more specific config.
- **group** — nodes labeled `config.nri/group=$NAME`.
- **node** — a single named node.

This lets one cluster serve mixed hardware and mixed workloads. See the
upstream [configuration guide](https://containers.github.io/nri-plugins/stable/docs/resource-policy/configuration.html).

## Scope of these pages

This section orients you and points to the details. For install options, the
full configuration schema, and cookbooks, the upstream
[NRI plugins documentation](https://containers.github.io/nri-plugins/stable/)
is the source of truth.
