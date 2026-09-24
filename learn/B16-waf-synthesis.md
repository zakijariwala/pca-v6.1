---
block: B16
title: Well-Architected Framework and synthesis
pillars: [security, reliability, cost, performance, operational-excellence, sustainability]
exam_guide_refs: ["1.1", "1.2", "1.5", "4.2", "6.1"]
last_verified: 2026-09-24
sources:
  - https://docs.cloud.google.com/architecture/framework
  - https://docs.cloud.google.com/architecture/framework/operational-excellence
  - https://docs.cloud.google.com/architecture/framework/security
  - https://docs.cloud.google.com/architecture/framework/reliability
  - https://docs.cloud.google.com/architecture/framework/cost-optimization
  - https://docs.cloud.google.com/architecture/framework/performance-optimization
  - https://docs.cloud.google.com/architecture/framework/sustainability
unverified: []
---
# B16 Well-Architected Framework and synthesis

## Chunk 1: The framework's shape
Problem it solves: The exam guide calls the Well-Architected Framework a key requirement. You need its structure in your head.

Mental model: Six pillars, each a non-functional focus area, plus cross-pillar perspectives (AI and ML, financial services) that apply the pillars to a domain.

| Pillar | Goal |
|---|---|
| Operational excellence | Deploy, operate, monitor, and manage workloads well |
| Security, privacy, and compliance | Protect data and workloads; meet regulations |
| Reliability | Resilient, highly available workloads |
| Cost optimization | Maximize business value of cloud spend |
| Performance optimization | Tune resources for performance |
| Sustainability | Workloads with a low environmental footprint |

Five framework-wide principles sit above the pillars: design for change, document your architecture, simplify and use fully managed services, decouple your architecture, use a stateless architecture.

Exam signals: "Which pillar does this recommendation serve?"

Trap: Forgetting sustainability. It's the sixth pillar.

Pillar tie-in: All six.

Check: Name the six pillars.
<details><summary>Answer</summary>Operational excellence; security, privacy, and compliance; reliability; cost optimization; performance optimization; sustainability.</details>

## Chunk 2: Pillar principles, one line each
Problem it solves: Recall the core principles fast enough to use them on a question.

Mental model:

| Pillar | Core principles |
|---|---|
| Operational excellence | Operational readiness with CloudOps; manage incidents and problems; manage and optimize resources; automate and manage change; improve and innovate |
| Security | Security by design; zero trust; shift-left security; preemptive cyber defense; use AI securely and responsibly |
| Reliability | Define reliability from user-experience goals; realistic targets; redundancy; horizontal scaling; observability; graceful degradation; test recovery from failures and data loss; thorough postmortems |
| Cost | Align spend with business value; culture of cost awareness; optimize resource usage; optimize continuously |
| Performance | Plan resource allocation; elasticity; modular design; monitor and improve |
| Sustainability | Low-carbon regions; energy-efficient AI and ML; optimize resource usage; energy-efficient software; optimize data and storage; measure and improve |

Exam signals: "Shift left" → security in CI. "Graceful degradation" → reliability. "Low-carbon region" → sustainability.

Trap: Treating the pillars as separate checklists. Most decisions trade one against another.

Pillar tie-in: All six.

Check: Which pillar lists "choose regions that consume low-carbon energy"?
<details><summary>Answer</summary>Sustainability.</details>

## Chunk 3: Business vs technical requirements
Problem it solves: Case studies mix goals and constraints. Sorting them drives the answer.

Mental model: Business requirements state outcomes: grow revenue, cut cost, onboard partners faster, comply with regulation. Technical requirements state how: 99.9% availability, hybrid connectivity, container management. Map each business requirement to a measurable success metric (KPI, ROI) and to the technical requirements that serve it (exam guide 1.1).

Exam signals: "The CEO wants..." is business. "Must support..." is technical.

Trap: Picking the elegant technical answer that ignores a stated business constraint such as cost or deadline.

Pillar tie-in: Cost and operational excellence.

Check: "Decrease infrastructure administration costs" (EHR). Business or technical, and which pillar?
<details><summary>Answer</summary>Business requirement; cost optimization, served by managed services and automation (operational excellence).</details>

## Chunk 4: Section 4 people and process questions
Problem it solves: 15% of the exam is about processes and people, not products.

Mental model: Exam guide 4.2 covers stakeholder management, change management, team skills readiness, decision-making, customer success, cost (CapEx and OpEx), and business continuity. Answers that win: involve stakeholders early, pilot before a full rollout, train teams before handing them new platforms, measure with agreed metrics, and roll out changes in stages.

Exam signals: "The team has no Kubernetes experience", "stakeholders resist the change."

Trap: A pure technology answer to a people problem, such as picking a more powerful product for a team that can't run it.

Pillar tie-in: Operational excellence.

Check: A team with no container experience must run a new microservices platform in 3 months. What should the answer include besides the platform?
<details><summary>Answer</summary>Skills readiness: training or partner support, a managed option (Cloud Run or GKE Autopilot) that lowers the ops load, and a staged rollout.</details>

## Chunk 5: The cross-service decision recap
Problem it solves: One page that points to the right block's decision tree.

Mental model:

| Decision | Rule of thumb | Block |
|---|---|---|
| Folder layout | Org → business unit or environment → projects | B01 |
| Access | Groups, predefined roles, no keys, deny for never-allowed | B02 |
| Network | One global custom VPC; Shared VPC for central control | B03 |
| Hybrid | HA VPN fast and cheap; Interconnect for volume | B04 |
| Compute | Cloud Run → GKE → Compute Engine | B06 |
| Storage | Class by access frequency, location by DR need | B07 |
| Database | Cloud SQL → AlloyDB → Spanner; Bigtable for key-value scale | B08 |
| Analytics | Pub/Sub → Dataflow → BigQuery | B09 |
| AI | Pre-trained API → Gemini → BigQuery ML → tune → custom | B10 |
| Keys | Default → CMEK → HSM → EKM | B11 |
| DR | Cold → warm → hot, by RTO/RPO and cost | B12 |
| Delivery | Cloud Build → Artifact Registry → Cloud Deploy | B13 |
| Migration | Rehost first under deadline | B14 |
| Cost | Rightsize, then commit | B15 |

Exam signals: Any scenario question.

Trap: Picking the most advanced option in each row by default. Start at the left.

Pillar tie-in: All six.

Check: A scenario needs a relational database for one region with standard MySQL. Where on the database rule do you stop?
<details><summary>Answer</summary>Cloud SQL, the first step. Nothing in the scenario pushes you further.</details>

## Chunk 6: Critiquing an architecture pillar by pillar
Problem it solves: The block's exit task, and the way case study questions are built.

Mental model: For each pillar, ask one question.
- Operational excellence: Is it deployed by pipeline and IaC, with alerts on SLOs?
- Security: Least privilege, no keys, private networking, encryption fit for the data?
- Reliability: Which failure domain does it survive, and does that match the target?
- Cost: Is anything idle, oversized, or over-replicated for the need?
- Performance: Is data near compute and users; does it scale out?
- Sustainability: Region choice, utilization, data kept longer than needed?

Exam signals: "What should the architect recommend first?" → the pillar with the biggest gap against stated requirements.

Trap: Fixing a minor gap when the design misses a hard requirement elsewhere.

Pillar tie-in: All six.

Check: A design runs a single-zone Cloud SQL instance for a system with a 99.9% requirement, and uses service account keys in CI. Which two findings come first?
<details><summary>Answer</summary>Reliability: make Cloud SQL highly available (regional). Security: replace keys with Workload Identity Federation or impersonation.</details>
