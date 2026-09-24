---
block: B14
title: Migration
pillars: [cost, reliability, operational-excellence]
exam_guide_refs: ["1.4", "1.1", "5.1"]
last_verified: 2026-09-24
sources:
  - https://docs.cloud.google.com/architecture/migration-to-gcp-getting-started
  - https://docs.cloud.google.com/architecture/migration-to-gcp-assessing-and-discovering-your-workloads
  - https://docs.cloud.google.com/migration-center/docs/migration-center-overview
  - https://docs.cloud.google.com/migrate/virtual-machines/docs/5.0/get-started/overview
  - https://docs.cloud.google.com/database-migration/docs/overview
  - https://docs.cloud.google.com/storage-transfer/docs/overview
  - https://docs.cloud.google.com/transfer-appliance/docs/4.0/overview
  - https://docs.cloud.google.com/architecture/migration-to-google-cloud-transferring-your-large-datasets
  - https://docs.cloud.google.com/compute/docs/nodes/bringing-your-own-licenses
unverified: []
---
# B14 Migration

## Chunk 1: Types of migration
Problem it solves: Each workload needs a migration approach that matches its value, deadline, and team.

Mental model: Google's migration guide names four types.

| Type | Also called | Change | Fits |
|---|---|---|---|
| Rehost | Lift and shift | Little or none | Tight deadline, app works as-is |
| Replatform | Lift and optimize | Some, such as moving to managed databases | Quick wins without a rewrite |
| Refactor | Move and improve | Code changes to use cloud features | App worth modernizing |
| Rebuild | Remove and replace (rip and replace) | Rewrite or replace with SaaS | Legacy app that blocks the business |

Section 1.1 of the exam guide frames the same choice as workload disposition: build, buy, modify, or deprecate. Some workloads should retire instead of moving.

Exam signals: "Data center lease expires in 6 months" (EHR) → rehost first, modernize later. "No plan to change the legacy insurer interfaces" → leave them in place and connect.

Trap: Refactoring everything before the lease ends.

Pillar tie-in: Cost and operational excellence.

Check: A company must leave its data center in 4 months. Which migration type fits most workloads?
<details><summary>Answer</summary>Rehost (lift and shift). Modernize after the deadline.</details>

## Chunk 2: Assess and plan: Migration Center
Problem it solves: You can't plan a move without an inventory, dependencies, and a cost estimate.

Mental model: Migration Center discovers assets in your current environment, assesses them, estimates future Google Cloud cost, and helps plan the migration. Group workloads into waves by dependency: move systems that depend on each other together, start with low-risk ones, and test each wave.

Exam signals: "Build a business case", "estimate cost before migrating", "unknown dependencies."

Trap: Moving an app in wave 1 while its database stays on-prem for wave 3, over a high-latency link.

Pillar tie-in: Cost and reliability.

Check: What Google tool discovers on-prem assets and estimates their cloud cost?
<details><summary>Answer</summary>Migration Center.</details>

## Chunk 3: Moving VMs: Migrate to Virtual Machines
Problem it solves: Hundreds of VMs must move with minimal downtime.

Mental model: Migrate to Virtual Machines moves VMs from on-prem vSphere, AWS, Azure, or Google Cloud VMware Engine to Compute Engine. It replicates disks in the background, lets you test-clone before cutover, then cuts over with a short outage. Plan waves and run a test clone for each.

Exam signals: "500 VMware VMs", "minimal downtime", "test before cutover."

Trap: Exporting disk images by hand for hundreds of VMs.

Pillar tie-in: Reliability and operational excellence.

Check: Name three sources Migrate to Virtual Machines supports.
<details><summary>Answer</summary>Any three of: on-prem vSphere, AWS, Azure, Google Cloud VMware Engine.</details>

## Chunk 4: Moving databases: Database Migration Service
Problem it solves: Databases must move with the app, without a long outage.

Mental model: Database Migration Service migrates into Cloud SQL and AlloyDB for PostgreSQL. Continuous migration keeps the destination in sync with the source until you cut over, giving minimal downtime. Homogeneous moves keep the engine (MySQL to Cloud SQL for MySQL). Heterogeneous moves change it (for example, Oracle to PostgreSQL) and need schema and code conversion.

Exam signals: "Minimal downtime database migration", "move MySQL to Cloud SQL."

Trap: A dump-and-restore with hours of downtime when the requirement is minimal downtime.

Pillar tie-in: Reliability.

Check: How does Database Migration Service keep downtime short?
<details><summary>Answer</summary>Continuous migration: it replicates ongoing changes until the destination is in sync, then you cut over.</details>

## Chunk 5: Moving data: online vs offline
Problem it solves: Pick a transfer method your bandwidth and deadline can support.

Mental model: Do the math first. At 1 Gbps, 1 GB takes about 8 seconds, and 100 TB takes over 10 days. At 100 Mbps, 100 TB takes over 100 days.

| Method | Fits |
|---|---|
| `gcloud storage` | Small to medium datasets over a good link |
| Storage Transfer Service | Large or scheduled transfers: from S3, Azure Blob, HTTP, other buckets, or on-prem file systems through agents |
| Transfer Appliance | Large data, limited bandwidth, tight deadline: Google ships a device, you fill it, ship it back |

Exam signals: "200 TB, 100 Mbps link, 1-month deadline" → Transfer Appliance. "Nightly sync from S3" → Storage Transfer Service.

Trap: Planning an online transfer without checking bandwidth. The math often rules it out.

Pillar tie-in: Cost and reliability.

Check: 100 TB over a 100 Mbps link. About how long online, and what's the alternative?
<details><summary>Answer</summary>Over 100 days. Transfer Appliance.</details>

## Chunk 6: Licensing and financial impact
Problem it solves: License terms can change the cost of a migration more than compute does.

Mental model: Choose between pay-as-you-go licenses included in Google Cloud images and bring your own license (BYOL). Licenses tied to physical cores or hosts need sole-tenant nodes (B05). Weigh CapEx you stop spending on hardware against OpEx you start paying monthly.

Exam signals: "Existing Windows Server and SQL Server licenses", "reduce licensing cost."

Trap: Ignoring per-core license terms and running BYOL software on shared hosts.

Pillar tie-in: Cost.

Check: Per-physical-core licenses move to Google Cloud. Where must the VMs run?
<details><summary>Answer</summary>On sole-tenant nodes, if the license terms require dedicated hardware.</details>

## Chunk 7: Planning 500 VMs and 200 TB with a fixed deadline
Problem it solves: Put the pieces together for the block's exit scenario.

Mental model:
1. Discover and assess with Migration Center; build the cost case.
2. Build the landing zone and hybrid link (B01 to B04) before wave 1.
3. Start Transfer Appliance or Storage Transfer Service early; data is the long pole.
4. Group VMs into waves by dependency; rehost with Migrate to Virtual Machines, test-clone each wave.
5. Move databases with Database Migration Service in continuous mode; cut over with their apps.
6. Leave modernization for after the deadline.

Exam signals: "Fixed date", "limited bandwidth", "minimal downtime."

Trap: Starting the 200 TB transfer in the last month.

Pillar tie-in: Reliability.

Check: Why start the data transfer before the first VM wave?
<details><summary>Answer</summary>Large data takes the longest to move, whether online or by appliance. It sets the critical path.</details>
