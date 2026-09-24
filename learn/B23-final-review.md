---
block: B23
title: Final review
pillars: [security, reliability, cost, performance, operational-excellence, sustainability]
exam_guide_refs: ["1", "2", "3", "4", "5", "6"]
last_verified: 2026-09-24
sources:
  - https://cloud.google.com/learn/certification/cloud-architect
  - https://cloud.google.com/learn/certification/guides/professional-cloud-architect
  - learn/ files B00 to B20 and P01 to P08, which carry the per-fact sources
unverified: []
---
# B23 Final review

Everything here comes from earlier blocks; the block ID in brackets tells you where to reread. Check `last_verified` on those files: facts older than 90 days need a fresh look at the docs.

## Chunk 1: One-screen cheat sheet per section

**Section 1: Design and planning (~25%)**
- Compute: Cloud Run → GKE (Autopilot first) → Compute Engine, stopping at the first that fits [B06].
- Database: Cloud SQL → AlloyDB (PostgreSQL + analytics) → Spanner (global, 99.999%); Bigtable for key-value scale; Firestore for documents; Memorystore for cache [B08].
- Storage: class by read frequency (30/90/365-day minimums); location by DR need; Autoclass when access varies [B07].
- AI: pre-trained API → Gemini → BigQuery ML → tune → custom [B10].
- Migration: rehost under deadline; Migration Center to assess; appliance when bandwidth math fails [B14].
- DR: cold, warm, hot by RTO/RPO and cost [B12].

**Section 2: Managing and provisioning (~18%)**
- Network: one global custom VPC; Shared VPC for central control; peering isn't transitive; NCC for hub-and-spoke [B03, B04].
- Hybrid: HA VPN (99.99% with both interfaces and BGP) vs Interconnect (Dedicated 10/100/400 Gbps circuits, Partner 50 Mbps to 50 Gbps) [B04].
- Egress: Cloud NAT for internet; Private Google Access for Google APIs only [B03].
- Load balancers: HTTP(S) → Application LB; TCP/UDP → Network LB; passthrough keeps client IP [B04].
- MIGs: regional for zone failure; autohealing initial delay ≥ boot time [B05].
- ML ops: Agent Platform (Vertex AI) Pipelines; Spot accelerators for restartable training [B10].

**Section 3: Security and compliance (~19%)**
- Identity: groups, predefined roles, no basic roles in prod, no keys: attached service accounts, impersonation, Workload Identity Federation [B02].
- Guardrails: deny policies beat allow; org policies block configurations regardless of IAM [B02].
- Keys: default → CMEK → Cloud HSM (FIPS 140-2 Level 3) → EKM (outside Google) [B11].
- Exfiltration: VPC Service Controls. Edge: Cloud Armor. Access without VPN: IAP [B11].
- Audit: Admin Activity always on; Data Access off except BigQuery; `_Required` 400 days [P05].
- AI: Model Armor for prompts and responses; Sensitive Data Protection for PII [B10].

**Section 4: Processes (~15%)**
- Restore service first (rollback), then diagnose [P07].
- SLOs and error budgets balance change and stability [B12].
- People: skills readiness, stakeholder buy-in, staged change [B16].
- Cost: CapEx to OpEx; rightsize then commit [B15].

**Section 5: Implementation (~11%)**
- Cloud Build → Artifact Registry → Cloud Deploy; Binary Authorization at deploy [B13].
- Terraform with remote state and impersonation; emulators for local tests [P02, B08].
- APIs to partners: Apigee [B19].

**Section 6: Operations excellence (~12%)**
- Burn-rate alerts over raw thresholds; uptime checks from outside; Managed Service for Prometheus keeps PromQL [P06].
- Canary with SLO gates [B13].
- Profiler for CPU and memory hot spots [P06].

Check: Which section carries the most weight?
<details><summary>Answer</summary>Section 1, design and planning, at about 25%.</details>

## Chunk 2: Top 30 traps
Each is a wrong answer that looks right.

