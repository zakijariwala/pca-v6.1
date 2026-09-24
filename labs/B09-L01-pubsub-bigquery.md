# LAB B09-L01: Pub/Sub to BigQuery, and a query cost check

## Goal
Stream messages from Pub/Sub into a partitioned BigQuery table with a BigQuery subscription (no pipeline code), then compare how many bytes a query scans with and without a partition filter.

## What reading can't teach
That no Dataflow job is needed for plain ingest, the permission the Pub/Sub service agent needs, and how dry runs report bytes before you pay.

## Cost ceiling
Under USD 0.10. A few messages, a tiny table, and dry runs, which are free. Verified: 2026-09-24.

## Time
30 to 45 minutes.

## Prerequisites
- A sandbox project with a budget alert (LAB B01-L01).
- Cloud Shell (it has `bq`).

## Steps (Cloud Shell)
Replace PROJECT_ID. Keep real IDs out of any file you commit.

```bash
export PROJECT_ID=PROJECT_ID
export PROJECT_NUMBER=$(gcloud projects describe "$PROJECT_ID" --format="value(projectNumber)")
export PUBSUB_SA=service-$PROJECT_NUMBER@gcp-sa-pubsub.iam.gserviceaccount.com
gcloud config set project "$PROJECT_ID"
gcloud services enable pubsub.googleapis.com bigquery.googleapis.com
```

1. Create a dataset and an ingestion-time partitioned table with one `data` column.
   ```bash
   bq mk --dataset --location=US "$PROJECT_ID:b09"
   bq mk --table --time_partitioning_type=DAY "$PROJECT_ID:b09.events" data:STRING
   ```
   Without a topic or table schema option, a BigQuery subscription writes each message to a column named `data`.
2. Let the Pub/Sub service agent write to the dataset.
   ```bash
   bq add-iam-policy-binding --member="serviceAccount:$PUBSUB_SA" \
     --role=roles/bigquery.dataEditor "$PROJECT_ID:b09"
   ```
   If your `bq` version lacks `add-iam-policy-binding`, grant `roles/bigquery.dataEditor` to the service agent at the project level instead.
3. Topic and BigQuery subscription.
   ```bash
   gcloud pubsub topics create b09-events
   gcloud pubsub subscriptions create b09-to-bq --topic=b09-events \
     --bigquery-table="$PROJECT_ID.b09.events"
   ```
4. Publish some messages.
   ```bash
   for i in $(seq 1 20); do
     gcloud pubsub topics publish b09-events --message="{\"device\":\"d$((i % 3))\",\"temp\":$((20 + i % 7))}"
   done
   ```
5. Query the table. Allow a minute for rows to land.
   ```bash
   bq query --use_legacy_sql=false \
     'SELECT JSON_VALUE(data, "$.device") AS device, AVG(CAST(JSON_VALUE(data, "$.temp") AS INT64)) AS avg_temp
      FROM b09.events GROUP BY device'
   ```
6. Dry-run two queries and compare bytes processed.
   ```bash
   bq query --use_legacy_sql=false --dry_run 'SELECT data FROM b09.events'
   bq query --use_legacy_sql=false --dry_run \
     'SELECT data FROM b09.events WHERE _PARTITIONTIME = TIMESTAMP_TRUNC(CURRENT_TIMESTAMP(), DAY)'
   ```

## Expected output
- Step 5: three rows, one per device, with an average temperature.
- Step 6: each dry run prints the bytes it would process. With one day of data both show the same small number. With many days of data, the partition filter scans only one day.

What to notice: nothing ran between Pub/Sub and BigQuery. The subscription did the writes.

## Teardown
```bash
gcloud pubsub subscriptions delete b09-to-bq
gcloud pubsub topics delete b09-events
bq rm -r -f "$PROJECT_ID:b09"
```

## Fallback: delete the project
```bash
gcloud projects delete "$PROJECT_ID"
```
