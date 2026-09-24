---
block: B20
title: KnightMotives Automotive
pillars: [security, reliability, cost, performance]
exam_guide_refs: ["case-studies", "1.3", "1.4", "2.1", "2.4", "3.2", "4.2"]
last_verified: 2026-09-24
sources:
  - https://cloud.google.com/learn/certification/guides/professional-cloud-architect
  - https://docs.cloud.google.com/distributed-cloud/docs
  - https://docs.cloud.google.com/kubernetes-engine/docs/fleets-overview
  - https://docs.cloud.google.com/gemini-enterprise-agent-platform/overview
  - https://docs.cloud.google.com/ai-hypercomputer/docs/overview
  - https://docs.cloud.google.com/bigquery/docs/editions-intro
  - https://docs.cloud.google.com/assured-workloads/docs/overview
  - https://docs.cloud.google.com/security-command-center/docs/security-command-center-overview
  - https://docs.cloud.google.com/apigee/docs/api-platform/get-started/what-apigee
  - https://docs.cloud.google.com/architecture/migration-to-gcp-getting-started
  - https://docs.cloud.google.com/network-connectivity/docs/interconnect/concepts/cci-overview
  - https://docs.cloud.google.com/kubernetes-engine/multi-cloud/docs/attached
  - https://docs.cloud.google.com/network-connectivity/docs/network-connectivity-center/concepts/overview
unverified: []
---
# B20 KnightMotives Automotive

Read the KnightMotives Automotive case study on Google's exam guide page before this block. This file paraphrases its themes and never quotes it.

## Chunk 1: KnightMotives at a glance
Problem it solves: Apply the B17 method to KnightMotives.

Mental model (paraphrased):
- Business: a car maker (electric, hybrid, combustion) investing in autonomous driving. Wants one AI-powered experience across all models within five years, a better build-to-order process for dealers and buyers, new revenue from its data, stronger security after past breaches, EU data protection compliance, and upskilled staff with better business-IT communication.
- Current state: on-prem for the most part, with some apps in other clouds. Supply chain on an old mainframe; outdated ERP. Separate code bases per vehicle line and heavy technical debt. Dealers lack budget for new hardware. Weak connectivity to plants and to vehicles in rural areas.
- Technical: consistent in-vehicle UX with AI across models; reliable connectivity for real-time features; network upgrades between plants and headquarters; hybrid modernization over time; AI and simulation for autonomous driving; a secure data platform for monetization; a security framework with incident response; modern dealer tools and a CRM.

Exam signals: "KnightMotives" + "EU" → data residency and privacy. "KnightMotives" + "autonomous" → large-scale training and simulation.

Trap: A big-bang mainframe replacement. The case asks for gradual modernization.

Pillar tie-in: Security first, given past breaches.

Check: What does the case say about the pace of legacy modernization?
<details><summary>Answer</summary>Gradual: a hybrid strategy that modernizes or replaces legacy systems over time.</details>

## Chunk 2: Requirements mapped to services
Problem it solves: Turn each requirement into the expected service.

Mental model:

| Requirement theme | Fits | Block |
|---|---|---|
| Hybrid and multicloud operations | GKE fleets across Google Cloud, on-prem (Google Distributed Cloud), and attached clusters in other clouds | B06, P04 |
| Plant and edge compute with weak links | Google Distributed Cloud at plants and edge sites | B04 |
| Plant-to-HQ and cloud networking | Interconnect, Cross-Cloud Interconnect, Network Connectivity Center | B04 |
| Autonomous driving ML and simulation | Agent Platform training, GPUs and TPUs, AI Hypercomputer; Spot for restartable jobs | B10 |
| Data monetization platform | BigQuery with governance (Knowledge Catalog), sharing controls, Sensitive Data Protection | B09, B11 |
| Breaches, incident response | Security Command Center, VPC Service Controls, least privilege, audit logs, a tested response plan | B11, P05 |
| EU data protection | Resource location org policy, EU regions, Assured Workloads data boundary controls, CMEK or EKM | B02, B11 |
| Dealer tools and build-to-order | Cloud Run apps, Apigee APIs over ERP and mainframe, Cloud SQL or Spanner for orders | B06, B08 |
| Mainframe and ERP modernization | Staged: wrap with APIs, then replatform or rebuild | B14 |
| Upskilling | Training plan, managed services to lower the ops load | B16 |

Exam signals: Each row's left column.

Trap: Moving EU driver data to a US region for cheaper compute.

Pillar tie-in: Security and reliability.

Check: How would you expose mainframe order data to new dealer apps without replacing the mainframe first?
<details><summary>Answer</summary>Put an API layer (Apigee) in front of it, reached over a private hybrid link. Modernize behind the API later.</details>

## Chunk 3: A target architecture
Problem it solves: One coherent picture.

Mental model:
- Foundation: org policies restricting EU data to EU regions; separate folders for regulated data; Security Command Center across the org.
- Connectivity: Interconnect from plants and HQ; Network Connectivity Center as the hub; Cross-Cloud Interconnect to other clouds.
- Edge: Google Distributed Cloud at plants for local processing when links drop.
- Data: vehicle and plant telemetry → Pub/Sub → Dataflow → BigQuery; governed with Knowledge Catalog; personal data de-identified before any sharing.
- AI: autonomous driving training on accelerators through Agent Platform; simulation at scale; in-vehicle AI features served from managed endpoints.
- Dealers: Cloud Run dealer apps and a CRM integration behind Apigee; orders in a managed database.

Exam signals: Questions often isolate security, EU data, or the AI training setup.

Trap: Sharing raw telemetry with partners to monetize it, without de-identification.

Pillar tie-in: Security and performance.

Check: Which controls keep EU personal data in the EU?
<details><summary>Answer</summary>The resource location org policy limited to EU regions, Assured Workloads data boundary controls where required, and access and key controls such as CMEK or EKM.</details>

## Chunk 4: Likely question angles
Problem it solves: Anticipate the questions.

Mental model:
- Security after breaches: posture management, exfiltration controls, incident response.
- EU compliance: residency and privacy for autonomous platforms.
- AI at scale: training infrastructure and cost models.
- Hybrid and edge: plants and rural connectivity.
- Business process: upskilling and change management (exam guide 4.2).
- Monetization: governed data sharing.

Exam signals: "Employees lack cloud skills" → training plus managed services.

Trap: Treating the people requirements as out of scope. Section 4 tests them.

Pillar tie-in: All.

Check: Name a control that limits data exfiltration from BigQuery even with valid credentials.
<details><summary>Answer</summary>VPC Service Controls.</details>

## Chunk 5: Practice
Scenario: KnightMotives trains autonomous driving models on large datasets. Jobs checkpoint every 30 minutes and can restart. Leadership wants to cut training cost without slowing the project much. Which approach fits best?

A) On-demand GPUs in a 3-year committed use discount before the team knows its steady baseline. B) Spot capacity for accelerators, with checkpointing to resume after preemption. C) Move training on-prem to reuse old servers. D) Train on general-purpose CPUs to avoid GPU cost.

Check: Which option, and why do the others fail?
<details><summary>Answer</summary>B. Checkpointed jobs tolerate preemption, and Spot cuts cost by a large margin. A locks in spend before the baseline is known. C leaves the team on obsolete hardware, which the case calls out. D slows training beyond reason for large models.</details>
