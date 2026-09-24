---
block: B21
title: Mixed practice set
pillars: [security, reliability, cost, performance, operational-excellence, sustainability]
exam_guide_refs: ["1", "2", "3", "4", "5", "6"]
last_verified: 2026-09-24
sources:
  - https://cloud.google.com/learn/certification/guides/professional-cloud-architect
  - learn/ files B00 to B20 and P01 to P08, which carry the per-fact sources
unverified: []
---
# B21 Mixed practice set

20 original questions across the six sections, weighted like the exam: 5 from section 1, 4 each from sections 2 and 3, 3 from section 4, 2 each from sections 5 and 6. Four refer to case studies. None come from the real exam.

How to run it:
1. Set a timer: about 2 minutes per question, 40 minutes total.
2. Write down your answer before opening the reveal.
3. For each miss, write one line: what you picked, what's true, why you picked it. That's your input for B22.
4. Fill the score table at the end.

Repeat the set after remediation. Once you pass it cold, move to B23.

## Chunk 1: Section 1, designing and planning (5)

### Q1 | 1.3 | pillars: cost, operational-excellence
A startup runs a stateless HTTP API. Traffic is near zero at night and spikes 20x at lunch. The team has no Kubernetes experience and wants the least operational effort. Where should the API run?

A) GKE Standard with cluster autoscaling. B) Cloud Run. C) A regional managed instance group with autoscaling. D) Compute Engine VMs with a cron job that resizes them.

<details><summary>Answer</summary>
B. Why: Cloud Run scales with requests, down to zero, with no cluster to run.
A fails: adds node and cluster operations the team can't staff.
C fails: works, but VMs don't scale to zero and need image and OS upkeep.
D fails: manual scaling can't follow lunch spikes.
</details>

### Q2 | 1.3 | pillars: performance, reliability
A trading app needs relational SQL with strong consistency for users in North America, Europe, and Asia, and a 99.999% availability target. Which database?

A) Cloud SQL with cross-region read replicas. B) Bigtable with replication. C) Spanner in a multi-region configuration. D) Firestore in a multi-region location.

<details><summary>Answer</summary>
C. Why: Spanner gives relational SQL, strong consistency at global scale, and a 99.999% SLA on multi-region configurations.
A fails: replicas are asynchronous and read-only; writes go to one region.
B fails: no multi-row SQL transactions.
D fails: a document database, not relational SQL.
</details>

### Q3 | 1.2 | pillars: reliability, cost
An internal reporting tool has an RTO of 24 hours and an RPO of 24 hours. The business wants the lowest DR cost. Which pattern?

A) Hot: active-active in two regions. B) Warm: a scaled-down copy running in a second region. C) Cold: daily backups restored to a new environment on disaster. D) No DR; rely on zonal redundancy.

<details><summary>Answer</summary>
C. Why: generous RTO and RPO fit a cold pattern, the cheapest option that meets them.
A fails: pays for full standby capacity the targets don't need.
B fails: runs standby resources all the time for a one-day RTO.
D fails: zonal redundancy doesn't cover a region loss.
</details>

### Q4 | 1.4 | pillars: cost, reliability
A company has 150 TB to move to Cloud Storage within 3 weeks. Its link to the internet is 100 Mbps and can't be upgraded in time. What do you recommend?

A) `gcloud storage cp` with parallel uploads. B) Storage Transfer Service with on-prem agents. C) Transfer Appliance. D) HA VPN and `rsync`.

<details><summary>Answer</summary>
C. Why: at 100 Mbps, 100 TB alone takes over 100 days online. Only an offline transfer meets 3 weeks.
A, B, D fail: all move data over the same 100 Mbps link.
</details>

### Q5 | case study | 1.1 | pillars: cost, reliability
Refer to Altostrat Media. Altostrat wants lower storage cost for its growing media library while popular titles stay fast. Access to each title drops after release, at different rates. What should you do?

A) Enable Autoclass on the media buckets. B) Lifecycle rule: move everything to Archive after 30 days. C) Move the library to Filestore. D) Change the buckets from region to multi-region.

<details><summary>Answer</summary>
A. Why: Autoclass moves each object to colder classes when unread and back to Standard when read, matching uneven access.
B fails: titles still in demand pay retrieval fees and a 365-day minimum.
C fails: file storage costs more per GB for this use.
D fails: raises cost.
</details>

## Chunk 2: Section 2, managing and provisioning (4)

### Q6 | 2.1 | pillars: reliability, security
A company needs an encrypted link to Google Cloud this week, with a 99.99% availability SLA, and has modest bandwidth needs. What should it configure?

A) Classic VPN with one tunnel. B) HA VPN with tunnels on both gateway interfaces and BGP through Cloud Router. C) Dedicated Interconnect. D) VPC Network Peering.

