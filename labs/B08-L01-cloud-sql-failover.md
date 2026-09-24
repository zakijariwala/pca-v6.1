# LAB B08-L01: Cloud SQL with private IP, and a failover test

## Goal
Create a highly available (regional) Cloud SQL for PostgreSQL instance with only a private IP. Connect from a VM in the same VPC, trigger a failover, and watch the connection drop and recover on the same IP.

## What reading can't teach
The private services access setup, how long a failover takes, and what your client sees during it.

## Cost ceiling
Under USD 2.00 for a small regional instance and one `e2-micro` VM, under 90 minutes. Cloud SQL bills by the hour and HA about doubles instance cost. [UNVERIFIED: check Cloud SQL pricing for your region and edition.] Verified: 2026-09-24.

## Time
60 to 90 minutes, including teardown. Instance creation takes several minutes.

## Prerequisites
- A sandbox project with a budget alert (LAB B01-L01).
- A `default` VPC network, or substitute your own network name.
- Cloud Shell.

## Steps (Cloud Shell)
Replace PROJECT_ID. Keep real IDs out of any file you commit.

```bash
export PROJECT_ID=PROJECT_ID
export REGION=us-central1
export ZONE=us-central1-a
export NET=default
export DB_PASS=$(openssl rand -base64 18)
gcloud config set project "$PROJECT_ID"
gcloud services enable sqladmin.googleapis.com servicenetworking.googleapis.com compute.googleapis.com iap.googleapis.com
```

1. Set up private services access: reserve a range and peer it with Google's service network.
   ```bash
   gcloud compute addresses create google-managed-services-$NET --global \
     --purpose=VPC_PEERING --prefix-length=16 --network="$NET"
   gcloud services vpc-peerings connect --service=servicenetworking.googleapis.com \
     --ranges=google-managed-services-$NET --network="$NET"
   ```
2. Create a regional (HA) instance with no public IP.
   ```bash
   gcloud sql instances create b08-pg --database-version=POSTGRES_16 \
     --edition=ENTERPRISE --tier=db-custom-1-3840 --region="$REGION" \
     --availability-type=REGIONAL --network="$NET" --no-assign-ip \
     --root-password="$DB_PASS"
   export DB_IP=$(gcloud sql instances describe b08-pg --format="value(ipAddresses[0].ipAddress)")
   echo "$DB_IP"
   ```
3. A client VM in the same VPC, reached through IAP.
   ```bash
   gcloud compute firewall-rules create b08-allow-iap --network="$NET" --direction=INGRESS \
     --rules=tcp:22 --source-ranges=35.235.240.0/20
   gcloud compute instances create b08-client --zone="$ZONE" --machine-type=e2-micro \
     --image-family=debian-12 --image-project=debian-cloud \
     --metadata=startup-script='apt-get update && apt-get install -y postgresql-client'
   ```
4. In a second Cloud Shell tab, run a query every 2 seconds from the VM. Leave it running.
   ```bash
   gcloud compute ssh b08-client --zone="$ZONE" --tunnel-through-iap --command="
     while true; do
       PGPASSWORD='$DB_PASS' psql -h $DB_IP -U postgres -tAc 'select now(), inet_server_addr()' || echo DOWN;
       sleep 2; done"
   ```
5. In the first tab, trigger a failover.
   ```bash
   gcloud sql instances failover b08-pg
   gcloud sql instances describe b08-pg --format="value(gceZone,secondaryGceZone)"
   ```

## Expected output
- Step 2: an instance with one private IP and no public IP.
- Step 4: a timestamp and the instance IP every 2 seconds.
- Step 5: step 4's loop prints `DOWN` for a short stretch, then resumes with the same IP. After failover, `gceZone` and `secondaryGceZone` have swapped.

What to notice: clients reconnect to the same IP. Your app still needs retry logic for the gap.

## Teardown
Delete in reverse order of creation.

```bash
gcloud compute instances delete b08-client --zone="$ZONE" --quiet
gcloud compute firewall-rules delete b08-allow-iap --quiet
gcloud sql instances delete b08-pg --quiet
gcloud services vpc-peerings delete --service=servicenetworking.googleapis.com --network="$NET" --quiet
gcloud compute addresses delete google-managed-services-$NET --global --quiet
```
The peering delete can fail for a while after the instance is deleted. Retry later if so.

Confirm nothing billable remains:
```bash
gcloud sql instances list
gcloud compute instances list
```

## Fallback: delete the project
```bash
gcloud projects delete "$PROJECT_ID"
```
