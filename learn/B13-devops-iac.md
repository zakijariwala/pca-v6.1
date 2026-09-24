---
block: B13
title: DevOps and IaC
pillars: [operational-excellence, reliability, security]
exam_guide_refs: ["3.1", "4.1", "5.1", "5.2", "6.3"]
last_verified: 2026-09-24
sources:
  - https://docs.cloud.google.com/build/docs/overview
  - https://docs.cloud.google.com/build/docs/securing-builds/configure-user-specified-service-accounts
  - https://docs.cloud.google.com/artifact-registry/docs/overview
  - https://docs.cloud.google.com/container-registry/docs/deprecations/container-registry-deprecation
  - https://docs.cloud.google.com/deploy/docs/overview
  - https://docs.cloud.google.com/deploy/docs/deployment-strategies/canary
  - https://docs.cloud.google.com/config-connector/docs/overview
  - https://docs.cloud.google.com/docs/terraform/policy-validation
  - https://docs.cloud.google.com/kubernetes-engine/enterprise/policy-controller/docs/overview
  - https://docs.cloud.google.com/source-repositories/docs
  - https://docs.cloud.google.com/binary-authorization/docs/overview
unverified: []
---
# B13 DevOps and IaC

## Chunk 1: The pipeline on Google Cloud
Problem it solves: Code must go from commit to production the same way every time.

Mental model:

| Stage | Service |
|---|---|
| Source | GitHub, GitLab, Bitbucket, or Secure Source Manager |
| Build and test | Cloud Build |
| Store artifacts | Artifact Registry |
| Release and promote | Cloud Deploy |
| Guard | Binary Authorization |

Container Registry shut down on March 18, 2025; Artifact Registry replaces it. Cloud Source Repositories closed to new customers on June 17, 2024.

Exam signals: "Modernize CI/CD for containers with a central platform" (Altostrat).

Trap: Container Registry or Cloud Source Repositories in a new design.

Pillar tie-in: Operational excellence.

Check: Where do new designs store container images?
<details><summary>Answer</summary>Artifact Registry.</details>

## Chunk 2: Cloud Build
Problem it solves: Builds on someone's laptop can't be trusted or repeated.

Mental model: Cloud Build runs a build as steps, each in its own container, defined in `cloudbuild.yaml`. Triggers start builds on commits or pull requests. Private pools run builds inside your network. Give each trigger a dedicated service account with only the roles it needs; with a user-specified service account, set a logging option such as `CLOUD_LOGGING_ONLY`.

Exam signals: "Run tests on every pull request", "build must reach private resources" → private pool.

Trap: One broad build service account shared by every pipeline.

Pillar tie-in: Security and operational excellence.

Check: What does each Cloud Build step run in?
<details><summary>Answer</summary>Its own container.</details>

## Chunk 3: Deployment strategies
Problem it solves: A bad release should hurt a few users, not all of them.

Mental model:

| Strategy | How | Rollback | Cost |
|---|---|---|---|
| Rolling | Replace instances in batches | Roll forward or back in batches | Low |
| Blue/green | Stand up the new version beside the old, switch traffic | Switch back | Double capacity during the switch |
| Canary | Send a small percentage to the new version, then increase | Send traffic back | Low |

Exam signals: "Minimize blast radius" → canary. "Instant rollback" → blue/green.

Trap: Blue/green for a stateful database schema change. Both versions must work with the same data.

Pillar tie-in: Reliability.

Check: Which strategy needs double capacity during the cutover?
<details><summary>Answer</summary>Blue/green.</details>

## Chunk 4: Cloud Deploy
Problem it solves: Promote the same release through dev, staging, and prod with approvals and history.

Mental model: A delivery pipeline lists targets in order. You create a release; Cloud Deploy rolls it out to the first target, and you promote it along the pipeline. Targets can require approval. Canary deployment works on GKE, GKE attached clusters, and Cloud Run, splitting traffic by percentage in phases.

Exam signals: "Promote the same artifact through environments", "require approval for prod", "canary on GKE or Cloud Run."

Trap: Rebuilding the image for each environment. You then test one artifact and ship another.

Pillar tie-in: Reliability and operational excellence.

Check: What does a Cloud Deploy delivery pipeline define?
<details><summary>Answer</summary>The ordered list of targets a release moves through, with options such as approvals and deployment strategy.</details>

## Chunk 5: IaC choices: Terraform and Config Connector
Problem it solves: Pick how infrastructure gets declared and reconciled.

Mental model: Terraform (P02) is the common default: plan, apply, state. Config Connector is a Kubernetes add-on that manages Google Cloud resources as Kubernetes objects, reconciled by a controller that keeps them in sync. It fits teams that run everything through Kubernetes and GitOps.

Exam signals: "Platform team manages everything with Kubernetes manifests" → Config Connector. "Multi-cloud IaC" → Terraform.

Trap: Config Connector for a team with no Kubernetes cluster.

Pillar tie-in: Operational excellence.

Check: What does Config Connector let you manage with Kubernetes tooling?
<details><summary>Answer</summary>Google Cloud resources, declared as Kubernetes custom resources.</details>

## Chunk 6: Policy as code
Problem it solves: Catch a public bucket or an open firewall before it reaches production.

Mental model: Check changes against constraints before and after deploy. `gcloud beta terraform vet` validates Terraform plans against constraints in a pipeline. Policy Controller (built on OPA Gatekeeper) audits and enforces constraints on Kubernetes objects at admission, with prebuilt policy bundles. Organization policies (B02) are the backstop in the cloud itself.

Exam signals: "Block non-compliant infrastructure before deployment."

Trap: Relying only on reviewers to spot risky settings in a 2,000-line plan.

Pillar tie-in: Security.

Check: Which tool validates a Terraform plan against constraints in CI?
<details><summary>Answer</summary>`gcloud beta terraform vet`.</details>

## Chunk 7: Designing a safe CI/CD path for GKE
Problem it solves: Put the parts together.

Mental model:
1. Pull request → Cloud Build runs tests and `terraform vet`.
2. Merge → Cloud Build builds the image, scans it, pushes to Artifact Registry, and signs an attestation.
3. Cloud Deploy creates a release → dev → staging (automatic) → prod (approval), with canary in prod.
4. Binary Authorization admits only attested images to the cluster.
5. SLO burn-rate alerts (P06) watch the canary; roll back on alert.

Exam signals: "Safe rollout", "only trusted images in production."

Trap: Skipping attestation, so a hand-pushed image can reach prod.

Pillar tie-in: Reliability and security.

Check: In this design, what stops an image built outside the pipeline from running in prod?
<details><summary>Answer</summary>Binary Authorization, which requires the pipeline's attestation at deploy time.</details>
