# LAB P01-L01: Every VM across two projects, as CSV

## Goal
List every VM in two projects as one CSV (project, name, zone, machine type, status) using only gcloud flags, with no grep, awk, or jq.

## What reading can't teach
How `--format` projections and transforms like `.basename()` behave on real output, and how `--filter` syntax fails when you get it wrong.

## Cost ceiling
Under USD 0.10 for two `e2-micro` VMs running under 30 minutes. [UNVERIFIED: check the pricing calculator for your region.] Verified: 2026-09-24.

## Time
30 minutes, including teardown.

## Prerequisites
- Two sandbox projects linked to a billing account with a budget alert (LAB B01-L01).
- Compute Engine API enabled in both.
- Cloud Shell.

## Steps (Cloud Shell)
Replace PROJECT_A and PROJECT_B. Keep real IDs out of any file you commit.

```bash
export PROJECT_A=PROJECT_A
export PROJECT_B=PROJECT_B
export ZONE=us-central1-a
for p in "$PROJECT_A" "$PROJECT_B"; do gcloud services enable compute.googleapis.com --project="$p"; done
```

1. Create one small VM in each project.
   ```bash
   gcloud compute instances create p01-vm-a --project="$PROJECT_A" --zone="$ZONE" --machine-type=e2-micro
   gcloud compute instances create p01-vm-b --project="$PROJECT_B" --zone="$ZONE" --machine-type=e2-micro
   ```
2. Stop one of them so the status column differs.
   ```bash
   gcloud compute instances stop p01-vm-b --project="$PROJECT_B" --zone="$ZONE"
   ```
3. Print one CSV header, then rows from both projects.
   ```bash
   echo "project,name,zone,machine_type,status"
   for p in "$PROJECT_A" "$PROJECT_B"; do
     gcloud compute instances list --project="$p" \
       --format="csv[no-heading](format('$p'),name,zone.basename(),machineType.basename(),status)"
   done
   ```
4. Filter to running VMs only.
   ```bash
   for p in "$PROJECT_A" "$PROJECT_B"; do
     gcloud compute instances list --project="$p" --filter="status=RUNNING" --format="value(name)"
   done
   ```
5. Try on your own: add a column for the internal IP. Hint: `networkInterfaces[0].networkIP`.

## Expected output
Step 3:
```
project,name,zone,machine_type,status
PROJECT_A,p01-vm-a,us-central1-a,e2-micro,RUNNING
PROJECT_B,p01-vm-b,us-central1-a,e2-micro,TERMINATED
```
Step 4 prints only `p01-vm-a`. A stopped VM shows `TERMINATED`, not `STOPPED`.

## Teardown
```bash
gcloud compute instances delete p01-vm-a --project="$PROJECT_A" --zone="$ZONE" --quiet
gcloud compute instances delete p01-vm-b --project="$PROJECT_B" --zone="$ZONE" --quiet
gcloud compute instances list --project="$PROJECT_A"
gcloud compute instances list --project="$PROJECT_B"
```
Both list commands should return nothing.

## Fallback: delete the project
```bash
gcloud projects delete "$PROJECT_A"
gcloud projects delete "$PROJECT_B"
```
