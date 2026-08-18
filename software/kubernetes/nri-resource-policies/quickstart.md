# Quick Start: Isolating a Latency-Critical Service with Balloons

Two identical Redis pods run on one node under the same noisy
neighbor. One is placed in a dedicated CPU pool; the other shares CPUs
with the noise.

This example can be used as a starting point in getting familiar with
the balloons policy and testing how different configuration options
affect latencies.

## Prerequisites

- A single-node Kubernetes cluster with the Balloons policy installed. See the
  [install steps](README.md).
- A node with at least 6 CPUs.
- `kubectl` pointing at the cluster.

## 1. Apply the policy

Three CPU pools: `dedicated` (selected by a pod label), `shared` (everything
else in the `default` namespace), and the implicit reserved pool for system
pods. Each work pool is fixed at 2 CPUs, so contention is deterministic.

```bash
cat > balloons-quickstart.yaml <<'EOF'
apiVersion: config.nri/v1alpha1
kind: BalloonsPolicy
metadata:
  name: default
  namespace: kube-system
spec:
  pinCPU: true
  reservedResources:
    cpu: cpuset:0-1
  balloonTypes:
  - name: dedicated
    minCPUs: 2
    maxCPUs: 2
    preferNewBalloons: true
    matchExpressions:
    - key: pod/labels/balloon
      operator: In
      values: ["dedicated"]
  - name: shared
    minCPUs: 2
    maxCPUs: 2
    namespaces: ["default"]
EOF
kubectl apply -f balloons-quickstart.yaml
```

The Helm chart ships a default `BalloonsPolicy`, so `kubectl apply` updates it
and prints a one-line warning about a missing annotation. That is expected and
harmless; the config is applied.

## 2. Deploy the workloads

Two Redis pods and one noisy neighbor. `redis-dedicated` carries the
`balloon: dedicated` label and lands in its own pool. `redis-shared` has no
label and shares the `shared` pool with `noisy`, which keeps 8 CPU-burning
loops running.

```bash
cat > quickstart-pods.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: redis-dedicated
  labels:
    balloon: dedicated
spec:
  containers:
  - name: redis
    image: docker.io/library/redis:7-alpine
---
apiVersion: v1
kind: Pod
metadata:
  name: redis-shared
spec:
  containers:
  - name: redis
    image: docker.io/library/redis:7-alpine
---
apiVersion: v1
kind: Pod
metadata:
  name: noisy
spec:
  containers:
  - name: burn
    image: docker.io/library/busybox:stable
    command: ["sh", "-c", "for i in $(seq 1 8); do while true; do :; done & done; wait"]
EOF
kubectl apply -f quickstart-pods.yaml
kubectl wait --for=condition=Ready --timeout=120s pod/redis-dedicated pod/redis-shared pod/noisy
```

## 3. Confirm which containers run on which CPUs

There are two simple ways to check.

**Per container (most direct).** Each container reports the CPUs it may run on.
No extra tools needed:

```bash
for p in redis-dedicated redis-shared noisy; do
  echo -n "$p: "
  kubectl exec "$p" -- grep Cpus_allowed_list /proc/1/status
done
```

Example output:

```text
redis-dedicated: Cpus_allowed_list:	10-11
redis-shared: Cpus_allowed_list:	4-5
noisy: Cpus_allowed_list:	4-5
```

`redis-dedicated` has its own CPUs. `redis-shared` and `noisy` share the same
pair. That shared pair is the contention this example creates. The exact CPU
numbers vary per run.

**Per pool (from NodeResourceTopology).** The plugin publishes each balloon and
its CPUs as a NodeResourceTopology object, so you can see the pools cluster-wide
(requires `jq`):

```bash
kubectl get noderesourcetopologies -o json \
  | jq -r '.items[].zones[] | select(.type=="balloon")
           | .name + "\t" + ([.attributes[] | select(.name=="cpuset").value][0])'
```

Example output:

```text
reserved[0]	0-1
default[0]	
dedicated[0]	10-11
shared[0]	4-5
```

`dedicated[0]` and `shared[0]` hold different CPUs; `reserved[0]` is the pool
for system pods.

## 4. Measure tail latency

Run the built-in `redis-benchmark` inside each Redis pod and read the latency
summary. The `p99` column is the 99th-percentile latency.

```bash
echo "== dedicated =="
kubectl exec redis-dedicated -- redis-benchmark -n 200000 -c 50 -t get 2>&1 \
  | grep -A2 "latency summary"
echo "== shared =="
kubectl exec redis-shared -- redis-benchmark -n 200000 -c 50 -t get 2>&1 \
  | grep -A2 "latency summary"
```

## 5. Clean up

```bash
kubectl delete -f quickstart-pods.yaml
kubectl delete -f balloons-quickstart.yaml
rm -f quickstart-pods.yaml balloons-quickstart.yaml
```

## Next

To tune frequency, C-states, scheduling, or device locality per pool, see the
[Balloons page](balloons.md).