<details><summary>Answer</summary>
B. Why: HA VPN with both interfaces and dynamic routing carries the 99.99% SLA and sets up in hours.
A fails: Classic VPN carries 99.9%.
C fails: takes longer to provision and isn't encrypted by default.
D fails: connects VPC networks, not on-prem.
</details>

### Q7 | 2.1 | pillars: security
VMs without external IPs must read objects from Cloud Storage. They must not reach the rest of the internet. What do you enable?

A) Cloud NAT for the subnet. B) Private Google Access on the subnet. C) External IPs with a firewall egress rule. D) VPC Network Peering to Google.

<details><summary>Answer</summary>
B. Why: Private Google Access lets internal-only VMs reach Google APIs, and nothing else.
A fails: opens general internet egress.
C fails: adds public exposure.
D fails: not how you reach Google APIs.
</details>

### Q8 | 2.3 | pillars: reliability
A managed instance group keeps recreating VMs that later turn out healthy. The app takes about 4 minutes to start. What do you change?

A) Raise the health check timeout to 60 seconds. B) Set the autohealing initial delay above the startup time. C) Switch to a zonal MIG. D) Turn off autohealing.

<details><summary>Answer</summary>
B. Why: during the initial delay the MIG ignores failed checks, so booting VMs aren't recreated.
A fails: a longer timeout per check doesn't cover 4 minutes of startup.
C fails: unrelated to boot time and reduces availability.
D fails: loses repair of VMs that have failed.
</details>

### Q9 | 2.4 | pillars: operational-excellence
A data science team retrains a model by hand in notebooks. Leadership wants repeatable, auditable retraining with lineage. What should they adopt?

A) A cron job on a VM that runs the notebook. B) Agent Platform Pipelines (Vertex AI Pipelines) defined with Kubeflow Pipelines. C) Cloud Run functions triggered daily. D) BigQuery scheduled queries.

<details><summary>Answer</summary>
B. Why: managed pipelines orchestrate data prep, training, evaluation, and deployment, and record lineage.
A fails: no lineage, fragile.
C fails: not built for multi-step ML training.
D fails: SQL scheduling, not ML orchestration.
</details>

## Chunk 3: Section 3, security and compliance (4)

### Q10 | 3.1 | pillars: security
A GitHub Actions pipeline must deploy to Google Cloud. Security forbids long-lived credentials. What do you set up?

A) A service account key stored as a GitHub secret. B) Workload Identity Federation trusting GitHub's OIDC provider, impersonating a deploy service account. C) A personal access token of an admin user. D) Basic Editor role for the pipeline's user.

<details><summary>Answer</summary>
B. Why: the pipeline exchanges its own short-lived token for Google credentials; no key exists.
A fails: a long-lived key, the thing security forbids.
C fails: ties deploys to a person and a long-lived secret.
D fails: over-privileged and still needs credentials.
</details>

### Q11 | 3.1 | pillars: security
Analysts with valid BigQuery read access must not be able to copy data to projects outside the company. What control addresses this?

A) Remove their BigQuery roles. B) VPC Service Controls perimeter around the data projects. C) CMEK on the datasets. D) Cloud Armor.

<details><summary>Answer</summary>
B. Why: a perimeter blocks data moving to resources outside it, even for principals IAM allows.
A fails: stops their work.
C fails: controls keys, not where data can be copied.
D fails: protects web apps at the edge.
</details>

### Q12 | 3.2 | pillars: security
A regulator asks who read objects in a patient-records bucket last month. Data Access logs were never enabled for Cloud Storage. What's true?

A) Admin Activity logs have the reads. B) The reads weren't recorded; enable Data Access logs now for the future. C) Access Transparency logs have them. D) VPC Flow Logs have them.

<details><summary>Answer</summary>
B. Why: Data Access logs are off by default for Cloud Storage, so past reads weren't logged.
A fails: Admin Activity records configuration changes, not data reads.
C fails: covers Google staff access, not customer users.
D fails: records network flows, not object reads by identity.
</details>

### Q13 | case study | 3.2 | pillars: security
Refer to KnightMotives Automotive. Driver data from EU customers must stay in the EU. What should you apply first?

A) An organization policy restricting resource locations to EU regions, on the folder that holds EU data. B) A deny policy on `storage.objects.get`. C) Cloud CDN with EU edge caching. D) Committed use discounts in EU regions.

<details><summary>Answer</summary>
A. Why: the resource locations constraint stops anyone creating resources outside the EU, whatever their IAM roles.
B fails: blocks reads, not location.
C fails: caching, not residency.
D fails: pricing, not control.
</details>

## Chunk 4: Section 4, technical and business processes (3)

