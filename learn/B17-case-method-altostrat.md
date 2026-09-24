---
block: B17
title: Case study method and Altostrat Media
pillars: [reliability, cost, security]
exam_guide_refs: ["case-studies", "1.1", "1.3", "2.1", "2.5", "3.1", "4.1"]
last_verified: 2026-09-24
sources:
  - https://cloud.google.com/learn/certification/guides/professional-cloud-architect
  - https://cloud.google.com/learn/certification/cloud-architect
  - https://docs.cloud.google.com/kubernetes-engine/docs/fleets-overview
  - https://docs.cloud.google.com/distributed-cloud/docs
  - https://docs.cloud.google.com/storage/docs/autoclass
  - https://docs.cloud.google.com/model-armor/overview
  - https://docs.cloud.google.com/gemini-enterprise-agent-platform/vertex-ai-name-changes
  - https://docs.cloud.google.com/deploy/docs/overview
  - https://docs.cloud.google.com/stackdriver/docs/managed-prometheus
  - https://docs.cloud.google.com/network-connectivity/docs/interconnect/concepts/dedicated-overview
  - https://docs.cloud.google.com/iam/docs/workforce-identity-federation
unverified: []
---
# B17 Case study method and Altostrat Media

Read the Altostrat Media case study on Google's exam guide page before this block. This file paraphrases its themes and never quotes it.

## Chunk 1: A method for any case
Problem it solves: Case questions reward the answer that fits this company, not the generic best practice.

Mental model: Read each case once, and fill five lists.

| List | What goes in it |
|---|---|
| Business requirements | Outcomes the company wants: revenue, cost, speed, compliance |
| Technical requirements | Capabilities the solution must have |
| Current state | What runs today, where, and what hurts |
| Constraints | Deadlines, regulations, skills, budget, things that must not change |
| Keywords | Words that point to services: "hybrid", "summarize", "99.9%" |

The standard exam uses 2 of the 4 cases, for 20 to 30% of the content. On the exam, reread the case's requirements before each case question; the answer turns on one of them.

Exam signals: Any question that says "refer to the case study."

Trap: Answering from memory of what "most companies" need.

Pillar tie-in: All pillars, weighted by what the case stresses.

Check: What are the five lists?
<details><summary>Answer</summary>Business requirements, technical requirements, current state, constraints, keywords.</details>

## Chunk 2: Altostrat Media at a glance
Problem it solves: Know the company well enough to answer without rereading every detail.

Mental model (paraphrased):
- Business: a media company with a large library of audio and video. Wants gen AI for recommendations, natural language interaction, round-the-clock self-service support, content summaries, metadata extraction, and filtering harmful content. Revenue growth through personalization. Leadership names reliability and cost control as top priorities.
- Current state: already on Google Cloud: GKE for the content platform, Cloud Storage for media, BigQuery as the warehouse, Cloud Run functions for event-driven media tasks. Some ingestion and archival still on-prem, due to move. Identity mixes Google identity with third-party providers. Monitoring mixes Cloud Monitoring with open-source Prometheus; alerts go by email.
- Technical: modern CI/CD for containers from a central platform; secure, fast hybrid links for ingestion; Kubernetes on-prem and in cloud; lower storage cost as media grows; AI that detects harmful content and is auditable and explainable.

Exam signals: "Altostrat" + "cost" → storage classes. "Altostrat" + "hybrid" → Kubernetes across environments and the ingestion link.

Trap: Treating Altostrat as a migration case. Most of it already runs on Google Cloud.

Pillar tie-in: Reliability and cost first, per the executive statement.

Check: Which two priorities does Altostrat's leadership name?
<details><summary>Answer</summary>Reliability and cost management.</details>

## Chunk 3: Requirements mapped to services
Problem it solves: Turn each requirement into the service the exam expects.

Mental model:

