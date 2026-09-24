---
block: P04
title: GKE operations
pillars: [reliability, operational-excellence]
exam_guide_refs: ["2.3", "4.1", "6.4"]
last_verified: 2026-09-24
sources:
  - https://docs.cloud.google.com/kubernetes-engine/docs/concepts/alias-ips
  - https://docs.cloud.google.com/kubernetes-engine/docs/how-to/flexible-pod-cidr
  - https://docs.cloud.google.com/kubernetes-engine/docs/concepts/release-channels
  - https://docs.cloud.google.com/kubernetes-engine/docs/concepts/maintenance-windows-and-exclusions
  - https://docs.cloud.google.com/kubernetes-engine/docs/concepts/horizontalpodautoscaler
  - https://docs.cloud.google.com/kubernetes-engine/docs/concepts/verticalpodautoscaler
  - https://docs.cloud.google.com/kubernetes-engine/docs/concepts/cluster-autoscaler
  - https://docs.cloud.google.com/kubernetes-engine/docs/concepts/node-auto-provisioning
  - https://docs.cloud.google.com/kubernetes-engine/docs/concepts/gateway-api
  - https://docs.cloud.google.com/kubernetes-engine/docs/troubleshooting/crashloopbackoff-events
  - https://docs.cloud.google.com/kubernetes-engine/docs/troubleshooting/oom-events
unverified: []
---
# P04 GKE operations

## Chunk 1: VPC-native clusters and IP exhaustion
Problem it solves: A cluster stops adding nodes while the subnet still has free addresses.

Mental model: VPC-native clusters give Pods IPs from a secondary range on the subnet (alias IPs). Each node reserves a block sized for double its max-Pods-per-node setting. At the default of 110 Pods per node, each node takes a /24. A /21 Pod range then fits only 8 nodes. When the Pod range runs out of blocks, you get `IP_SPACE_EXHAUSTED`.

Fixes: add Pod ranges (discontiguous multi-Pod CIDR), lower max Pods per node on new node pools, or rebuild with a bigger range.

Exam signals: "Cluster can't scale, subnet has free IPs."

Trap: Expanding the primary subnet range. Pods don't use it.

Pillar tie-in: Reliability.

Check: With max Pods per node at 110 and a /21 Pod range, how many nodes fit?
<details><summary>Answer</summary>8. Each node takes a /24, and a /21 holds 2^(24-21) = 8 of them.</details>

## Chunk 2: Release channels and maintenance
Problem it solves: Upgrades must happen, but not during your peak.

Mental model: Release channels set how fast versions reach you: Rapid, Regular, Stable, and (Standard only) Extended. A version moves Rapid → Regular → Stable as it proves stable. Extended keeps a minor version for up to 24 months after it reaches Regular. Maintenance windows say when GKE may upgrade; maintenance exclusions say when it may not. Scope-limited exclusions can block minor or node upgrades for up to 180 days or until end of life.

Exam signals: "No upgrades during Black Friday" points to a maintenance exclusion. "Slow, well-tested versions" points to Stable.

Trap: Pinning a static version forever to avoid upgrades. It reaches end of support and gets upgraded anyway.

Pillar tie-in: Reliability.

Check: How do you stop GKE upgrades during a two-week sales event?
<details><summary>Answer</summary>A maintenance exclusion covering those two weeks.</details>

## Chunk 3: The four autoscalers
Problem it solves: Scale Pods and nodes to load without paying for idle capacity.

Mental model:

| Scaler | Changes | On |
|---|---|---|
| Horizontal Pod Autoscaler (HPA) | Pod count | CPU, memory, custom or external metrics |
| Vertical Pod Autoscaler (VPA) | Pod requests and limits | Observed usage |
| Cluster autoscaler | Node count per pool | Pending Pods, idle nodes |
| Node auto-provisioning (node pool auto-creation) | Creates new node pools | Pending Pods that fit no pool |