| # | Trap | Why it fails | Block |
|---|---|---|---|
| 1 | Downloading a service account key when another method works | Long-lived secret; impersonation or federation does the job | B02 |
| 2 | Basic Editor "to unblock" someone | Near-total project access | B02 |
| 3 | IAM to enforce a "never allowed" rule | An admin can still grant it; use org policy or deny | B02 |
| 4 | A budget to cap spending | Budgets only alert | B01 |
| 5 | IAM condition on a label | Only tags drive conditions | B01 |
| 6 | One VPC per region | GCP VPCs are global | B03 |
| 7 | Auto mode VPC in production | Its ranges may overlap on-prem | B03 |
| 8 | Chaining peerings for transitivity | Peering isn't transitive | B03 |
| 9 | Cloud NAT when only Google APIs are needed | Private Google Access covers it | B03 |
| 10 | One HA VPN tunnel and calling it HA | 99.99% needs both interfaces | B04 |
| 11 | Network LB for URL-based routing | That's an Application LB | B04 |
| 12 | A zonal VM "survives" because the VPC is global | VM scope is zonal | B00 |
| 13 | Autohealing initial delay shorter than boot | Recreate loop | B05 |
| 14 | Spot VMs for a stateful database | Preemption | B05 |
| 15 | GKE for one spiky stateless API | Cloud Run does it with less ops | B06 |
| 16 | App Engine for a new app | Google recommends Cloud Run | B06 |
| 17 | Archive class for data read weekly | Retrieval fees and 365-day minimum | B07 |
| 18 | Versioning as compliance retention | Use a locked retention policy | B07 |
| 19 | Read replica as HA | Replicas don't fail over on their own | B08 |
| 20 | Spanner for a small regional app | Cloud SQL fits at lower cost | B08 |
| 21 | Dataflow only to copy messages into BigQuery | BigQuery subscription does it | B09 |
| 22 | Custom model for a standard vision task | Pre-trained API | B10 |
| 23 | System prompt as prompt-injection defense | Model Armor | B10 |
| 24 | IAM alone against exfiltration | VPC Service Controls | B11 |
| 25 | Cloud Armor to control employee access | IAP | B11 |
| 26 | Audit logs for Google staff access | Access Transparency | B11 |
| 27 | 100% SLO | No error budget, no change | B12 |
| 28 | Hot DR for a one-day RTO | Cold or warm is cheaper | B12 |
| 29 | Refactoring everything before a lease ends | Rehost first | B14 |
| 30 | Buying commitments before rightsizing | Locks in waste | B15 |

Check: Which three traps involve identity and access?
<details><summary>Answer</summary>Any three of 1, 2, 3, 5, 24, 25, 26.</details>

## Chunk 3: The 48-hour plan
Problem it solves: The last two days decide how much of your preparation shows up in the exam room.

**48 to 24 hours out**
- Rerun B21 cold. Remediate any section below its pass mark with B22, 30 minutes per weak spot, no more.
- Reread the four case studies on Google's page and your five lists for each [B17 method]. The exam uses 2 of the 4.
- Reread Chunk 2 above.

**24 hours out**
- Light review only: Chunk 1 above and your own contrast tables from B22.
- Check exam logistics on Google's certification page: time, format (online or test center), and the current candidate rules.
- Stop studying by early evening. Sleep.

**Exam day**
- 2 hours, 50 to 60 questions, multiple choice and multiple select.
- First pass: answer what you know, flag the rest. Second pass: flagged questions.
- For each question, underline the tiebreaker: cheapest, least ops, fewest changes, most secure, Google-recommended [B00].
- For case questions, check the company's stated requirement before you pick.

Check: How many case studies appear on the standard exam?
<details><summary>Answer</summary>2, making up 20 to 30% of the content. Prepare all 4.</details>

## Chunk 4: 15 rapid-fire questions
Answer each in under 20 seconds.

1. Spot VM discount range? <details><summary>Answer</summary>60 to 91% off on-demand.</details>
2. Minimum storage duration for Coldline? <details><summary>Answer</summary>90 days.</details>
3. Longest V4 signed URL? <details><summary>Answer</summary>7 days.</details>
4. Default Cloud NAT minimum ports per VM, static allocation? <details><summary>Answer</summary>64.</details>
5. Health check source ranges? <details><summary>Answer</summary>`35.191.0.0/16` and `130.211.0.0/22`.</details>
6. IAP TCP forwarding source range? <details><summary>Answer</summary>`35.235.240.0/20`.</details>
7. Spanner SLA, multi-region? <details><summary>Answer</summary>99.999%.</details>
8. Audit log type that's off by default? <details><summary>Answer</summary>Data Access (except BigQuery).</details>
9. Retention of the `_Required` log bucket? <details><summary>Answer</summary>400 days.</details>
10. Default wait before a KMS key version is destroyed? <details><summary>Answer</summary>30 days.</details>
11. Folder nesting limit? <details><summary>Answer</summary>10 levels.</details>
12. Days of metrics behind machine type recommendations? <details><summary>Answer</summary>8.</details>
13. What HA VPN needs for 99.99%? <details><summary>Answer</summary>Tunnels on both interfaces, BGP through Cloud Router.</details>
14. Current name for Vertex AI? <details><summary>Answer</summary>Gemini Enterprise Agent Platform.</details>
15. Service that replaced Container Registry? <details><summary>Answer</summary>Artifact Registry.</details>

Check: Which of these would you reread before the exam? List the numbers you missed and their blocks.
<details><summary>Answer</summary>Your list. Every fact above comes from an earlier block or lab: for example 5 from P03, 6 from the B03 lab, 9 from P05, 10 from B11.</details>