| Requirement theme | Fits | Block |
|---|---|---|
| Kubernetes on-prem and in cloud, managed from one place | GKE fleets; Google Distributed Cloud for on-prem clusters | B06, P04 |
| Container CI/CD from one platform | Cloud Build, Artifact Registry, Cloud Deploy | B13 |
| Fast, secure hybrid ingestion | Dedicated or Partner Interconnect (HA VPN if bandwidth is modest) | B04 |
| Storage cost for a growing library | Autoclass or lifecycle rules to colder classes | B07 |
| Summaries, metadata, natural language support | Gemini on Agent Platform; pre-trained Speech, Video, Vision APIs; agents | B10 |
| Harmful content detection, auditable AI | Model Armor, content safety filters, logged prompts and outputs, human review | B10, B11 |
| Trends and content strategy | BigQuery, Looker | B09 |
| Consistent monitoring | Managed Service for Prometheus, Cloud Monitoring, alerting to on-call tools | P06 |
| Mixed identity providers | Workforce Identity Federation or Cloud Identity federation | B02 |

Exam signals: Each row's left column is the keyword to spot.

Trap: Rebuilding the whole platform on a new stack. Altostrat's stack already fits; improve it.

Pillar tie-in: Operational excellence and cost.

Check: Which feature lowers Altostrat's storage cost when access patterns vary by title?
<details><summary>Answer</summary>Autoclass, which moves each object to a colder class when it isn't read and back when it is.</details>

## Chunk 4: A target architecture
Problem it solves: Hold one coherent picture so answers stay consistent.

Mental model:
- Ingest: on-prem ingestion → Interconnect → Cloud Storage (Autoclass).
- Process: object events → Cloud Run functions → Speech, Video, and Gemini for transcripts, metadata, summaries → BigQuery.
- Serve: GKE (with on-prem clusters in the same fleet) behind a global load balancer and Cloud CDN.
- Engage: a Gemini-based support agent, with Model Armor screening prompts and responses.
- Operate: Cloud Build → Artifact Registry → Cloud Deploy; Managed Service for Prometheus; SLO burn-rate alerts to on-call tooling instead of email.

Exam signals: Questions often isolate one arrow of this picture.

Trap: Moving the archive to Archive class when editors pull old footage every week.

Pillar tie-in: Reliability and cost.

Check: Where does Model Armor sit in this design?
<details><summary>Answer</summary>Between users and the model: it screens prompts on the way in and responses on the way out.</details>

## Chunk 5: Likely question angles
Problem it solves: Anticipate what the exam asks about this case.

Mental model:
- Cost: pick storage classes and lifecycle for media at scale.
- Hybrid: pick the link and the Kubernetes management model across on-prem and cloud.
- AI: pick the least-effort service for summaries or moderation; make AI auditable.
- Delivery: modernize CI/CD with safe rollouts.
- Operations: fix email-only alerting; keep Prometheus investments.

Exam signals: "Altostrat wants to..." followed by one of the business requirements.

Trap: Custom-training a moderation model when safety filters and Model Armor meet the need.

Pillar tie-in: All.

Check: Altostrat's alerts go to email and get missed. What's the fix?
<details><summary>Answer</summary>SLO-based alerting policies routed to on-call channels (PagerDuty, chat, Pub/Sub) with burn-rate conditions, not raw email.</details>

## Chunk 6: Practice
Scenario: Altostrat wants to cut the cost of its media library without slowing playback of popular titles. Access varies by title and drops off after release. Which approach fits best?

A) Move all objects to Archive class. B) Turn on Autoclass for the media buckets. C) Copy older titles to Persistent Disk snapshots. D) Keep everything in multi-region Standard and buy committed use discounts.

Check: Which option, and why do the others fail?
<details><summary>Answer</summary>B. Autoclass moves unread objects to colder classes and brings them back to Standard when read, so popular titles stay fast. A adds retrieval cost and a 365-day minimum for titles still in use. C misuses disk snapshots as object storage. D doesn't lower storage cost; committed use discounts apply to compute, not Cloud Storage.</details>