HPA and VPA complement each other when they don't act on the same metric: HPA for count, VPA for size.

Exam signals: "Traffic doubles at lunch" points to HPA. "Requests set wrong, OOM kills" points to VPA.

Trap: HPA and VPA both scaling on CPU for the same workload. They fight.

Pillar tie-in: Cost and reliability.

Check: Pods sit Pending because no node has enough memory. Which scaler adds capacity?
<details><summary>Answer</summary>The cluster autoscaler adds nodes to a pool; node auto-provisioning creates a new pool if no existing shape fits. In Autopilot, GKE handles this for you.</details>

## Chunk 4: PodDisruptionBudgets
Problem it solves: Upgrades and node drains take down every replica at once.

Mental model: A PodDisruptionBudget sets how many replicas must stay up during voluntary disruptions such as upgrades and scale-down. GKE respects it when draining nodes, within limits.

Exam signals: "Upgrades cause brief outages."

Trap: Running one replica and adding a PDB. With one replica, any drain means downtime; add replicas first.

Pillar tie-in: Reliability.

Check: What does a PDB protect against?
<details><summary>Answer</summary>Too many replicas going down at once during voluntary disruptions such as node drains and upgrades.</details>

## Chunk 5: Gateway API and Ingress
Problem it solves: Expose services through a Google Cloud load balancer from Kubernetes objects.

Mental model: Ingress is the older API. Gateway API splits roles: a platform team owns the Gateway (from a GatewayClass such as `gke-l7-global-external-managed`), app teams own HTTPRoutes. The GKE Gateway controller builds the load balancer. Gateway doesn't infer health checks from Pod readiness; if your app doesn't return 200 on `GET /`, add a HealthCheckPolicy.

Exam signals: "Platform team controls the load balancer, app teams control routes."

Trap: Migrating from Ingress to Gateway and seeing backends go unhealthy because the health check path changed.

Pillar tie-in: Operational excellence.

Check: After moving to Gateway API, backends show unhealthy. The app serves health on `/healthz`. Fix?
<details><summary>Answer</summary>Add a HealthCheckPolicy pointing to `/healthz`. Gateway doesn't infer health checks the way Ingress did.</details>

## Chunk 6: First response on a sick workload
Problem it solves: A Pod is broken. You need the cause in minutes.

Mental model:

| Status | Meaning | First check |
|---|---|---|
| CrashLoopBackOff | Container starts and exits, over and over | `kubectl logs POD --previous`, then events |
| OOMKilled (exit 137) | Container exceeded its memory limit | Memory limit vs actual use |
| Pending | Scheduler can't place the Pod | `kubectl describe pod`: insufficient CPU or memory, node selectors, Pod limit per node |
| ImagePullBackOff | Can't fetch the image | Image name, registry permissions |

```bash
kubectl get pods
kubectl describe pod POD
kubectl logs POD --previous
kubectl get events --sort-by=.lastTimestamp
```

Exam signals: "Pod restarts every few seconds", "Pods never start."

Trap: Deleting the Pod to "fix" CrashLoopBackOff. The replacement crashes the same way.

Pillar tie-in: Operational excellence.

Check: A container exits with code 137. What happened?
<details><summary>Answer</summary>The kernel OOM killer ended it: the container went over its memory limit.</details>

## Chunk 7: Autopilot constraints
Problem it solves: Knowing what Autopilot won't let you do before you commit.

Mental model: Autopilot manages nodes, so it restricts node-level access: no SSH to nodes and limits on privileged workloads. It enforces minimum and maximum resource requests and applies defaults when a Pod sets none.

Exam signals: "Needs privileged DaemonSets or node kernel tuning" points to Standard.

Trap: Choosing Autopilot for a security agent that needs host-level access.

Pillar tie-in: Operational excellence.

Check: A Pod in Autopilot declares no resource requests. What happens?
<details><summary>Answer</summary>Autopilot applies default requests to it.</details>
