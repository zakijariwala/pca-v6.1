# Curriculum

Production proficiency first, PCA v6.1 second. 32 blocks in 5 phases.

Scope source: [Professional Cloud Architect exam guide v6.1](https://cloud.google.com/learn/certification/guides/professional-cloud-architect) (exams on or after October 30, 2025) and its four case studies, linked from that page. Exam guide refs below use its section numbers:

| Section | Title | Weight |
|---|---|---|
| 1 | Designing and planning a cloud solution architecture | ~25% |
| 2 | Managing and provisioning a solution infrastructure | ~18% |
| 3 | Designing for security and compliance | ~19% |
| 4 | Analyzing and optimizing technical and business processes | ~15% |
| 5 | Managing implementation | ~11% |
| 6 | Ensuring solution and operations excellence | ~12% |

Case study codes: AM = Altostrat Media, CR = Cymbal Retail, EHR = EHR Healthcare, KM = KnightMotives Automotive. Tie-ins name the theme only. Read the case studies on Google's site.

Pillars follow the Well-Architected Framework: security, reliability, cost, performance, operational-excellence, sustainability.

Each block: 1 to 2 sessions of 60 to 90 minutes. P08 takes 4 to 6. Labs are optional; each lab candidate is listed because running it shows something reading can't. Labs need a sandbox project and a budget alert. See `templates/lab.md`.

Lab constraints on a free trial account: the trial has no organization node, so folders, org policies, and Shared VPC stay theory unless you set up Cloud Identity Free with a domain you own. GKE, Cloud SQL, and load balancers burn trial credit fastest. Tear down daily.

## Sequence

| Phase | Blocks |
|---|---|
| 0 Orientation | B00, P01 |
| 1 Foundations | B01, B02, B03, P02, B04, P03, B05, B06, P04 |
| 2 Data, AI, security, operations | B07, B08, B09, B10, B11, P05, B12, P06, B13, P07, B14, B15, P08, B16 |
| 3 Case studies | B17, B18, B19, B20 |
| 4 Exam readiness | B21, B22, B23 |

B blocks teach. P blocks practice: troubleshooting, Terraform, drills, capstone.

---

# Phase 0: Orientation

## B00 Exam map and GCP mental model
- **Exam guide refs:** overview, case studies section, 1.2 (WAF familiarity), 5.2 (Cloud Shell, gcloud)
- **In scope:** PCA format and case-study style; the 6 v6.1 domains; how Google writes questions (best answer vs correct answer); GCP's shape: global network, regions, zones, projects as the unit of everything; console vs gcloud vs Cloud Shell vs APIs. For learners who know AWS or Azure, an equivalents table.
- **Out of scope:** CLI scripting depth (P01), hierarchy and billing (B01), any service deep dive.
- **Exit:** explain why projects matter more on GCP than accounts elsewhere.
- **Pillars:** operational-excellence
- **Case study tie-ins:** all four as format examples: how a case splits into business requirements, technical requirements, and current state.
- **Lab candidates:** activate Cloud Shell, run `gcloud config list`, create and delete a project. Doing shows the project lifecycle and the pending-deletion state.

## P01 CLI and API fluency
- **Exam guide refs:** 5.2
- **In scope:** gcloud configurations for multiple projects; `--format`, `--filter`, projections for scripting; enabling APIs and why calls fail when one is off; ADC vs user credentials vs impersonation; gcloud storage, bq, kubectl basics; alpha/beta commands; Cloud Shell limits (ephemeral VM, 5 GB home); installing gcloud locally.
- **Out of scope:** Terraform (P02), IAM design (B02), GKE operations (P04).
- **Exit:** find any resource and its config without the console.
- **Pillars:** operational-excellence, security
- **Case study tie-ins:** none direct.
- **Lab candidates:** script that lists every VM across two projects as CSV, using only gcloud flags. Doing builds the filter and format muscle memory reading can't.

---

# Phase 1: Foundations

## B01 Resource hierarchy and billing
- **Exam guide refs:** 3.1 (resource hierarchy), 1.1 (cost optimization), 4.2 (cost optimization, CapEx/OpEx)
- **In scope:** org, folders, projects, resources; policy inheritance; billing accounts and linking; labels vs tags; quotas; Cloud Identity and Google Workspace as the identity root; landing zone basics.
- **Out of scope:** IAM roles and org policy constraints (B02), cost optimization techniques (B15).
- **Exit:** design a folder structure for a company with 3 business units and a dev/prod split.
- **Pillars:** security, cost, operational-excellence
- **Case study tie-ins:** EHR (Active Directory as identity source, many customer environments); AM (Google identity mixed with third-party IdPs).
- **Lab candidates:** budget with alerts at 50/90/100% before anything else runs. Doing sets the safety net for every later lab.

## B02 IAM and org policy
- **Exam guide refs:** 3.1 (IAM, separation of duties, security controls: organization policy, secure remote access: service account impersonation, Workload Identity Federation)
- **In scope:** principals, basic vs predefined vs custom roles; allow and deny policies; service accounts, keys vs impersonation; Workload Identity Federation; IAM Conditions; Organization Policy Service constraints; least privilege patterns; groups over users.
- **Out of scope:** audit and troubleshooting tooling depth (P05), IAP and VPC Service Controls (B11), GKE Workload Identity (B06, P04).
- **Exit:** pick the right grant level and explain why service account keys are an exam red flag.
- **Pillars:** security
- **Case study tie-ins:** EHR (external identity source, regulated access); CR (customer data handling); AM (third-party identity providers).
- **Lab candidates:** impersonate a service account, then prove a denied action with Policy Troubleshooter. Doing shows the exact error and trace output.

## B03 VPC networking I
- **Exam guide refs:** 1.3 (cloud-native networking: VPC, peering, firewalls, routing, Shared VPC), 2.1 (VPC design, security protection), 3.1 (hierarchical firewall policy)
- **In scope:** global VPC, regional subnets, auto vs custom mode; firewall rules and firewall policies; routes; Cloud NAT; Private Google Access; VPC peering vs Shared VPC; IP planning.
- **Out of scope:** load balancing, DNS, hybrid links, Private Service Connect (B04); diagnostics (P03).
- **Exit:** choose between Shared VPC and peering for a given org.
- **Pillars:** security, reliability, performance
- **Case study tie-ins:** EHR and CR (multiple environments sharing a network); AM (GKE networking).
- **Lab candidates:** custom VPC, two subnets, firewall rule, VM without external IP reaching the internet through Cloud NAT. Doing shows egress without a public IP.

## P02 Terraform on GCP foundations
- **Exam guide refs:** 5.2 (infrastructure as code), 2.3 (infrastructure orchestration, resource configuration), 4.1 (service catalog and provisioning)
- **In scope:** google vs google-beta providers; remote state in a Cloud Storage bucket with versioning; plan/apply/destroy; variables, outputs, modules; importing existing resources; drift; the Cloud Foundation Toolkit modules; service account for Terraform with impersonation, no keys.
- **Out of scope:** CI/CD pipelines for Terraform (B13), policy as code (B13).
- **Exit:** all later labs run from Terraform.
- **Pillars:** operational-excellence, reliability
- **Case study tie-ins:** EHR (outages from misconfigured systems; provision new environments on demand); AM (simplify infrastructure management).
- **Lab candidates:** rebuild the B03 VPC, subnets, firewall, and NAT in Terraform with remote state. Doing exposes state, drift, and import behavior.

## B04 VPC networking II: load balancing and hybrid
- **Exam guide refs:** 1.3 (integration with on-premises and multicloud, load balancers, Private Service Connect), 2.1 (hybrid networking, multicloud, VPC design and load balancing)
- **In scope:** load balancer family (global vs regional, external vs internal, L4 vs L7, proxy vs passthrough); Cloud DNS; Cloud CDN; Cloud VPN (HA VPN); Dedicated vs Partner Interconnect; Network Connectivity Center; Private Service Connect.
- **Out of scope:** Cloud Armor and IAP (B11), network diagnostics (P03).
- **Exit:** pick the load balancer and hybrid link from a requirements paragraph.
- **Pillars:** reliability, performance, security
- **Case study tie-ins:** EHR (secure, high-performance on-prem link; reduce latency to all customers; legacy insurer interfaces stay on-prem); AM (hybrid connectivity for ingestion, media delivery); CR (on-prem systems during migration); KM (plant-to-headquarters network upgrades, on-prem plus other clouds).
- **Lab candidates:** HA VPN between two VPCs to simulate hybrid. Doing shows BGP session state and tunnel failover.

## P03 Network troubleshooting
- **Exam guide refs:** 4.1 (troubleshooting and root cause analysis), 6.4 (support of deployed solutions)
- **In scope:** Connectivity Tests; VPC Flow Logs; firewall rule logging; Network Intelligence Center; failure patterns: missing firewall rule, missing route, Private Google Access off, Cloud NAT port exhaustion, DNS resolution in hybrid setups, load balancer health check ranges not allowed, MTU on VPN.
- **Out of scope:** network design (B03, B04).
- **Exit:** a repeatable "packet can't get there" checklist.
- **Pillars:** operational-excellence, reliability
- **Case study tie-ins:** EHR (outages from misconfiguration).
- **Lab candidates:** the tutor provides a deliberately broken Terraform network; find and fix 3 faults using only diagnostic tools. Doing builds diagnosis order.

## B05 Compute Engine
- **Exam guide refs:** 1.3 (choosing compute resources: Spot VMs, custom machine types, specialized workloads), 2.3 (compute provisioning, spot vs standard, patch management)
- **In scope:** machine families; instance templates; managed instance groups, autoscaling, autohealing, regional MIGs; Spot VMs; sole-tenant nodes; disks and snapshots; startup scripts; OS Login.
- **Out of scope:** containers and serverless (B06), disk types in depth (B07), snapshot-based DR design (B12).
- **Exit:** know when a VM beats containers or serverless.
- **Pillars:** reliability, cost, performance
- **Case study tie-ins:** EHR (traffic spikes, colocation exit); CR (data-center hosting cost).
- **Lab candidates:** MIG behind an HTTP load balancer, then kill a VM and watch autohealing. Doing shows recreation timing and health check behavior.

## B06 Containers and serverless
- **Exam guide refs:** 1.3 (mapping compute needs to GKE, Cloud Run, Cloud Run functions), 2.3 (container orchestration, serverless computing, serverless networking)
- **In scope:** GKE Standard vs Autopilot; node pools, regional clusters, Workload Identity for GKE; Cloud Run services and jobs; Cloud Run functions; App Engine positioning; the compute decision tree across VMs, GKE, Cloud Run.
- **Out of scope:** GKE day-2 operations (P04), CI/CD (B13).
- **Exit:** walk the compute decision tree for 3 scenarios.
- **Pillars:** operational-excellence, cost, reliability
- **Case study tie-ins:** AM (GKE platform, Cloud Run functions for event-driven media tasks); EHR (containerized apps on several clusters, consistent management); CR (Kubernetes clusters).
- **Lab candidates:** deploy a container to Cloud Run from Cloud Shell. Doing shows revisions, URLs, and scale to zero.

## P04 GKE operations
- **Exam guide refs:** 2.3 (container orchestration), 4.1 (troubleshooting), 6.4 (support of deployed solutions)
- **In scope:** kubectl against GKE; VPC-native clusters, IP exhaustion planning; release channels, upgrades, maintenance windows and exclusions; HPA, VPA, cluster autoscaler, node auto-provisioning; PodDisruptionBudgets; Gateway API and Ingress; Workload Identity; debugging CrashLoopBackOff, Pending pods, OOMKilled; GKE logs and metrics; Autopilot constraints.
- **Out of scope:** compute choice (B06), CI/CD rollouts (B13).
- **Exit:** run first response on a sick GKE workload.
- **Pillars:** reliability, operational-excellence
- **Case study tie-ins:** AM (scalable Kubernetes on-prem and in cloud); EHR (multiple container environments).
- **Lab candidates:** Autopilot cluster, deploy an app with HPA, break it three ways, fix each, delete the cluster. Doing shows real failure states. GKE bills by the hour: tear down the same session.

---

# Phase 2: Data, AI, security, operations

## B07 Storage
- **Exam guide refs:** 1.3 (choosing storage types: object, file), 2.2 (storage allocation, security and access, transfer and latency, retention and life cycle, growth planning, data protection)
- **In scope:** Cloud Storage classes, location types, lifecycle rules, versioning, retention and object lock, signed URLs; Persistent Disk vs Hyperdisk vs Local SSD; Filestore; NetApp Volumes positioning.
- **Out of scope:** databases (B08), CMEK (B11), transfer services (B14).
- **Exit:** pick storage class and location type from an access pattern plus a DR need.
- **Pillars:** cost, reliability, security
- **Case study tie-ins:** AM (media library cost at growing volume, archival); CR (product images).
- **Lab candidates:** lifecycle rule plus signed URL. Doing shows URL expiry and that lifecycle actions run asynchronously.

## B08 Databases
- **Exam guide refs:** 1.3 (choosing storage types: databases), 2.2 (storage allocation, data growth, data protection), 5.2 (Cloud Emulators: Bigtable, Spanner, Firestore)
- **In scope:** Cloud SQL (HA, read replicas), AlloyDB, Spanner, Bigtable, Firestore, Memorystore; consistency and scale trade-offs; the database selection tree.
- **Out of scope:** analytics warehouse (B09), database migration tooling (B14).
- **Exit:** choose a database from a one-line workload description in under 30 seconds.
- **Pillars:** reliability, performance, cost
- **Case study tie-ins:** EHR and CR (MySQL, SQL Server, Redis, MongoDB estates to rehome).
- **Lab candidates:** Cloud SQL with private IP, failover test. Doing shows failover time and connection behavior. Cloud SQL bills by the hour.

## B09 Data and analytics
- **Exam guide refs:** 1.1 (movement of data), 1.3 (choosing data processing solutions), 2.2 (data processing and compute provisioning)
- **In scope:** BigQuery (storage vs compute, partitioning, clustering, pricing models); Pub/Sub; Dataflow; Dataproc; Cloud Composer; Datastream; Dataplex; Looker; batch vs streaming pipeline patterns.
- **Out of scope:** BigQuery ML and Vertex AI (B10), billing export analysis (B15).
- **Exit:** sketch an ingestion-to-dashboard pipeline for streaming IoT data.
- **Pillars:** performance, cost, operational-excellence
- **Case study tie-ins:** AM (BigQuery warehouse, content trend analysis); EHR (healthcare trend insights, ingest from new providers); CR (SFTP and batch ETL to replace, data silos); KM (siloed corporate data, data monetization platform).
- **Lab candidates:** Pub/Sub to BigQuery subscription, partitioned table, query cost check. Doing shows bytes scanned with and without partition pruning.

## B10 AI/ML and securing AI
- **Exam guide refs:** 1.3 (ML/AI solutions: Gemini, Agent Builder, Model Garden, AI Hypercomputer), 2.4 (Vertex AI Pipelines, data integration, AI Hypercomputer, GPUs and TPUs), 2.5 (Google AI APIs, Gemini Enterprise, Model Garden), 3.1 (securing AI: Model Armor, Sensitive Data Protection, secure model deployment)
- **In scope:** Vertex AI platform (training, endpoints, pipelines, Model Garden); Gemini on Vertex; pre-trained APIs; BigQuery ML; build vs buy vs fine-tune decisions; Model Armor; Sensitive Data Protection; responsible AI basics.
- **Out of scope:** general data pipelines (B09), non-AI security controls (B11).
- **Exit:** pick the least-effort AI option that meets a given requirement.
- **Pillars:** security, cost, performance
- **Case study tie-ins:** AM (summarization, metadata extraction, harmful content detection, explainable and auditable AI, chatbots); CR (attribute and image generation, conversational commerce, product discovery, human-in-the-loop review); EHR (predictions on provider data); KM (autonomous driving ML and simulation, replacing obsolete AI infrastructure).
- **Lab candidates:** none in the contract.

## B11 Security and compliance
- **Exam guide refs:** 3.1 (data security, security controls, Cloud KMS and CMEK, secure remote access, software supply chain), 3.2 (legislation, commercial data, certifications, audits)
- **In scope:** Cloud KMS, CMEK, CSEK, Cloud HSM, EKM; Secret Manager; VPC Service Controls; Identity-Aware Proxy; Cloud Armor; Security Command Center; Binary Authorization; Access Transparency; Assured Workloads; compliance mapping (HIPAA, PCI, GDPR).
- **Out of scope:** IAM fundamentals (B02), audit log investigation (P05), AI-specific controls (B10).
- **Exit:** harden a regulated workload and name the control for each threat.
- **Pillars:** security
- **Case study tie-ins:** EHR (health record regulation); CR (customer data, payment data); AM (content safety); KM (past breaches, EU data protection, incident response plan).
- **Lab candidates:** CMEK on a bucket, then disable the key and watch reads fail. Doing shows the failure mode and recovery.

## P05 IAM troubleshooting and audit
- **Exam guide refs:** 3.1 (auditing, separation of duties), 3.2 (audits, including logs), 4.1 (troubleshooting)
- **In scope:** Policy Troubleshooter; Policy Analyzer ("who can do X on Y"); IAM Recommender and unused permissions; Cloud Audit Logs types (Admin Activity, Data Access, System Event, Policy Denied) and which are on by default; answering "who deleted this bucket"; service account sprawl cleanup; org policy violations.
- **Out of scope:** IAM design (B02).
- **Exit:** answer any "why can't they" or "who did it" ticket.
- **Pillars:** security, operational-excellence
- **Case study tie-ins:** EHR (regulatory audits).
- **Lab candidates:** grant, deny, and trace access for a test service account; query audit logs for the change. Doing shows log latency and field layout.

## B12 Reliability, observability, DR
- **Exam guide refs:** 1.1 (business continuity plan, observability), 1.2 (high availability and failover, backup and recovery), 2.2 (data protection), 4.1 (disaster recovery), 4.2 (business continuity), 6.2 (monitoring and logging)
- **In scope:** Cloud Monitoring, Logging, log sinks and routing, Trace; SLIs, SLOs, error budgets; HA patterns across zones and regions; DR tiers (backup-restore, pilot light, warm, hot) with RTO/RPO; Backup and DR Service.
- **Out of scope:** hands-on alerting and dashboards (P06).
- **Exit:** map an RTO/RPO pair to a DR pattern and its cost.
- **Pillars:** reliability, cost
- **Case study tie-ins:** EHR (99.9% availability target, DR plan rework); AM (reliability as a top priority).
- **Lab candidates:** snapshot schedule plus restore to a new VM. Doing shows restore time.

## P06 Observability in practice
- **Exam guide refs:** 6.2 (monitoring and logging, alerting strategies), 1.1 (observability), 6.1 (operational excellence pillar)
- **In scope:** Logging query language; log-based metrics; log sinks to BigQuery and Cloud Storage; alerting policies and notification channels; uptime checks; dashboards; Error Reporting; Cloud Trace; Managed Service for Prometheus; alert fatigue and SLO-based alerting.
- **Out of scope:** SLO theory and DR (B12).
- **Exit:** every capstone component ships with an alert.
- **Pillars:** operational-excellence, reliability
- **Case study tie-ins:** EHR (email alerts ignored, consistent logging and retention); AM (Prometheus alongside Cloud Monitoring, email alerts); CR (Grafana, Nagios, Elastic to consolidate).
- **Lab candidates:** SLO plus burn-rate alert on a Cloud Run service; trigger it on purpose. Doing shows alert timing and noise.

## B13 DevOps and IaC
- **Exam guide refs:** 3.1 (securing software supply chain), 4.1 (SDLC, CI/CD, testing and validation), 5.1 (application and infrastructure deployment, testing frameworks), 5.2 (IaC), 6.3 (deployment and release management)
- **In scope:** Cloud Build, Artifact Registry, Cloud Deploy; deployment strategies (rolling, blue/green, canary); Terraform on GCP (state, modules, remote backend); Config Connector positioning; policy as code.
- **Out of scope:** Terraform basics (P02), GKE operations (P04).
- **Exit:** design a CI/CD path with a safe rollout for a GKE app.
- **Pillars:** operational-excellence, reliability, security
- **Case study tie-ins:** AM (centralized CI/CD for containers); EHR (fast continuous deployment).
- **Lab candidates:** Cloud Build pipeline that runs `terraform plan` on commit. Doing shows build service account permissions.

## P07 Incident drills
- **Exam guide refs:** 4.1 (troubleshooting and root cause analysis), 4.3 and 6.6 (reliability in production), 6.4 (support)
- **In scope:** the tutor plays the system: gives a symptom and alert text; the learner responds with commands and reasoning; the tutor returns plausible output. 5 incidents across networking, IAM, GKE, Cloud SQL, quota/billing. After each, a 5-line postmortem (impact, root cause, detection, fix, prevention).
- **Out of scope:** real resources.
- **Exit:** scored on time to diagnosis and whether the obvious got checked first.
- **Pillars:** operational-excellence, reliability
- **Case study tie-ins:** EHR (outages from misconfiguration and capacity).
- **Lab candidates:** none. The drill is simulated.

## B14 Migration
- **Exam guide refs:** 1.4 (integrating with existing systems, Migration Center, methodologies, network and dependency planning, license and financial impact), 1.1 (workload disposition), 5.1 (migration tooling)
- **In scope:** the Rs (rehost, replatform, refactor, and others); Migration Center; Migrate to Virtual Machines; Database Migration Service; Storage Transfer Service; Transfer Appliance; bandwidth math for choosing transfer methods; migration wave planning.
- **Out of scope:** hybrid link design (B04).
- **Exit:** plan a migration for 500 VMs plus 200 TB with a fixed deadline.
- **Pillars:** cost, reliability, operational-excellence
- **Case study tie-ins:** EHR (colocation lease deadline); CR (on-prem databases, file integrations); AM (legacy ingestion and archival systems); KM (mainframe supply chain and legacy ERP, gradual hybrid modernization).
- **Lab candidates:** Storage Transfer job between buckets. Doing shows job setup and service agent permissions.

## B15 Cost optimization
- **Exam guide refs:** 1.1 (cost optimization), 2.3 (spot vs standard), 4.2 (cost and resource optimization, CapEx/OpEx)
- **In scope:** pricing models; sustained use and committed use discounts; Spot; rightsizing recommendations; budgets and alerts; billing export to BigQuery; FinOps practices; cost trade-offs inside other design choices.
- **Out of scope:** billing account structure (B01).
- **Exit:** cut a given architecture's cost 30% without breaking its SLO.
- **Pillars:** cost, sustainability
- **Case study tie-ins:** AM (storage cost at scale); CR (call center and data-center cost); EHR (administration cost); KM (data monetization to fund new investment).
- **Lab candidates:** none in the contract.

## P08 Capstone
- **Exam guide refs:** 5.1, 5.2, 6.3; integrates sections 1 to 3.
- **In scope:** build in Terraform, in one project, from a clean state, in 5 milestones:
  - M1 Network: custom VPC, private subnets, NAT, firewall policies
  - M2 Compute: containerized app on Cloud Run (or GKE if P04 went well), Artifact Registry
  - M3 Data: Cloud SQL private IP, Secret Manager, no service account keys anywhere
  - M4 Edge and security: global external Application Load Balancer, Cloud Armor, IAP for admin paths
  - M5 Ops: Cloud Build pipeline with canary through Cloud Deploy, SLO alerts, budget alert, full teardown

  Each milestone gets a senior-engineer-style code review of the Terraform.
- **Out of scope:** new services not taught in B01 to B15.
- **Exit:** explain every resource and its failure mode.
- **Pillars:** security, reliability, cost, performance, operational-excellence
- **Case study tie-ins:** none direct; the build mirrors a typical case target architecture.
- **Lab candidates:** the whole block is a lab. Load balancer and Cloud SQL bill by the hour.

## B16 Well-Architected Framework and synthesis
- **Exam guide refs:** 1.1 (business requirements, trade-offs, KPIs and ROI), 1.2 (WAF), 1.5 (future improvements), 4.2 (business processes), 6.1 (operational excellence pillar)
- **In scope:** the six WAF pillars and their key principles; business vs technical requirements; stakeholder and process questions from section 4; a cross-service decision framework recap drawn from B01 to B15.
- **Out of scope:** new services.
- **Exit:** critique a sample architecture pillar by pillar.
- **Pillars:** security, reliability, cost, performance, operational-excellence, sustainability
- **Case study tie-ins:** all four as critique targets; KM for section 4.2 themes (upskilling, business and technical team communication).
- **Lab candidates:** none.

---

# Phase 3: Case studies

## B17 Case study method and Altostrat Media
- **Exam guide refs:** case studies; sections 1 to 6 as applied
- **In scope:** a repeatable method for reading any case (business requirements, technical requirements, constraints, current state, keyword extraction); then Altostrat Media end to end; likely question angles; target architecture.
- **Out of scope:** the other three cases (B18 to B20).
- **Exit:** 10 case-style questions on Altostrat Media.
- **Pillars:** reliability, cost, security
- **Case study tie-ins:** AM themes: hybrid Kubernetes, media storage cost, gen AI for summaries and moderation, AI auditability, observability consolidation.
- **Lab candidates:** none.

## B18 Cymbal Retail
- **Exam guide refs:** case studies; sections 1 to 6 as applied
- **In scope:** apply the B17 method to Cymbal Retail; target architecture; likely question angles.
- **Out of scope:** other cases.
- **Exit:** 10 case-style questions.
- **Pillars:** performance, cost, security
- **Case study tie-ins:** CR themes: gen AI catalog enrichment, conversational commerce and product discovery, human-in-the-loop, database and integration modernization, call center cost.
- **Lab candidates:** none.

## B19 EHR Healthcare
- **Exam guide refs:** case studies; 3.2 in depth; sections 1 to 6 as applied
- **In scope:** apply the B17 method to EHR Healthcare; compliance-heavy design choices; target architecture.
- **Out of scope:** other cases.
- **Exit:** 10 case-style questions.
- **Pillars:** security, reliability, operational-excellence
- **Case study tie-ins:** EHR themes: colocation exit, availability target, hybrid link to legacy integrations, regulated data, observability overhaul.
- **Lab candidates:** none.

## B20 KnightMotives Automotive
- **Exam guide refs:** case studies; sections 1 to 6 as applied
- **In scope:** apply the B17 method to KnightMotives; data, AI, and edge angles; target architecture.
- **Out of scope:** other cases.
- **Exit:** 10 case-style questions.
- **Pillars:** security, reliability, cost, performance
- **Case study tie-ins:** KM themes: hybrid modernization off a mainframe and legacy ERP, unreliable build-to-order ordering and dealer tooling, siloed data and monetization, autonomous driving ML, rural and plant connectivity, EU data protection after past breaches, workforce upskilling.
- **Lab candidates:** none.

---

# Phase 4: Exam readiness

## B21 Mixed practice set (repeatable)
- **Exam guide refs:** sections 1 to 6, weighted by exam percentage
- **In scope:** 20 exam-style questions across all domains, weighted toward the open weak spots. One at a time, full explanations after each. Ends with a domain-by-domain score and block report.
- **Out of scope:** new teaching.
- **Exit:** domain scores recorded.
- **Pillars:** all
- **Case study tie-ins:** mixed.
- **Lab candidates:** none.

## B22 Remediation (template)
- **Exam guide refs:** per the weak spots chosen
- **In scope:** re-teach only the listed weak-spot topics, starting from the misconception, then test with 8 questions.
- **Out of scope:** topics not on the list.
- **Exit:** 8 questions passed on the listed topics.
- **Pillars:** per topic
- **Case study tie-ins:** per topic.
- **Lab candidates:** none.

## B23 Final review
- **Exam guide refs:** sections 1 to 6
- **In scope:** a one-screen decision cheat sheet per domain, the top 30 exam traps, and a 48-hour pre-exam plan. Then 15 rapid-fire questions.
- **Out of scope:** new teaching.
- **Exit:** 15 rapid-fire questions done.
- **Pillars:** all
- **Case study tie-ins:** all four.
- **Lab candidates:** none.

---

# Gaps against exam guide v6.1

Exam guide items no block contract names. No block was added; each gap lists the block best placed to absorb it if the contract changes.

| Exam guide item | Section | Nearest block |
|---|---|---|
| Gemini Cloud Assist | 1.2, 5.1 | B16 or B13 |
| API management (Apigee) | 5.1 | B13 |
| Cloud Emulators, Cloud Code, Cloud Shell Editor, API client libraries, API access best practices | 5.2 | P01 |
| Google Cloud VMware Engine | 2.3 | B14 |
| Multicloud and on-prem Kubernetes (GKE Enterprise, fleet management) | 1.3, 2.1 | B06 or P04 |
| Cross-cloud networking beyond NCC | 2.1 | B04 |
| AI Hypercomputer, GPUs and TPUs, consumption models | 1.3, 2.4 | B10 |
| Agent Builder, Gemini Enterprise (AI agents, NotebookLM), named Google AI APIs | 1.3, 2.5 | B10 |
| Chrome Enterprise Premium and context-aware access | 3.1 | B11 |
| Separation of duties as an explicit design topic | 3.1 | B02 |
| Data sovereignty, children's privacy, SOC 2 | 3.2 | B11 |
| Chaos engineering, penetration testing, load testing | 4.3, 6.6 | P07 or B12 |
| Profiling and benchmarking (Cloud Profiler) | 6.2 | P06 |
| Quality control measures | 6.5 | B13 |
| Stakeholder, change, and team-readiness management; customer success | 4.2 | B16 (partial today) |
| Software license implications (BYOL, sole-tenant licensing) | 1.4 | B14 (partial via B05 sole-tenant) |
| Sustainability pillar in chunk tagging | WAF | templates/learn.md lists 5 pillars, not 6 |
