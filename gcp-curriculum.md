# GCP Curriculum

Production proficiency first, PCA v6.1 second. 32 blocks.

Source of truth for scope: the PCA v6.1 exam guide and the four case study PDFs.

- Exam guide: https://services.google.com/fh/files/misc/v6.1_pca_professional_cloud_architect_exam_guide_english.pdf
- Cymbal Retail: https://services.google.com/fh/files/misc/v6.1_pca_cymbal_retail_case_study_english.pdf
- Altostrat Media: https://services.google.com/fh/files/misc/altostrat_media_case_study_english.pdf
- EHR Healthcare: https://services.google.com/fh/files/blogs/master_case_study_ehr_healthcare.pdf
- KnightMotives Automotive: from https://cloud.google.com/learn/certification/guides/professional-cloud-architect

## 1. Setup

1. Upload the exam guide and the four case study PDFs to project knowledge.
2. Paste section 2 into the project instructions.
3. Keep one chat named "PCA Orchestrator" for planning.
4. Start each block in a new chat. Name it like "B03 VPC I".
5. Paste each BLOCK REPORT back into the orchestrator.

## 2. Project instructions

```
ROLE
You are my GCP tutor. Goal 1: production proficiency, so I can join a GCP team and ramp fast. Goal 2: pass PCA v6.1 (Oct 2025). When they conflict, teach for production and flag the exam angle separately. I am a GCP beginner with middling sysadmin experience, working Linux CLI skills, and solid software engineering basics.

SOURCE OF TRUTH
- Exam guide and case study PDFs in project knowledge define PCA scope. They do not cap the production scope.
- GCP renames and retires products often. Web search official docs before stating limits, quotas, pricing, flags, or product names. Flag anything unverified.
- The chat titled "PCA Orchestrator" holds the curriculum. Block prompts are the scope contract.

SESSION FLOW (every block chat)
1. Ask 2 or 3 diagnostic questions. Skip what I know.
2. Teach one chunk per message, then stop. Each chunk covers:
   - Problem it solves
   - Mental model on GCP
   - Production lens: default settings that bite, how it fails, what I check first when it breaks, what it costs
   - Exam signals: keywords that point to it
   - Trap: the wrong answer that looks right
   - Doc: the one official page worth bookmarking
3. End each chunk with one check question. Correct me bluntly.
4. Labs are default, not optional. Use gcloud (and Terraform once P02 is done) in Cloud Shell. Mark [LAB]. Every lab ends with a teardown command and a line on what it cost.
5. After scope, run 8 to 10 scenario questions, one at a time. Explain why each wrong option fails. Include at least 2 troubleshooting questions ("X is broken, what do you check first").
6. Close with a BLOCK REPORT in a code block:
   Block ID | Covered | Score x/10 | Labs done | Weak spots | Parking lot | Next

RULES
- Terse, mobile-friendly. Tables for comparisons. No artifacts, no filler, no motivational lines, no em dashes.
- Stay in block scope. Log drift as one line under "Parking lot" and return.
- Compare services by trade-off (cost, ops burden, scale, latency, consistency).
- Tie each service to the Well-Architected Framework pillar it serves.
- Use Linux and on-prem sysadmin analogies where they fit.
- Prefer how real teams run it (IaC, least privilege, private networking) over console clicks. Show the console path only when no CLI path exists.
- "deeper" = one level down. "exam only" = decision rule and move on. "prod only" = skip exam framing.
- Cost safety: never leave a lab resource running. Remind me if a lab uses anything billed by the hour.
```

## 3. Sequence

32 blocks. 10 to 14 weeks at 5 sessions a week. One block is 1 to 2 sessions of 60 to 90 minutes. P08 takes 4 to 6.

Phase 0: Orientation
- B00 Exam map and GCP mental model
- P01 CLI and API fluency

Phase 1: Foundations
- B01 Resource hierarchy and billing
- B02 IAM and org policy
- B03 VPC networking I
- P02 Terraform on GCP foundations
- B04 Load balancing and hybrid
- P03 Network troubleshooting
- B05 Compute Engine
- B06 Containers and serverless
- P04 GKE operations

Phase 2: Data, AI, security, operations
- B07 Storage
- B08 Databases
- B09 Data and analytics
- B10 AI/ML and securing AI
- B11 Security and compliance
- P05 IAM troubleshooting and audit
- B12 Reliability, observability, DR
- P06 Observability in practice
- B13 DevOps and IaC
- P07 Incident drills
- B14 Migration
- B15 Cost optimization
- P08 Capstone (4 to 6 sessions)
- B16 Well-Architected Framework and synthesis

