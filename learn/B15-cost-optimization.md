---
block: B15
title: Cost optimization
pillars: [cost, sustainability]
exam_guide_refs: ["1.1", "2.3", "4.2"]
last_verified: 2026-09-24
sources:
  - https://docs.cloud.google.com/architecture/framework/cost-optimization
  - https://docs.cloud.google.com/compute/docs/sustained-use-discounts
  - https://docs.cloud.google.com/compute/docs/instances/committed-use-discounts-overview
  - https://docs.cloud.google.com/docs/cuds-spend-based
  - https://docs.cloud.google.com/compute/docs/instances/spot
  - https://docs.cloud.google.com/compute/docs/instances/apply-machine-type-recommendations-for-instances
  - https://docs.cloud.google.com/billing/docs/how-to/budgets
  - https://docs.cloud.google.com/billing/docs/how-to/export-data-bigquery
unverified: []
---
# B15 Cost optimization

## Chunk 1: The cost principles
Problem it solves: Cost cutting without a frame cuts the wrong things.

Mental model: The Well-Architected Framework's cost pillar has four core principles:
1. Align cloud spending with business value.
2. Foster a culture of cost awareness: people see the cost of their choices.
3. Optimize resource usage: provision what you need, pay for what you use.
4. Optimize continuously: monitor and adjust.

Exam signals: "Teams don't know what their services cost" → culture and visibility. "Reduce spend without hurting the SLO" → resource optimization.

Trap: Across-the-board cuts that break an SLO.

Pillar tie-in: Cost.

Check: Name the four cost optimization principles.
<details><summary>Answer</summary>Align spending with business value; foster a culture of cost awareness; optimize resource usage; optimize continuously.</details>

## Chunk 2: Discounts: sustained use, committed use, Spot
Problem it solves: Pay less for the same compute.

Mental model:

| Discount | How you get it | Size |
|---|---|---|
| Sustained use (SUD) | Automatic for eligible machine types that run much of the month | Up to 20% or 30%, by machine type |
| Resource-based committed use (CUD) | Commit to vCPU and memory in a region for 1 or 3 years | Up to 55%, up to 70% for memory-optimized |
| Spend-based committed use | Commit to an hourly spend on a service for 1 or 3 years | Varies by service |
| Spot VMs | Accept preemption | 60 to 91% off on-demand |

Exam signals: "Steady baseline, 3-year horizon" → CUD. "Batch, restartable" → Spot. "No commitment" → SUD, which applies on its own.

Trap: Committing for 3 years to cover a spike. Commit to the baseline; use on-demand or Spot for the peaks.

Pillar tie-in: Cost.

Check: A workload runs 24/7 at a steady size for the next 3 years. Which discount?
<details><summary>Answer</summary>A 3-year committed use discount on its baseline.</details>

## Chunk 3: Rightsizing and idle resources
Problem it solves: Oversized and forgotten resources are the easiest savings.

Mental model: Compute Engine recommends machine type changes from the previous 8 days of Cloud Monitoring metrics. Recommender also flags idle VMs, idle disks, and unused IP addresses. Act on them: resize, stop, or delete. For GKE, right requests (VPA) and Autopilot billing (B06) do the same job.

Exam signals: "VMs run at 10% CPU", "unattached disks."

Trap: Buying a CUD before rightsizing. You lock in the waste.

Pillar tie-in: Cost and sustainability.

Check: How many days of metrics feed Compute Engine's machine type recommendations?
<details><summary>Answer</summary>8 days.</details>

## Chunk 4: Visibility: budgets, labels, billing export
Problem it solves: You can't cut what you can't attribute.

Mental model: Budgets alert on actual or forecast spend (B01). Labels and tags attribute cost to teams and apps. Billing export to BigQuery sends detailed usage and cost data through the day; build reports with SQL or Looker. Export starts from the day you turn it on, so turn it on early.

Exam signals: "Show cost by team", "chargeback."

Trap: Turning on billing export the day finance asks for last year's breakdown.

Pillar tie-in: Cost.

Check: Why turn on billing export to BigQuery on day one?
<details><summary>Answer</summary>Export covers usage from the date you enable it. Earlier history won't be in the dataset.</details>

## Chunk 5: Cost inside design choices
Problem it solves: Architecture decides most of the bill.

Mental model: Revisit choices from earlier blocks through a cost lens:
- Storage class and location (B07): regional Standard beats multi-region for data used in one region.
- Serverless and scale to zero (B06) for spiky or idle workloads.
- BigQuery partitioning and pricing model (B09).
- Network egress: keep chatty services in the same region; cache with Cloud CDN.
- Managed services cut labor cost, which the bill doesn't show.

CapEx to OpEx: moving to cloud trades hardware purchases for monthly spend; plan for that shift (exam guide 4.2).

Exam signals: "Reduce cost" with a specific architecture: find the biggest line first.

Trap: Shaving VM sizes while cross-region egress dominates the bill.

Pillar tie-in: Cost.

Check: A dev environment sits idle nights and weekends. Name two ways to cut its cost.
<details><summary>Answer</summary>Any two of: schedule VMs to stop off-hours; move services to Cloud Run so they scale to zero; use Spot VMs for dev; delete and recreate environments from IaC.</details>

## Chunk 6: Cutting 30% without breaking the SLO
Problem it solves: The block's exit exercise.

Mental model: Work in this order.
1. Measure: billing export by service and label. Find the top 3 lines.
2. Remove waste: idle and unattached resources.
3. Rightsize: apply recommendations; set GKE requests.
4. Change pricing: CUDs for the baseline, Spot for batch.
5. Change design: storage classes, scale to zero, cut egress.
6. Check the SLO dashboards after each step (P06).

Exam signals: "Without affecting reliability."

Trap: Removing redundancy (a zone, a replica) to hit the number.

Pillar tie-in: Cost and reliability.

Check: Why rightsize before buying commitments?
<details><summary>Answer</summary>Commitments lock in capacity. Rightsizing first means you commit to what you need, not to today's waste.</details>
