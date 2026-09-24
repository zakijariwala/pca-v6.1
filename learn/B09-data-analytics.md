---
block: B09
title: Data and analytics
pillars: [performance, cost, operational-excellence]
exam_guide_refs: ["1.1", "1.3", "2.2"]
last_verified: 2026-09-24
sources:
  - https://docs.cloud.google.com/bigquery/docs/editions-intro
  - https://docs.cloud.google.com/bigquery/docs/partitioned-tables
  - https://docs.cloud.google.com/bigquery/docs/clustered-tables
  - https://docs.cloud.google.com/pubsub/docs/overview
  - https://docs.cloud.google.com/pubsub/docs/bigquery
  - https://docs.cloud.google.com/dataflow/docs/overview
  - https://docs.cloud.google.com/managed-spark/docs/concepts/overview
  - https://docs.cloud.google.com/composer/docs/composer-3/composer-overview
  - https://docs.cloud.google.com/datastream/docs/overview
  - https://docs.cloud.google.com/lakehouse/docs/introduction
unverified: []
---
# B09 Data and analytics

Several products here were renamed after the exam guide came out. This file uses the current name and gives the exam guide name in brackets.

| Exam guide name | Current name |
|---|---|
| Dataproc | Managed Service for Apache Spark |
| Cloud Composer | Managed Service for Apache Airflow |
| Dataplex | Knowledge Catalog |
| BigLake | Lakehouse |

## Chunk 1: BigQuery: storage, compute, pricing
Problem it solves: Query terabytes with SQL without running a warehouse cluster.

Mental model: BigQuery separates storage from compute. You pay for storage, and for compute one of two ways:

| Model | Pay for | Fits |
|---|---|---|
| On-demand | Data processed per query (per TiB) | Spiky or light use |
| Capacity (editions) | Slots, reserved or autoscaled | Steady heavy use, predictable cost |

Editions are Standard, Enterprise, and Enterprise Plus. You can mix models per project.

Exam signals: "Unpredictable ad hoc queries" → on-demand. "Predictable monthly analytics spend" → capacity with reservations.

Trap: `SELECT *` on a wide table under on-demand pricing. You pay for every column scanned.

Pillar tie-in: Cost.

Check: Under on-demand pricing, what drives cost?
<details><summary>Answer</summary>Bytes processed by each query.</details>

## Chunk 2: Partitioning and clustering
Problem it solves: Queries scan the whole table when they need one day of it.

Mental model: Partitioning splits a table by a time-unit column, ingestion time, or an integer range. A query that filters on the partition column scans only matching partitions (pruning). Clustering sorts storage blocks by up to four columns, so filters on those columns skip blocks. Use both: partition by date, cluster by customer ID.

Exam signals: "Queries always filter by date and cost too much" → partition by date.

Trap: Partitioning on a column the queries never filter on.

Pillar tie-in: Cost and performance.

Check: A table is partitioned by `event_date`. A query filters only on `customer_id`. Does partitioning cut its cost?
<details><summary>Answer</summary>No. Pruning needs a filter on the partition column. Cluster on `customer_id` to help that query.</details>

## Chunk 3: Pub/Sub
Problem it solves: Decouple producers from consumers and absorb bursts.

Mental model: Pub/Sub is asynchronous messaging. Publishers send to a topic; each subscription gets a copy. Subscribers pull, or Pub/Sub pushes to an endpoint. Export subscriptions write straight to BigQuery or Cloud Storage with no pipeline code, when messages need no transformation.

Exam signals: "Ingest events from millions of devices", "decouple services", "buffer spikes."

Trap: Building a Dataflow job only to copy messages unchanged into BigQuery. A BigQuery subscription does that alone.

Pillar tie-in: Reliability.

Check: Messages need no transformation before landing in BigQuery. What's the simplest path?
<details><summary>Answer</summary>A Pub/Sub BigQuery subscription.</details>

## Chunk 4: Dataflow vs Managed Service for Apache Spark
Problem it solves: Transform data at scale in batch or streaming.

Mental model:

| | Dataflow | Managed Service for Apache Spark (Dataproc) |
|---|---|---|
| Model | Apache Beam; same code for batch and streaming | Spark and Hadoop ecosystem |
| Ops | Serverless, autoscales workers | Clusters you size, or serverless Spark |
| Fits | New pipelines, streaming with exactly-once processing | Existing Spark or Hadoop jobs to move as-is |

Exam signals: "Existing Spark jobs", "Hadoop migration" → Managed Spark. "New streaming pipeline, minimal ops" → Dataflow.

Trap: Rewriting working Spark jobs in Beam during a migration with a deadline.

Pillar tie-in: Operational excellence.

Check: A company moves 300 existing Spark jobs from on-prem Hadoop. Which service?
<details><summary>Answer</summary>Managed Service for Apache Spark (Dataproc in the exam guide). The jobs run as-is.</details>

## Chunk 5: Orchestration and CDC
Problem it solves: Multi-step pipelines need scheduling and dependencies. Operational databases need to feed analytics without batch dumps.

Mental model: Managed Service for Apache Airflow (Cloud Composer) runs Apache Airflow; you define workflows as DAGs. Datastream is serverless change data capture: it streams changes from operational databases into BigQuery or Cloud Storage, with minimal latency.

Exam signals: "Run job B after job A, retry on failure" → Managed Airflow. "Replicate MySQL changes to BigQuery in near real time" → Datastream.

Trap: Nightly full exports from the production database when the requirement says near real time.

Pillar tie-in: Operational excellence.

Check: Which service streams database changes into BigQuery without servers to manage?
<details><summary>Answer</summary>Datastream.</details>

## Chunk 6: Governance and BI
Problem it solves: Data spread across lakes and warehouses needs a catalog, quality checks, and a place for business users to look at it.

Mental model: Knowledge Catalog (Dataplex) gives governance: discovery, lineage, profiling, data quality. Lakehouse (BigLake) lets BigQuery and open engines query Apache Iceberg data in Cloud Storage with shared governance; its APIs still use the BigLake name. Looker is the BI and semantic modeling layer on top.

Exam signals: "Central governance across data lake and warehouse" → Knowledge Catalog. "Dashboards with one definition of revenue" → Looker.

Trap: Copying lake data into BigQuery tables only to govern it.

Pillar tie-in: Security and operational excellence.

Check: What did Google rename Dataplex to?
<details><summary>Answer</summary>Knowledge Catalog.</details>

## Chunk 7: Batch vs streaming patterns
Problem it solves: A requirements paragraph must map to a pipeline shape.

Mental model:
- Streaming IoT: devices → Pub/Sub → Dataflow (windowing, enrichment) → BigQuery → Looker. Use Bigtable for low-latency lookups by device.
- Batch: files land in Cloud Storage → Dataflow or Managed Spark → BigQuery, scheduled by Managed Airflow.
- Simple ingest: Pub/Sub → BigQuery subscription.

Exam signals: "Real-time dashboard" → streaming. "Nightly reports" → batch.

Trap: A streaming design for data that's reported once a day.

Pillar tie-in: Cost.

Check: Sketch an ingestion-to-dashboard path for streaming IoT data.
<details><summary>Answer</summary>Devices → Pub/Sub → Dataflow → BigQuery (partitioned by time) → Looker. Add Bigtable if the app needs millisecond lookups per device.</details>