Phase 3: Case studies
- B17 Case method and Altostrat Media
- B18 Cymbal Retail
- B19 EHR Healthcare
- B20 KnightMotives Automotive

Phase 4: Exam readiness
- B21 Mixed practice set (repeatable)
- B22 Remediation (template)
- B23 Final review

## 4. Constraints

- Free trial gives $300 for 90 days. GKE, Cloud SQL, and load balancers burn it fastest. Tear down daily.
- Free trial has no organization node. Folders, org policies, and Shared VPC stay theory unless you set up Cloud Identity Free with a domain you own.
- Off platform only for: the GCP console for labs, and the free official practice exam after two B21 rounds.
- P07 simulates incidents. P08 gives you a real system to break.

## 5. Block prompts

### Phase 0: Orientation

```
B00 - Exam map and GCP mental model
Scope: PCA format and case-study style; the 6 v6.1 domains; how Google writes questions (best answer vs correct answer); GCP's shape: global network, regions, zones, projects as the unit of everything; console vs gcloud vs Cloud Shell vs APIs. If I know AWS or Azure, add an equivalents table.
Lab: activate Cloud Shell, run gcloud config list, create and delete a project.
Exit: I can explain why projects matter more on GCP than accounts elsewhere.
```

```
P01 - CLI and API fluency
Scope: gcloud configurations for multiple projects; --format, --filter, projections for scripting; enabling APIs and why calls fail when one is off; ADC vs user credentials vs impersonation; gcloud storage, bq, kubectl basics; alpha/beta commands; Cloud Shell limits (ephemeral VM, 5 GB home); installing gcloud locally.
Lab: script that lists every VM across two projects as CSV, using only gcloud flags.
Exit: I can find any resource and its config without the console.
```

### Phase 1: Foundations

```
B01 - Resource hierarchy and billing
Scope: org, folders, projects, resources; policy inheritance; billing accounts and linking; labels vs tags; quotas; Cloud Identity and Google Workspace as the identity root; landing zone basics.
Lab: budget with alerts at 50/90/100% before anything else runs.
Exit: design a folder structure for a company with 3 business units and dev/prod split.
```

```
B02 - IAM and org policy
Scope: principals, basic vs predefined vs custom roles; allow and deny policies; service accounts, keys vs impersonation; Workload Identity Federation; IAM Conditions; Organization Policy Service constraints; least privilege patterns; groups over users.
Lab: impersonate an SA, then prove a denied action with Policy Troubleshooter.
Exit: I can pick the right grant level and explain why SA keys are an exam red flag.
```

```
B03 - VPC networking I
Scope: global VPC, regional subnets, auto vs custom mode; firewall rules and firewall policies; routes; Cloud NAT; Private Google Access; VPC peering vs Shared VPC; IP planning.
Lab: custom VPC, two subnets, firewall rule, VM without external IP reaching the internet via NAT.
Exit: I can choose between Shared VPC and peering for a given org.
```

```
P02 - Terraform on GCP foundations
Scope: google vs google-beta providers; remote state in a GCS bucket with versioning; plan/apply/destroy; variables, outputs, modules; importing existing resources; drift; the Cloud Foundation Toolkit modules; service account for Terraform with impersonation, no keys.
Lab: rebuild the B03 VPC, subnets, firewall, and NAT in Terraform with remote state.
Exit: all later labs run from Terraform.
```

```
B04 - VPC networking II: load balancing and hybrid
Scope: load balancer family (global vs regional, external vs internal, L4 vs L7, proxy vs passthrough); Cloud DNS; Cloud CDN; Cloud VPN (HA VPN); Dedicated vs Partner Interconnect; Network Connectivity Center; Private Service Connect.
Lab: HA VPN between two VPCs to simulate hybrid.
Exit: pick the load balancer and hybrid link from a requirements paragraph.
```

```
P03 - Network troubleshooting
Scope: Connectivity Tests; VPC Flow Logs; firewall rule logging; Network Intelligence Center; failure patterns: missing firewall rule, missing route, Private Google Access off, Cloud NAT port exhaustion, DNS resolution in hybrid setups, LB health check ranges not allowed, MTU on VPN.
Lab: tutor gives me a deliberately broken Terraform network; I find and fix 3 faults using only diagnostic tools.
Exit: a repeatable "packet can't get there" checklist.
```

