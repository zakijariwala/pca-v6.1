# LAB B14-L01: Storage Transfer job between buckets

## Goal
Run a Storage Transfer Service job that copies objects from one bucket to another, then rerun it after adding files and see that it copies only what changed.

## What reading can't teach
The service agent authorization step, how a transfer job differs from a one-off copy, and what an incremental run reports.

## Cost ceiling
Under USD 0.05. A few small objects in two regional buckets. Transfers between Cloud Storage buckets can add storage operation and network charges at scale. [UNVERIFIED: check Storage Transfer Service pricing.] Verified: 2026-09-24.

## Time
30 minutes.

## Prerequisites
- A sandbox project with a budget alert (LAB B01-L01).
- Cloud Shell.

## Steps (Cloud Shell)
Replace PROJECT_ID. Keep real IDs out of any file you commit.

```bash
export PROJECT_ID=PROJECT_ID
export REGION=us-central1
export SRC=gs://$PROJECT_ID-b14-src
export DST=gs://$PROJECT_ID-b14-dst
gcloud config set project "$PROJECT_ID"
gcloud services enable storagetransfer.googleapis.com
```

1. Two buckets, and some files in the source.
   ```bash
   gcloud storage buckets create "$SRC" "$DST" --location="$REGION" --uniform-bucket-level-access
   for i in $(seq 1 20); do echo "record $i" > /tmp/r$i.txt; done
   gcloud storage cp /tmp/r*.txt "$SRC/archive/"
   ```
2. Grant the Storage Transfer Service agent what it needs.
   ```bash
   gcloud transfer authorize --add-missing
   ```
3. Create and run the job.
   ```bash
   gcloud transfer jobs create "$SRC/" "$DST/" --name=b14-archive-copy
   gcloud transfer operations list --job-names=b14-archive-copy --limit=1 \
     --format="yaml(metadata.status,metadata.counters)"
   ```
   Repeat the list command until the status is `SUCCESS`.
4. Add files to the source and run the same job again.
   ```bash
   for i in $(seq 21 25); do echo "record $i" > /tmp/r$i.txt; done
   gcloud storage cp /tmp/r2[1-5].txt "$SRC/archive/"
   gcloud transfer jobs run b14-archive-copy
   gcloud transfer operations list --job-names=b14-archive-copy --limit=1 \
     --format="yaml(metadata.status,metadata.counters)"
   ```
5. Compare the buckets.
   ```bash
   gcloud storage ls "$SRC/archive/" | wc -l
   gcloud storage ls "$DST/archive/" | wc -l
   ```

## Expected output
- Step 3: status `SUCCESS`; counters show 20 objects copied.
- Step 4: the second run copies 5 objects and skips the 20 that already match.
- Step 5: both counts are 25.

What to notice: a transfer job is a saved, repeatable definition you can schedule (`--schedule-repeats-every`) or rerun. For S3, Azure, or on-prem sources, the source argument and credentials change; the model stays the same.

## Teardown
```bash
gcloud transfer jobs delete b14-archive-copy
gcloud storage rm -r "$SRC" "$DST"
```

## Fallback: delete the project
```bash
gcloud projects delete "$PROJECT_ID"
```
