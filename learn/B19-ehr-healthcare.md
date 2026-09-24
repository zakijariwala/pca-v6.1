---
block: B19
title: EHR Healthcare
pillars: [security, reliability, operational-excellence]
exam_guide_refs: ["case-studies", "1.4", "2.1", "3.1", "3.2", "6.2"]
last_verified: 2026-09-24
sources:
  - https://cloud.google.com/learn/certification/guides/professional-cloud-architect
  - https://cloud.google.com/security/compliance/hipaa
  - https://docs.cloud.google.com/healthcare-api/docs/introduction
  - https://docs.cloud.google.com/managed-microsoft-ad/docs/overview
  - https://docs.cloud.google.com/apigee/docs/api-platform/get-started/what-apigee
  - https://docs.cloud.google.com/kubernetes-engine/docs/fleets-overview
  - https://docs.cloud.google.com/network-connectivity/docs/interconnect/concepts/dedicated-overview
  - https://docs.cloud.google.com/sql/docs/mysql/high-availability
  - https://docs.cloud.google.com/logging/docs/buckets
  - https://docs.cloud.google.com/kubernetes-engine/config-sync/docs/overview
  - https://docs.cloud.google.com/kubernetes-engine/enterprise/policy-controller/docs/overview
unverified: []
---
# B19 EHR Healthcare

Read the EHR Healthcare case study on Google's exam guide page before this block. This file paraphrases its themes and never quotes it.

## Chunk 1: EHR at a glance
Problem it solves: Apply the B17 method to EHR.

Mental model (paraphrased):
- Business: a SaaS provider of electronic health records to medical offices, hospitals, and insurers, growing fast. Wants faster onboarding of insurers, at least 99.9% availability for customer-facing systems, central visibility with proactive action, lower latency, regulatory compliance, lower admin cost, and analytics and predictions on healthcare trends.
- Current state: several colocation facilities, one lease ending soon. Web apps, many containerized on Kubernetes. MySQL, SQL Server, Redis, MongoDB. Legacy file and API integrations with insurers stay on-prem for years. Users in Microsoft Active Directory. Open-source monitoring; email alerts get ignored. Leadership blames outages on misconfiguration, capacity limits during spikes, and inconsistent monitoring.
- Technical: keep legacy insurer interfaces reachable from on-prem and cloud; consistent management of container apps; secure, fast on-prem link; consistent logging, retention, monitoring, and alerting; many container environments; scale and create environments on demand; ingest data from new providers.

Exam signals: "EHR" + "lease" → migration deadline. "EHR" + "compliance" → health data controls.

Trap: Planning to modernize the legacy insurer integrations. The case says they stay for now.

Pillar tie-in: Security and reliability.

Check: What availability does EHR require for customer-facing systems?
<details><summary>Answer</summary>At least 99.9%.</details>

## Chunk 2: Requirements mapped to services
Problem it solves: Turn each requirement into the expected service.

Mental model:

| Requirement theme | Fits | Block |
|---|---|---|
| Leave colocation before the lease ends | Rehost first: Migrate to Virtual Machines, Database Migration Service | B14 |
| Secure, fast on-prem link | Dedicated or Partner Interconnect, HA VPN as backup | B04 |
| Consistent container management across environments | GKE with fleets, Policy Controller, Config Sync | B06, P04, B13 |
| 99.9% availability, handle spikes | Regional GKE, Cloud SQL HA, autoscaling, global load balancing | B12 |
| Lower latency to all customers | Global external Application LB, Cloud CDN, regional placement | B04 |
| Consistent logging, retention, alerting | Aggregated log sinks, log buckets with set retention, SLO alerts to on-call | P05, P06 |
| Active Directory users | Sync to Cloud Identity or federate; Managed Microsoft AD for Windows workloads | B01, B02 |
| Onboard insurers, ingest provider data | Apigee for partner APIs; Pub/Sub, Dataflow into BigQuery; Cloud Healthcare API for FHIR, HL7v2, DICOM | B09 |
| Compliance | BAA where required, CMEK, VPC Service Controls, audit logs, Sensitive Data Protection | B11 |
| Insights and predictions | BigQuery, BigQuery ML, Agent Platform | B09, B10 |

Exam signals: Each row's left column.

Trap: HA VPN alone for high-volume, latency-sensitive clinical traffic.

Pillar tie-in: Security, reliability, operational excellence.

Check: Which Google Cloud API handles FHIR and HL7v2 data?
<details><summary>Answer</summary>The Cloud Healthcare API.</details>

## Chunk 3: A target architecture
Problem it solves: One coherent picture.

Mental model:
- Foundation: landing zone with folders per environment, org policies, Shared VPC, Interconnect to the remaining colocation site.
- Apps: regional GKE clusters in a fleet, deployed by Cloud Build and Cloud Deploy; config and policy enforced across clusters.
- Data: Cloud SQL HA for MySQL and SQL Server, Memorystore for Redis, MongoDB on a managed option; CMEK on regulated stores.
- Edge: global external Application LB with Cloud Armor and Cloud CDN.
- Partners: Apigee fronts APIs for new insurers; legacy integrations reached over Interconnect.
- Operate: aggregated log sinks with fixed retention; SLOs with burn-rate alerts to on-call tooling; VPC Service Controls around health data.

Exam signals: "Proactive action" → SLO alerts and autoscaling, not email.

Trap: A single-zone database under a 99.9% requirement.

Pillar tie-in: Reliability and security.

Check: EHR's alerts are ignored. Name two changes.
<details><summary>Answer</summary>Any two of: SLO-based burn-rate alerts; routing to on-call tools instead of email; fewer, actionable alerts with documentation; central dashboards across environments.</details>

## Chunk 4: Likely question angles
Problem it solves: Anticipate the questions.

Mental model:
- Migration under a deadline: rehost vs modernize; move data and databases.
- Hybrid: which link; DNS and routing to on-prem insurers.
- Reliability: design for 99.9%; DR plan rework.
- Compliance: health data controls, audit, retention.
- Operations: consistent environments through IaC and policy; fixing monitoring.
- Partners: API management for onboarding.

Exam signals: "Faster onboarding of insurers" → Apigee and standard interfaces.

Trap: Moving everything to one region with no DR because "it's all in Google Cloud now."

Pillar tie-in: All.

Check: What does Apigee add for EHR's insurer onboarding?
<details><summary>Answer</summary>A managed API layer: security, rate limits, analytics, and developer onboarding for partner APIs.</details>

## Chunk 5: Practice
Scenario: EHR must connect its remaining colocation site to Google Cloud for large clinical data transfers with consistent latency. Traffic must stay off the public internet. The site sits in a building that also hosts a Google colocation facility. Which option fits best?

A) HA VPN over the internet. B) Dedicated Interconnect with redundant connections. C) Partner Interconnect through a carrier. D) VPC Network Peering.

Check: Which option, and why do the others fail?
<details><summary>Answer</summary>B. Dedicated Interconnect gives private, high-bandwidth, consistent-latency links, and the site can reach the colocation facility. A runs over the internet. C works but adds a provider when direct access is available. D connects VPC networks to each other, not on-prem sites.</details>
