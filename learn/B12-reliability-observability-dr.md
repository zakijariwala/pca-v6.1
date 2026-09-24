---
block: B12
title: Reliability, observability, DR
pillars: [reliability, cost]
exam_guide_refs: ["1.1", "1.2", "2.2", "4.1", "4.2", "6.2"]
last_verified: 2026-09-24
sources:
  - https://docs.cloud.google.com/stackdriver/docs/solutions/slo-monitoring
  - https://docs.cloud.google.com/stackdriver/docs/solutions/slo-monitoring/alerting-on-budget-burn-rate
  - https://docs.cloud.google.com/logging/docs/routing/overview
  - https://docs.cloud.google.com/trace/docs/overview
  - https://docs.cloud.google.com/architecture/dr-scenarios-planning-guide
  - https://docs.cloud.google.com/architecture/dr-scenarios-building-blocks
  - https://docs.cloud.google.com/backup-disaster-recovery/docs/concepts/backup-dr
  - https://docs.cloud.google.com/compute/docs/disks/snapshots
unverified: []
---
# B12 Reliability, observability, DR

## Chunk 1: The observability suite
Problem it solves: You can't fix what you can't see.

Mental model:

| Tool | Answers |
|---|---|
| Cloud Monitoring | Is it healthy? Metrics, dashboards, alerts, uptime checks |
| Cloud Logging | What happened? Logs, queries, routing |
| Cloud Trace | Where is the time going? Distributed request latency |
| Error Reporting | What's crashing? Grouped errors |

Log sinks route logs to log buckets, Cloud Storage, BigQuery, or Pub/Sub, at project, folder, or org level.

Exam signals: "Slow requests across microservices" → Trace. "Stream logs to a SIEM" → sink to Pub/Sub.

Trap: Using logs to measure latency across services. Trace does it without parsing.

Pillar tie-in: Operational excellence.

Check: Logs must go to a third-party SIEM in near real time. Which sink destination?
<details><summary>Answer</summary>Pub/Sub.</details>

## Chunk 2: SLIs, SLOs, error budgets
Problem it solves: "Reliable enough" needs a number both engineering and business accept.

Mental model: An SLI measures performance, such as the fraction of requests that succeed. An SLO sets the target, such as 99.9% over 30 days. The error budget is 1 − SLO: at 99.9%, you may fail 0.1% of requests. While budget remains, ship features. When it runs out, spend effort on reliability. Burn rate is how fast you consume the budget; alert on burn rate rather than on single failures.

Exam signals: "Balance feature velocity and stability", "stop paging on every error."

Trap: An SLO of 100%. It leaves no budget and blocks all change.

Pillar tie-in: Reliability and operational excellence.

Check: An SLO is 99.5% of requests succeed over 30 days. What's the error budget?
<details><summary>Answer</summary>0.5% of requests in the 30-day window.</details>

## Chunk 3: HA across zones and regions
Problem it solves: Match the failure domain you design for to the availability target.

Mental model:

| Target survives | Design |
|---|---|
| VM failure | MIG with autohealing |
| Zone failure | Regional MIG, regional GKE, Cloud SQL HA, regional disks |
| Region failure | Multi-region: global load balancer, backends in two regions, replicated data (Spanner multi-region, dual-region buckets, cross-region replicas) |

Every step up costs more and adds complexity.

Exam signals: "99.9% for all customer-facing systems" (EHR) often means zone-level HA plus a tested DR plan.

Trap: Multi-region active-active for a workload with a 99.5% target.

Pillar tie-in: Reliability and cost.

Check: What's the minimum design that survives a zone failure for a stateless web tier?
<details><summary>Answer</summary>A regional MIG (or regional GKE) behind a load balancer.</details>

## Chunk 4: RTO, RPO, and DR patterns
Problem it solves: Pick a DR design that meets the recovery targets at the lowest cost.

Mental model: RTO is how long you can be down. RPO is how much data you can lose, measured in time. Smaller values cost more. Google's DR guide groups patterns as cold, warm, and hot:

| Google pattern | Also called | Standby | Fits |
|---|---|---|---|
| Cold | Backup and restore | Backups only; rebuild on disaster | Hours to days RTO |
| Warm | Pilot light / warm standby | Scaled-down copy running, data replicating | Minutes to hours |
| Hot | Active-active / hot standby | Full capacity in both sites | Near zero |

Exam signals: "RPO 24 hours, RTO 12 hours, lowest cost" → cold. "RTO minutes" → warm or hot.

Trap: Hot DR for an internal tool with a one-day RTO.

Pillar tie-in: Reliability and cost.

Check: RTO 4 hours, RPO 15 minutes, cost matters. Which pattern?
<details><summary>Answer</summary>Warm: data replicates without pause (meeting the 15-minute RPO) and a scaled-down environment scales up within 4 hours.</details>

## Chunk 5: Backups on Google Cloud
Problem it solves: Backups that ransomware or an admin mistake can't delete.

Mental model: Snapshots protect disks (B05). Cloud SQL has automated backups and point-in-time recovery. Backup and DR Service manages backups from one place for Compute Engine, Persistent Disk, Filestore, VMware Engine VMs, Cloud SQL, and AlloyDB, using backup plans and backup vaults. Backup vaults store immutable, indelible backups.

Exam signals: "Backups must survive a compromised admin account", "central backup policy across services."

Trap: Backups stored in the same project with the same admins as production.

Pillar tie-in: Reliability and security.

Check: What Backup and DR Service feature protects backups from deletion?
<details><summary>Answer</summary>Backup vaults, which hold immutable and indelible backups.</details>

## Chunk 6: Test the plan
Problem it solves: An untested DR plan fails when you need it.

Mental model: Run DR drills on a schedule: restore from backups, fail over databases, promote replicas, and time each step against the RTO. Automate the recovery with IaC so a rebuild doesn't depend on memory.

Exam signals: "Ensure the DR plan works", "validate RTO."

Trap: Assuming a backup is good because the job reported success. Restore it.

Pillar tie-in: Reliability and operational excellence.

Check: What proves a backup works?
<details><summary>Answer</summary>A timed restore test.</details>
