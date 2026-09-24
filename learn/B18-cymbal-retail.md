---
block: B18
title: Cymbal Retail
pillars: [performance, cost, security]
exam_guide_refs: ["case-studies", "1.1", "1.3", "1.4", "2.5", "3.2"]
last_verified: 2026-09-24
sources:
  - https://cloud.google.com/learn/certification/guides/professional-cloud-architect
  - https://docs.cloud.google.com/gemini-enterprise-agent-platform/vertex-ai-name-changes
  - https://docs.cloud.google.com/retail/docs/overview
  - https://cloud.google.com/gemini-enterprise-cx
  - https://docs.cloud.google.com/database-migration/docs/overview
  - https://docs.cloud.google.com/memorystore/docs/valkey/product-overview
  - https://docs.cloud.google.com/datastream/docs/overview
  - https://docs.cloud.google.com/sensitive-data-protection/docs/sensitive-data-protection-overview
unverified: []
---
# B18 Cymbal Retail

Read the Cymbal Retail case study on Google's exam guide page before this block. This file paraphrases its themes and never quotes it.

## Chunk 1: Cymbal at a glance
Problem it solves: Apply the B17 method to Cymbal.

Mental model (paraphrased):
- Business: a fast-growing online retailer with a huge, varied catalog. Wants gen AI to fill in product attributes, descriptions, and images from supplier data; natural language shopping assistance; better search relevance; higher conversion; lower call center and data center costs.
- Current state: mixed on-prem and cloud. MySQL, SQL Server, Redis, and MongoDB hold catalog and customer data. Kubernetes runs apps. SFTP and batch ETL connect legacy systems. The web app browses the catalog with plain database queries. An IVR routes calls to agents who key in orders. Monitoring uses open-source tools. Data silos block a single view of the customer.
- Technical: generate attributes and image variations; natural language product discovery; scale with the catalog; a human-in-the-loop review screen before content goes live; secure, compliant handling of customer data.

Exam signals: "Cymbal" + "catalog" → gen AI enrichment with review. "Cymbal" + "search" → AI commerce search.

Trap: Publishing generated content straight to the catalog. The case requires human review.

Pillar tie-in: Performance and cost.

Check: What review step does Cymbal require before generated content updates the catalog?
<details><summary>Answer</summary>Human-in-the-loop: associates approve, reject, or edit suggestions.</details>

## Chunk 2: Requirements mapped to services
Problem it solves: Turn each requirement into the expected service.

Mental model:

| Requirement theme | Fits | Block |
|---|---|---|
| Attribute and description generation | Gemini on Agent Platform, prompted with supplier data | B10 |
| Image variations and edits | Image generation models on Agent Platform (Imagen) | B10 |
| Natural language product discovery | AI Commerce Search (old name: Vertex AI Search for commerce) | B10 |
| Conversational shopping and support, fewer agent calls | Gemini Enterprise for Customer Experience agents | B10 |
| Human review UI | A small app (Cloud Run) over a review queue (Firestore or Cloud SQL) | B06, B08 |
| Modernize databases | Cloud SQL for MySQL and SQL Server, Memorystore for Redis or Valkey, Firestore or partner MongoDB | B08, B14 |
| Replace SFTP and batch ETL | Cloud Storage landing + Storage Transfer, Pub/Sub, Datastream, Dataflow | B09, B14 |
| Unified customer view | BigQuery with governed data | B09 |
| Customer data security | IAM, CMEK where required, Sensitive Data Protection, VPC Service Controls | B11 |

Exam signals: Each row's left column.

Trap: Keyword search tuning in SQL when the requirement is natural language discovery.

Pillar tie-in: Performance and security.

Check: What's the current name for Vertex AI Search for commerce?
<details><summary>Answer</summary>AI Commerce Search (on Gemini Enterprise for Customer Experience).</details>

## Chunk 3: A target architecture
Problem it solves: One coherent picture.

Mental model:
- Enrich: supplier files land in Cloud Storage → Cloud Run jobs call Gemini and image models → drafts go to a review queue → associates approve in a review app → approved data writes to the catalog database and search index.
- Discover: web and app call AI Commerce Search; a conversational agent handles shopping questions and hands off to humans when needed.
- Data: Datastream streams operational database changes to BigQuery for a single customer view.
- Platform: databases moved with Database Migration Service; apps on GKE; Memorystore for sessions and cache.
- Guard: Sensitive Data Protection masks personal data in analytics; Model Armor screens the shopping agent.

Exam signals: Questions often pick one stage: enrich, discover, or data.

Trap: Sending customer personal data into prompts without masking.

Pillar tie-in: Security and performance.

Check: How do operational database changes reach BigQuery with low latency and no custom code?
<details><summary>Answer</summary>Datastream change data capture into BigQuery.</details>

## Chunk 4: Likely question angles
Problem it solves: Anticipate the questions.

Mental model:
- Gen AI with guardrails: generation plus human review; grounding generated attributes in supplier data.
- Search and conversation: managed commerce search and CX agents over custom builds.
- Modernization: database migration paths, replacing file transfers with events.
- Cost: fewer call center minutes through self-service; exit data center hosting.
- Compliance: customer and payment data handling.

Exam signals: "Minimize development effort" points to the managed AI offerings.

Trap: Building a custom recommendation model when commerce search and recommendations are managed.

Pillar tie-in: Cost and operational excellence.

Check: Cymbal wants to cut call center costs. Which capability addresses that most?
<details><summary>Answer</summary>Conversational self-service agents that handle common questions and orders, with handoff to humans.</details>

## Chunk 5: Practice
Scenario: Cymbal wants generated product descriptions live within a day of supplier upload, without publishing errors. Which design fits best?

A) Generate descriptions with Gemini and write them straight to the catalog. B) Generate with Gemini, queue drafts, and have associates approve in a review app before publishing. C) Train a custom language model on the existing catalog first. D) Have associates write descriptions and use Gemini only for spell checking.

Check: Which option, and why do the others fail?
<details><summary>Answer</summary>B. It meets the speed goal and the human-in-the-loop requirement. A skips the required review. C adds months of work the requirement doesn't need. D doesn't reduce manual effort, which is the business goal.</details>