### Q14 | 4.1 | pillars: reliability
A release on GKE causes errors for 5% of users. A fix will take 2 hours. What's the first action?

A) Debug in production until the fix is ready. B) Roll back to the previous release, then debug. C) Scale up the node pool. D) Delete the failing Pods.

<details><summary>Answer</summary>
B. Why: restore service first; the rollback takes minutes.
A fails: users stay broken for hours.
C fails: capacity isn't the cause.
D fails: replacements run the same bad release.
</details>

### Q15 | 4.2 | pillars: operational-excellence
A company moves to microservices. Its ops team has never run containers, and go-live is in 3 months. Which plan fits best?

A) GKE Standard with custom node pools for full control. B) Cloud Run or GKE Autopilot, a training plan for the team, and a staged rollout. C) Delay the move until the team hires Kubernetes experts. D) Self-managed Kubernetes on Compute Engine.

<details><summary>Answer</summary>
B. Why: managed platforms lower the ops load, training builds skills, staging limits risk.
A and D fail: more ops burden than the team can carry.
C fails: ignores the business deadline.
</details>

### Q16 | case study | 4.1 | pillars: operational-excellence
Refer to EHR Healthcare. Alerts go by email and get ignored; outages come from misconfiguration and capacity spikes. What change helps most?

A) Send alerts to more email addresses. B) SLO-based burn-rate alerts routed to on-call tooling, plus infrastructure as code with policy checks. C) Add more dashboards. D) Double all capacity.

<details><summary>Answer</summary>
B. Why: burn-rate alerts cut noise and reach people who act; IaC and policy prevent misconfiguration.
A fails: more ignored email.
C fails: dashboards don't page anyone.
D fails: expensive and doesn't fix misconfiguration.
</details>

## Chunk 5: Section 5, managing implementation (2)

### Q17 | 5.2 | pillars: operational-excellence
Developers need to test code against Pub/Sub and Spanner without touching shared cloud resources or incurring cost. What should they use?

A) A shared dev project. B) The local emulators in the Google Cloud CLI. C) Production with a test flag. D) Mock every client call by hand.

<details><summary>Answer</summary>
B. Why: the CLI ships emulators for Pub/Sub and Spanner (and Bigtable, Firestore).
A fails: tests collide and cost money.
C fails: risks production.
D fails: slow and misses real behavior.
</details>

### Q18 | 5.1 | pillars: security, operational-excellence
A company exposes APIs to 200 partner insurers. It needs keys, quotas, rate limits, and usage analytics per partner. What should it use?

A) Cloud Armor. B) Apigee. C) VPC Service Controls. D) A custom proxy on Compute Engine.

<details><summary>Answer</summary>
B. Why: Apigee is Google Cloud's API management platform: security, quotas, rate limits, analytics, partner onboarding.
A fails: edge security, not API management.
C fails: a data exfiltration perimeter, not API management.
D fails: builds what Apigee provides.
</details>

## Chunk 6: Section 6, solution and operations excellence (2)

### Q19 | 6.2 | pillars: reliability
An SLO is 99.9% of requests succeed over 30 days. The team gets paged for every brief error spike. What alerting should replace it?

A) Alert when any request fails. B) Burn-rate alerts on the error budget: fast burn pages, slow burn opens a ticket. C) Alert on CPU above 80%. D) No alerts; review the SLO weekly.

<details><summary>Answer</summary>
B. Why: alerts track how fast the budget burns, so blips don't page and real incidents do.
A fails: noise.
C fails: a cause metric, not user impact.
D fails: misses fast incidents.
</details>

### Q20 | 6.3 | pillars: reliability
A team wants to release a new version to 10% of users on Cloud Run, watch errors, then expand. What should it use?

A) Blue/green with a DNS switch. B) A canary strategy in Cloud Deploy, or a Cloud Run traffic split by revision. C) A rolling update of VMs. D) A new project per release.

<details><summary>Answer</summary>
B. Why: both shift a percentage of traffic to the new revision and let you expand or roll back.
A fails: all-or-nothing.
C fails: Cloud Run has no VMs to roll.
D fails: heavy and doesn't split traffic.
</details>

## Score table

| Section | Questions | Your correct | Pass mark |
|---|---|---|---|
| 1 Design and planning | Q1 to Q5 | | 4 |
| 2 Managing and provisioning | Q6 to Q9 | | 3 |
| 3 Security and compliance | Q10 to Q13 | | 3 |
| 4 Processes | Q14 to Q16 | | 2 |
| 5 Implementation | Q17, Q18 | | 2 |
| 6 Operations excellence | Q19, Q20 | | 2 |

Any section below its pass mark goes into B22. The pass marks are this course's study targets, not Google's scoring.