```
B05 - Compute Engine
Scope: machine families; instance templates; managed instance groups, autoscaling, autohealing, regional MIGs; Spot VMs; sole-tenant nodes; disks and snapshots; startup scripts; OS Login.
Lab: MIG behind an HTTP load balancer, then kill a VM and watch autohealing.
Exit: I know when a VM beats containers or serverless.
```

```
B06 - Containers and serverless
Scope: GKE Standard vs Autopilot; node pools, regional clusters, Workload Identity for GKE; Cloud Run services and jobs; Cloud Run functions; App Engine positioning; the compute decision tree across VMs, GKE, Cloud Run.
Lab: deploy a container to Cloud Run from Cloud Shell.
Exit: walk the compute decision tree for 3 scenarios.
```

```
P04 - GKE operations
Scope: kubectl against GKE; VPC-native clusters, IP exhaustion planning; release channels, upgrades, maintenance windows and exclusions; HPA, VPA, cluster autoscaler, node auto-provisioning; PodDisruptionBudgets; Gateway API and Ingress; Workload Identity; debugging CrashLoopBackOff, Pending pods, OOMKilled; GKE logs and metrics; Autopilot constraints.
Lab: Autopilot cluster, deploy an app with HPA, break it three ways, fix each, delete the cluster.
Exit: I can run first-response on a sick GKE workload.
```

### Phase 2: Data, AI, security, operations

```
B07 - Storage
Scope: Cloud Storage classes, location types, lifecycle rules, versioning, retention and object lock, signed URLs; Persistent Disk vs Hyperdisk vs Local SSD; Filestore; NetApp Volumes positioning.
Lab: lifecycle rule plus signed URL.
Exit: pick storage class and location type from access pattern plus DR need.
```

```
B08 - Databases
Scope: Cloud SQL (HA, read replicas), AlloyDB, Spanner, Bigtable, Firestore, Memorystore; consistency and scale trade-offs; the database selection tree.
Lab: Cloud SQL with private IP, failover test.
Exit: choose a database from a one-line workload description in under 30 seconds.
```

```
B09 - Data and analytics
Scope: BigQuery (storage vs compute, partitioning, clustering, pricing models); Pub/Sub; Dataflow; Dataproc; Cloud Composer; Datastream; Dataplex; Looker; batch vs streaming pipeline patterns.
Lab: Pub/Sub to BigQuery subscription, partitioned table, query cost check.
Exit: sketch an ingestion-to-dashboard pipeline for streaming IoT data.
```

```
B10 - AI/ML and securing AI
Scope: Vertex AI platform (training, endpoints, pipelines, Model Garden); Gemini on Vertex; pre-trained APIs; BigQuery ML; build vs buy vs fine-tune decisions; Model Armor; Sensitive Data Protection; responsible AI basics.
Exit: pick the least-effort AI option that meets a given requirement.
```

```
B11 - Security and compliance
Scope: Cloud KMS, CMEK, CSEK, Cloud HSM, EKM; Secret Manager; VPC Service Controls; Identity-Aware Proxy; Cloud Armor; Security Command Center; Binary Authorization; Access Transparency; Assured Workloads; compliance mapping (HIPAA, PCI, GDPR).
Lab: CMEK on a bucket, then disable the key and watch reads fail.
Exit: harden a regulated workload and name the control for each threat.
```

```
P05 - IAM troubleshooting and audit
Scope: Policy Troubleshooter; Policy Analyzer ("who can do X on Y"); IAM Recommender and unused permissions; Cloud Audit Logs types (Admin Activity, Data Access, System Event, Policy Denied) and which are on by default; answering "who deleted this bucket"; service account sprawl cleanup; org policy violations.
Lab: grant, deny, and trace access for a test SA; query audit logs for the change.
Exit: I can answer any "why can't they / who did" ticket.
```

```
B12 - Reliability, observability, DR
Scope: Cloud Monitoring, Logging, log sinks and routing, Trace; SLIs, SLOs, error budgets; HA patterns across zones and regions; DR tiers (backup-restore, pilot light, warm, hot) with RTO/RPO; Backup and DR Service.
Lab: snapshot schedule plus restore to a new VM.
Exit: map an RTO/RPO pair to a DR pattern and its cost.
```

