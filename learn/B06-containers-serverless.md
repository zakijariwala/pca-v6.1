---
block: B06
title: Containers and serverless
pillars: [operational-excellence, cost, reliability]
exam_guide_refs: ["1.3", "2.3"]
last_verified: 2026-09-24
sources:
  - https://docs.cloud.google.com/kubernetes-engine/docs/concepts/choose-cluster-mode
  - https://docs.cloud.google.com/kubernetes-engine/docs/concepts/autopilot-overview
  - https://cloud.google.com/kubernetes-engine/pricing
  - https://docs.cloud.google.com/kubernetes-engine/docs/concepts/workload-identity
  - https://docs.cloud.google.com/run/docs/functions/comparison
  - https://cloud.google.com/run/pricing
  - https://docs.cloud.google.com/run/docs/configuring/connecting-vpc
  - https://docs.cloud.google.com/appengine/docs/the-appengine-environments
unverified: []
---
# B06 Containers and serverless

## Chunk 1: GKE Standard vs Autopilot
Problem it solves: You want Kubernetes without owning more of it than you must.

Mental model:

| | Autopilot | Standard |
|---|---|---|
| Nodes | Google manages and bin-packs | You manage node pools |
| Billing (general-purpose) | Per Pod resource requests, per second | Per node VM |
| Unused node capacity | Not billed | You pay for it |
| Control | Less: some node-level settings locked | Full |

Autopilot sets default requests when a Pod has none and raises requests below its minimums.

Exam signals: "Minimize cluster operations" points to Autopilot. "Custom node configuration, privileged DaemonSets" points to Standard.

Trap: Standard for a small team with no Kubernetes ops experience.

Pillar tie-in: Operational excellence and cost.

Check: Under Autopilot's Pod-based billing, do you pay for idle space on a node?
<details><summary>Answer</summary>No. You pay for Pod resource requests, not the node.</details>

## Chunk 2: Cluster shape
Problem it solves: Control plane and nodes must survive a zone failure.

Mental model: A regional cluster runs control plane replicas and nodes across zones in a region. A zonal cluster has one control plane in one zone. Node pools group nodes with the same configuration, such as one pool for general work and one with GPUs.

Exam signals: "Cluster must survive a zone outage" points to a regional cluster.

Trap: A zonal cluster for production because it looks cheaper.

Pillar tie-in: Reliability.

Check: What's the difference in control plane between a zonal and a regional GKE cluster?
<details><summary>Answer</summary>Zonal: one control plane in one zone. Regional: replicated across zones in the region.</details>

## Chunk 3: Workload Identity Federation for GKE
Problem it solves: Pods need to call Google APIs without key files mounted as secrets.

Mental model: Grant IAM roles to a principal that represents a Kubernetes ServiceAccount. Pods running as that ServiceAccount get short-lived Google tokens. The principal identifier includes the project number, not the project ID.

Exam signals: "Pods need access to Cloud Storage", "no service account keys in the cluster."

Trap: Storing a service account key in a Kubernetes Secret.

Pillar tie-in: Security.

Check: A Pod must read a bucket. What's the keyless setup?
<details><summary>Answer</summary>Workload Identity Federation for GKE: grant the bucket role to the IAM principal for the Pod's Kubernetes ServiceAccount.</details>

## Chunk 4: Cloud Run services, jobs, and functions
Problem it solves: Run a container with no cluster at all.

Mental model:

| | Runs | Ends |
|---|---|---|
| Cloud Run service | Container that serves requests; scales to zero | Never; scales with traffic |
| Cloud Run job | Container that runs to completion | When the task finishes |
| Cloud Run functions | Your function code, built into a Cloud Run service | Same as a service |

Cloud Run functions is the new name for Cloud Functions (2nd gen); it runs on Cloud Run. Cloud Run bills request-based (charged while handling requests) or instance-based. Minimum instances cut cold starts and cost idle time.

For private networking, Direct VPC egress sends traffic into a VPC without a Serverless VPC Access connector.

Exam signals: "Event-driven", "stateless HTTP", "scale to zero" point to Cloud Run. "Nightly batch in a container" points to a Cloud Run job.

Trap: A GKE cluster for one stateless API with spiky traffic.

Pillar tie-in: Cost and operational excellence.

Check: A containerized task processes a file and exits, once an hour. Service or job?
<details><summary>Answer</summary>A Cloud Run job, triggered on a schedule.</details>

## Chunk 5: App Engine positioning
Problem it solves: Legacy apps still run on App Engine. New ones shouldn't start there.

Mental model: App Engine offers a standard and a flexible environment. Google recommends Cloud Run for new applications; it covers the same workloads with more flexibility. Expect App Engine in exam answers as an existing system or a distractor.

Exam signals: "Existing App Engine app" means keep it or migrate it. "New app" points to Cloud Run.

Trap: Choosing App Engine for a greenfield service.

Pillar tie-in: Operational excellence.

Check: What does Google recommend over App Engine for new apps?
<details><summary>Answer</summary>Cloud Run.</details>

## Chunk 6: The compute decision tree
Problem it solves: A scenario gives you requirements. You pick the platform in under a minute.

Mental model: Ask in order:
1. Can it run as a stateless container that handles requests or runs to completion? → Cloud Run.
2. Does it need Kubernetes features: many services, sidecars, custom controllers, stateful sets, portability across clouds? → GKE (Autopilot unless you need node control).
3. Does it need OS control, special licenses, or can't be containerized? → Compute Engine.
4. Is it a small piece of code reacting to an event? → Cloud Run functions.

Exam signals: "Minimize operational overhead" pushes you up the list. "Existing Kubernetes manifests" points to GKE.

Trap: Jumping to GKE because the company "uses containers." Cloud Run runs containers too.

Pillar tie-in: Operational excellence and cost.

Check: A team has 40 microservices with Helm charts and a service mesh. Which platform?
<details><summary>Answer</summary>GKE. The Kubernetes tooling and mesh point there; Autopilot unless they need node-level control.</details>
