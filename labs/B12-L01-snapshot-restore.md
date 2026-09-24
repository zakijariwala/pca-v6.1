# LAB B12-L01: Snapshot schedule and restore to a new VM

## Goal
Attach a daily snapshot schedule to a VM's disk, take a snapshot on demand, then restore it as a new VM in a different region and confirm your data is there.

## What reading can't teach
How long a restore takes, that snapshots cross regions, and that the restored VM is a separate machine you must reconfigure.

## Cost ceiling
Under USD 0.50 for two `e2-micro` VMs under 1 hour and a small snapshot. Snapshots bill for storage until deleted. [UNVERIFIED: check the pricing calculator for your region.] Verified: 2026-09-24.

## Time
45 minutes, including teardown.

## Prerequisites
- A sandbox project with a budget alert (LAB B01-L01).
- A `default` VPC network, or add `--network`/`--subnet` flags.
- Cloud Shell.

## Steps (Cloud Shell)
Replace PROJECT_ID. Keep real IDs out of any file you commit.

```bash
export PROJECT_ID=PROJECT_ID
export REGION=us-central1
export ZONE=us-central1-a
export DR_ZONE=us-east1-b
gcloud config set project "$PROJECT_ID"
gcloud services enable compute.googleapis.com
```

1. A VM with a file you'll want back.
   ```bash
   gcloud compute instances create b12-src --zone="$ZONE" --machine-type=e2-micro \
     --image-family=debian-12 --image-project=debian-cloud \
     --metadata=startup-script='echo "order ledger v1 $(date)" > /var/ledger.txt'
   ```
2. A daily snapshot schedule, attached to the boot disk.
   ```bash
   gcloud compute resource-policies create snapshot-schedule b12-daily --region="$REGION" \
     --daily-schedule --start-time=04:00 --max-retention-days=7
   gcloud compute disks add-resource-policies b12-src --zone="$ZONE" --resource-policies=b12-daily
   ```
3. Don't wait for 04:00. Take a snapshot now.
   ```bash
   gcloud compute snapshots create b12-snap-now --source-disk=b12-src --source-disk-zone="$ZONE"
   gcloud compute snapshots describe b12-snap-now --format="value(status,storageLocations)"
   ```
4. Restore into another region: a new disk from the snapshot, then a VM on it.
   ```bash
   time gcloud compute disks create b12-restored --zone="$DR_ZONE" --source-snapshot=b12-snap-now
   gcloud compute instances create b12-dr --zone="$DR_ZONE" --machine-type=e2-micro \
     --disk=name=b12-restored,boot=yes,auto-delete=yes
   ```
5. Check the file.
   ```bash
   gcloud compute ssh b12-dr --zone="$DR_ZONE" --command="cat /var/ledger.txt"
   ```

## Expected output
- Step 3: `READY` and a multi-region storage location such as `us`.
- Step 4: the disk create reports its time; note it against your RTO.
- Step 5: the `order ledger v1` line with the original timestamp.

What to notice: the restored VM has a new name, internal IP, and zone. Anything that pointed at `b12-src` must be updated. That's part of your RTO.

If SSH fails, allow `tcp:22` from `35.235.240.0/20` on the network and add `--tunnel-through-iap`.

## Teardown
```bash
gcloud compute instances delete b12-dr --zone="$DR_ZONE" --quiet
gcloud compute instances delete b12-src --zone="$ZONE" --quiet
gcloud compute snapshots delete b12-snap-now --quiet
gcloud compute resource-policies delete b12-daily --region="$REGION" --quiet
gcloud compute snapshots list
```
If the schedule created snapshots before you deleted it, delete those too.

## Fallback: delete the project
```bash
gcloud projects delete "$PROJECT_ID"
```
