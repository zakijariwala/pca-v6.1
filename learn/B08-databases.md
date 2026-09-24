---
block: B08
title: Databases
pillars: [reliability, performance, cost]
exam_guide_refs: ["1.3", "2.2", "5.2"]
last_verified: 2026-09-24
sources:
  - https://docs.cloud.google.com/sql/docs/mysql/high-availability
  - https://docs.cloud.google.com/sql/docs/mysql/replication
  - https://docs.cloud.google.com/alloydb/docs/overview
  - https://docs.cloud.google.com/spanner/docs/instance-configurations
  - https://docs.cloud.google.com/spanner/sla
  - https://docs.cloud.google.com/bigtable/docs/overview
  - https://docs.cloud.google.com/firestore/native/docs/overview
  - https://docs.cloud.google.com/memorystore/docs/redis/memorystore-for-redis-overview
  - https://docs.cloud.google.com/memorystore/docs/valkey/product-overview
unverified: []
---
# B08 Databases

## Chunk 1: The selection tree
Problem it solves: A one-line workload description must map to one database in seconds.

Mental model: Ask in order.
1. Analytics over large datasets, SQL, not serving an app? → BigQuery (B09).
2. Relational, SQL, transactions?
   - Global scale or 99.999% availability → Spanner.
   - PostgreSQL with heavy analytics on live data → AlloyDB.
   - Standard MySQL, PostgreSQL, or SQL Server → Cloud SQL.
3. Non-relational?
   - Massive key-value or time series, single-digit-ms latency → Bigtable.
   - Documents for web and mobile apps, serverless → Firestore.
   - Cache or session store → Memorystore.

Exam signals: "Global inventory with strong consistency" → Spanner. "IoT time series at millions of writes per second" → Bigtable. "Lift an existing MySQL app" → Cloud SQL.

Trap: Spanner for a small regional app. It works, but costs more than Cloud SQL for no gain.

Pillar tie-in: Cost and performance.

Check: A mobile app stores user profiles as JSON-like documents and syncs offline. Which database?
<details><summary>Answer</summary>Firestore.</details>

## Chunk 2: Cloud SQL HA and replicas
Problem it solves: A managed MySQL, PostgreSQL, or SQL Server instance must survive a zone failure and scale reads.

Mental model: An HA configuration pairs a primary and a standby in two zones of one region. Writes replicate synchronously to disks in both zones before commit. On failure, the standby becomes primary and clients reconnect to the same address. Read replicas scale reads and replicate asynchronously; cross-region replicas give DR, and you promote one to primary in a disaster.

| Need | Feature |
|---|---|
| Survive a zone failure | HA (regional) configuration |
| Offload reads | Read replicas |
| Survive a region failure | Cross-region replica, promoted on disaster |

Exam signals: "Automatic failover within a region" → HA. "Reporting queries slow down the app" → read replica.

Trap: A read replica as the HA answer. Replicas don't fail over on their own.

Pillar tie-in: Reliability.

Check: Does Cloud SQL HA replicate synchronously or asynchronously?
<details><summary>Answer</summary>Synchronously, to disks in both zones before a write commits.</details>

## Chunk 3: AlloyDB
Problem it solves: A PostgreSQL workload needs more performance and analytics on live data than standard Cloud SQL.

Mental model: AlloyDB for PostgreSQL is fully managed and PostgreSQL-compatible. Its columnar engine speeds analytical queries on transactional data (HTAP). Standard tools such as psql and pgAdmin work.

Exam signals: "PostgreSQL", "real-time analytics on operational data", "higher throughput than Cloud SQL."

Trap: AlloyDB for a MySQL workload. It speaks PostgreSQL only.

Pillar tie-in: Performance.

Check: What feature lets AlloyDB run analytics on live transactional data?
<details><summary>Answer</summary>Its columnar engine, which accelerates analytical queries (HTAP).</details>

## Chunk 4: Spanner
Problem it solves: Relational SQL with strong consistency at global scale, with no sharding by hand.

Mental model: Spanner scales horizontally with strong consistency. Instance configurations are regional, dual-region, or multi-region. The SLA is 99.999% for multi-region and dual-region configurations and 99.99% for regional.

Exam signals: "Global users", "strongly consistent transactions", "99.999%", "no downtime for schema changes."

Trap: Bigtable for global financial transactions. Bigtable lacks multi-row SQL transactions.

Pillar tie-in: Reliability.

Check: Which Spanner configurations carry the 99.999% SLA?
<details><summary>Answer</summary>Multi-region and dual-region. Regional carries 99.99%.</details>

## Chunk 5: Bigtable
Problem it solves: Terabytes to petabytes of key-value data with high write throughput and low latency.

Mental model: A sorted key-value map: rows keyed by one row key, with many columns. Reads and writes are fast when you know the key. It supports the Apache HBase API. Throughput scales with nodes. Row key design decides performance; hot keys (like timestamps as a prefix) concentrate load on one node.

Exam signals: "Time series", "IoT telemetry", "ad tech", "HBase migration", "single-digit millisecond latency at scale."

Trap: Timestamp-first row keys. All new writes land on one tablet.

Pillar tie-in: Performance.

Check: An existing HBase cluster must move to managed Google Cloud. Which service?
<details><summary>Answer</summary>Bigtable, which supports the HBase API.</details>

## Chunk 6: Firestore and Memorystore
Problem it solves: App data that isn't relational, and hot data that must answer in microseconds.

Mental model: Firestore is a serverless document database with ACID transactions and automatic multi-region replication; its SLA is 99.99% regional and 99.999% multi-region. Memorystore is managed in-memory storage: Memorystore for Redis, for Redis Cluster, for Valkey, and for Memcached. Use it as a cache or session store, not the system of record.

Exam signals: "Session state for a web tier", "cache hot product data" → Memorystore. "Serverless backend for a mobile app" → Firestore.

Trap: Memorystore as the only copy of data that must survive restarts.

Pillar tie-in: Performance.

Check: What's Memorystore's role in most designs?
<details><summary>Answer</summary>A cache or session store in front of a durable database.</details>

## Chunk 7: Emulators for local testing
Problem it solves: Developers need to test against Bigtable, Spanner, Firestore, and Pub/Sub without cloud cost or shared state.

Mental model: The Google Cloud CLI ships local emulators for Bigtable, Spanner, Firestore (and Datastore), and Pub/Sub. Start one with `gcloud emulators ... start` and point the client library at it with an environment variable.

Exam signals: "Unit tests without calling production services."

Trap: A shared dev instance as the only test target. Tests collide and cost money.

Pillar tie-in: Operational excellence and cost.

Check: Name two services with local emulators in the Google Cloud CLI.
<details><summary>Answer</summary>Any two of: Bigtable, Spanner, Firestore (or Datastore), Pub/Sub.</details>