```
P06 - Observability in practice
Scope: Logging query language; log-based metrics; log sinks to BigQuery and GCS; alerting policies and notification channels; uptime checks; dashboards; Error Reporting; Cloud Trace; Managed Service for Prometheus; alert fatigue and SLO-based alerting.
Lab: SLO plus burn-rate alert on a Cloud Run service; trigger it on purpose.
Exit: every capstone component ships with an alert.
```

```
B13 - DevOps and IaC
Scope: Cloud Build, Artifact Registry, Cloud Deploy; deployment strategies (rolling, blue/green, canary); Terraform on GCP (state, modules, remote backend); Config Connector positioning; policy as code.
Lab: Cloud Build pipeline that runs terraform plan on commit.
Exit: design a CI/CD path with a safe rollout for a GKE app.
```

```
P07 - Incident drills
Format: tutor plays the system. Tutor gives a symptom and alert text; I respond with commands and reasoning; tutor returns plausible output. 5 incidents across networking, IAM, GKE, Cloud SQL, quota/billing. After each, I write a 5-line postmortem (impact, root cause, detection, fix, prevention).
Exit: scored on time-to-diagnosis and whether I checked the obvious first.
```

```
B14 - Migration
Scope: the Rs (rehost, replatform, refactor, etc.); Migration Center; Migrate to Virtual Machines; Database Migration Service; Storage Transfer Service; Transfer Appliance; bandwidth math for choosing transfer methods; migration wave planning.
Lab: Storage Transfer job between buckets.
Exit: plan a migration for 500 VMs plus 200 TB with a fixed deadline.
```

```
B15 - Cost optimization
Scope: pricing models; sustained use and committed use discounts; Spot; rightsizing recommendations; budgets and alerts; billing export to BigQuery; FinOps practices; cost trade-offs inside other design choices.
Exit: cut a given architecture's cost 30% without breaking its SLO.
```

```
P08 - Capstone (multi-session, milestones M1 to M5)
Build in Terraform, in one project, from a clean state:
M1 Network: custom VPC, private subnets, NAT, firewall policies
M2 Compute: containerized app on Cloud Run (or GKE if P04 went well), Artifact Registry
M3 Data: Cloud SQL private IP, Secret Manager, no SA keys anywhere
M4 Edge and security: global external ALB, Cloud Armor, IAP for admin paths
M5 Ops: Cloud Build pipeline with canary via Cloud Deploy, SLO alerts, budget alert, full teardown script
Each milestone: tutor reviews my Terraform like a senior engineer in code review. Log gaps to the orchestrator.
Exit: I can explain every resource and its failure mode.
```

```
B16 - Well-Architected Framework and synthesis
Scope: the six WAF pillars and their key principles; business vs technical requirements; stakeholder and process questions from domain 4; a cross-service decision framework recap drawn from B01 to B15.
Exit: critique a sample architecture pillar by pillar.
```

### Phase 3: Case studies

```
B17 - Case study method + Altostrat Media
Scope: a repeatable method for reading any case (business reqs, technical reqs, constraints, current state, keyword extraction); then Altostrat Media end to end from the project knowledge PDF; likely question angles; target architecture.
Exit: 10 case-style questions on Altostrat.
```

```
B18 - Cymbal Retail
Scope: apply the B17 method to Cymbal Retail from the project knowledge PDF; target architecture; likely question angles.
Exit: 10 case-style questions.
```

```
B19 - EHR Healthcare
Scope: apply the B17 method to EHR Healthcare; compliance-heavy design choices; target architecture.
Exit: 10 case-style questions.
```

```
B20 - KnightMotives Automotive
Scope: apply the B17 method to KnightMotives; data, AI, and edge angles; target architecture.
Exit: 10 case-style questions.
```

### Phase 4: Exam readiness

```
B21 - Mixed practice set (repeatable)
Run 20 exam-style questions across all domains, weighted toward the weak spots I paste below. One at a time, full explanations after each. End with a domain-by-domain score and BLOCK REPORT.
Weak spots: [paste from orchestrator]
```

```
B22 - Remediation (template)
Re-teach only these topics, starting from the misconception, then test with 8 questions: [paste list from orchestrator]
```

```
B23 - Final review
Give me a one-screen decision cheat sheet per domain, the top 30 exam traps, and a 48-hour pre-exam plan. Then 15 rapid-fire questions.
```
