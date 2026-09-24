---
block: B00
title: Exam map and GCP mental model
pillars: [operational-excellence]
exam_guide_refs: [overview, case-studies, "1.2", "5.2"]
last_verified: 2026-09-24
sources:
  - https://cloud.google.com/learn/certification/cloud-architect
  - https://cloud.google.com/learn/certification/guides/professional-cloud-architect
  - https://docs.cloud.google.com/compute/docs/regions-zones
  - https://docs.cloud.google.com/compute/docs/regions-zones/global-regional-zonal-resources
  - https://docs.cloud.google.com/vpc/docs/vpc
  - https://docs.cloud.google.com/resource-manager/docs/creating-managing-projects
  - https://docs.cloud.google.com/billing/docs/how-to/modify-project
  - https://docs.cloud.google.com/shell/docs/how-cloud-shell-works
  - https://docs.cloud.google.com/shell/docs/quotas-limits
unverified:
  - Cloud Shell weekly usage cap (hours per week)
  - AWS and Azure names in the Chunk 7 table (outside Google docs; Azure Synapse Analytics positioning may have shifted)
---
# B00 Exam map and GCP mental model

## Chunk 1: The exam format
Problem it solves: You can't plan study time or pace yourself on exam day without knowing the shape of the test.

Mental model:

| Item | Standard exam |
|---|---|
| Length | 2 hours |
| Questions | 50 to 60 |
| Format | Multiple choice and multiple select |
| Languages | English, Japanese |
| Delivery | Online proctored or at a test center |
| Fee | USD 200 plus tax |
| Valid for | 2 years |
| Case studies on the exam | 2, making up 20 to 30% of content |

The exam guide publishes four case studies: Altostrat Media, Cymbal Retail, EHR Healthcare, KnightMotives Automotive. Your exam draws on two of them, and you won't know which two in advance. Study all four.

A shorter renewal exam exists for current holders: 1 hour, 25 questions, one case study focused on generative AI.

Exam signals: "Refer to the case study" in the stem means the answer must fit that company's stated requirements, not a generic best practice.

Trap: Studying two case studies and gambling on the draw.

Pillar tie-in: Operational excellence. Pacing is a plan, the same as a runbook.

Check: The exam guide lists four case studies. How many appear on your exam, and what share of content do they cover?
<details><summary>Answer</summary>Two, covering 20 to 30% of the content. You can't choose which two, so prepare all four.</details>

## Chunk 2: The six domains
Problem it solves: Weights tell you where study hours pay off.

Mental model:

| Section | Domain | Weight |
|---|---|---|
| 1 | Designing and planning a cloud solution architecture | ~25% |
| 2 | Managing and provisioning a solution infrastructure | ~18% |
| 3 | Designing for security and compliance | ~19% |
| 4 | Analyzing and optimizing technical and business processes | ~15% |
| 5 | Managing implementation | ~11% |
| 6 | Ensuring solution and operations excellence | ~12% |

Sections 1 and 3 carry 44% between them. Design and security questions dominate. The Well-Architected Framework pillars (operational excellence, security, reliability, performance optimization, cost optimization, sustainability) run through all six.

Exam signals: Business words (KPI, ROI, stakeholder, CapEx, OpEx) point to sections 1 and 4. Tool words (gcloud, Terraform, emulators, client libraries) point to section 5.

Trap: Treating section 4 as soft skills you can skip. At 15% it outweighs section 5.

Pillar tie-in: All six pillars. The exam guide names the framework as a key requirement.

Check: Which two sections together make up close to half the exam?
<details><summary>Answer</summary>Section 1, design and planning (~25%), and section 3, security and compliance (~19%). Together ~44%.</details>

## Chunk 3: Best answer, not correct answer
Problem it solves: Several options on a PCA question work. You lose points by picking one that works but ignores a stated constraint.

Mental model: Read the stem like a requirements doc. Underline each constraint: cost, latency, ops burden, compliance, timeline, existing skills. Score each option against every constraint. The best answer meets all of them with the least new moving parts. A sysadmin analogy: two fixes both bring the server back, and the one you ship is the one that doesn't need a change freeze exception.

Elimination order:
1. Drop options that break a hard constraint (region, compliance, "no code changes").
2. Drop options that add ops burden the stem didn't ask for.
3. Among what's left, prefer managed services and Google-recommended patterns.

Exam signals: "Most cost-effective", "least operational overhead", "fewest changes", "Google-recommended". Each phrase is the tiebreaker.

Trap: The most powerful or most complete option. If the stem says "minimize operational overhead", a self-managed cluster loses to a managed service even when both scale.

Pillar tie-in: Every pillar. The tiebreaker phrase tells you which pillar wins the trade-off.

Check: Two options both meet the latency target. One uses a managed service, the other a set of VMs you patch. The stem says "minimize operational overhead." Which wins and why?
<details><summary>Answer</summary>The managed service. Both meet latency, so the tiebreaker decides, and patching VMs is operational overhead the stem told you to avoid.</details>

## Chunk 4: Global network, regions, zones
Problem it solves: Placement decides availability, latency, and cost. You need the scope of each resource before you can design for failure.

Mental model: A region is a geographic area holding a set of zones. A zone is a deployment area inside a region. Zones in a region share high-bandwidth, low-latency links. Every Compute Engine resource has one of three scopes:

| Scope | Reachable from | Examples |
|---|---|---|
| Global | Any region or zone | VPC networks (with routes and firewall rules), disk images, snapshots |
| Regional | Resources in the same region | Subnets, regional managed instance groups, regional static IPs |
| Zonal | Resources in the same zone | VM instances, zonal persistent disks |

The VPC network is global and its subnets are regional. One network spans every region you use, joined by Google's own backbone. On-prem, a "network" stops at the building. Here it spans continents.

Exam signals: "Survive a zone failure" points to regional resources or multi-zone deployment. "Survive a region failure" points to multi-region design. "Single network across continents" points to one global VPC.

Trap: Assuming a zonal VM survives a zone outage because the VPC is global. The network scope doesn't lift the VM's scope.

Pillar tie-in: Reliability. Scope sets the failure domain.

Check: A team runs one VM in one zone and says it's highly available because its VPC is global. What's wrong?
<details><summary>Answer</summary>The VM is a zonal resource. If its zone fails, the VM goes with it. A global VPC doesn't change that. They need instances spread across zones, for example a regional managed instance group.</details>

## Chunk 5: Projects are the unit of everything
Problem it solves: Every resource, API, bill, and permission hangs off a project. Misread the project and you misread the whole design.

Mental model: A project is the container for service-level resources. It holds:
- Enabled APIs. A call to a disabled API fails.
- Billing, through a link to one Cloud Billing account. Without an active link, paid services stop.
- IAM policies that grant access to its resources.
- Quotas.

Project ID rules: 6 to 30 characters, lowercase letters, digits, and hyphens. You set it at creation and can't change it. Shutting down a project puts it in a pending-deletion state for 30 days, during which you can restore it.

In AWS the account is the boundary; in Azure it's the subscription. On GCP a single organization often holds hundreds of projects, one per app per environment, because the project is cheap and it's the isolation line for billing, IAM, and quotas.

Exam signals: "Separate billing per team", "isolate dev from prod", "limit blast radius" point to separate projects.

Trap: One shared project for dev and prod with IAM to keep them apart. It works, but it shares quotas and billing, and one bad grant crosses environments.

Pillar tie-in: Security (isolation) and cost (billing attribution).

Check: Why do GCP designs create more projects than AWS designs create accounts?
<details><summary>Answer</summary>The project is the default boundary for APIs, billing, IAM, and quotas, and it's cheap to create. Splitting by app and environment gives isolation and clean cost attribution without extra tooling.</details>

## Chunk 6: Console, gcloud, Cloud Shell, APIs
Problem it solves: You need to know which tool fits a task and which one the exam expects.

Mental model: All four sit on the same Google Cloud APIs.

| Tool | Use it for | Weakness |
|---|---|---|
| Console | Exploring, one-off views | Not repeatable |
| gcloud CLI | Scripts, repeatable admin | Needs install and auth off Cloud Shell |
| Cloud Shell | Browser terminal with gcloud and auth ready | Ephemeral VM, limits below |
| APIs and client libraries | Apps and automation | Most code to write |

Cloud Shell facts:
- You get an ephemeral VM running a container, plus 5 GB of persistent `$HOME`.
- The VM stops after 40 minutes of inactivity. Sessions end after 12 hours.
- If you don't open Cloud Shell for 120 days, Google deletes `$HOME` after an email warning.
- Ephemeral mode skips the persistent disk; files vanish at session end.
- A weekly usage cap applies. [UNVERIFIED: hours per week]

Exam signals: "Repeatable", "automate", "CI pipeline" point to gcloud or infrastructure as code. "Quick check with no install" points to Cloud Shell.

Trap: Storing lab state outside `$HOME` in Cloud Shell and expecting it to survive a restart.

Pillar tie-in: Operational excellence. Repeatable tools beat console clicks.

Check: You install a tool to `/usr/local/bin` in Cloud Shell and it's gone the next day. Why?
<details><summary>Answer</summary>Only `$HOME` persists. The VM is ephemeral, so anything outside `$HOME` resets when the VM is recycled.</details>

## Chunk 7: Equivalents for AWS and Azure users
Problem it solves: Prior cloud knowledge speeds you up until a false match misleads you.

Mental model:

| Concept | Google Cloud | AWS | Azure |
|---|---|---|---|
| Isolation and billing unit | Project | Account | Subscription |
| Top of hierarchy | Organization | Organization | Management group / tenant |
| Grouping | Folder | Organizational unit | Management group |
| Private network | VPC (global) | VPC (regional) | VNet (regional) |
| VMs | Compute Engine | EC2 | Virtual Machines |
| Managed Kubernetes | GKE | EKS | AKS |
| Object storage | Cloud Storage | S3 | Blob Storage |
| Data warehouse | BigQuery | Redshift | Synapse Analytics [UNVERIFIED] |

Exam signals: Stems describing a migration from another cloud expect you to map the source service to its GCP counterpart.

Trap: The VPC row. A GCP VPC spans regions; AWS and Azure networks stop at one region. Designs ported from AWS often create one VPC per region on GCP when one global VPC would do.

Pillar tie-in: Operational excellence. Fewer networks means less to manage.

Check: A team moving from AWS plans one VPC per region on GCP "to match what we had." What would you ask them?
<details><summary>Answer</summary>Why they need separate networks. A GCP VPC is global with regional subnets, so one VPC covers every region. Separate VPCs make sense only for a real isolation need, and then they add peering or other links to manage.</details>
