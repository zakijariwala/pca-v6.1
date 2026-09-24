# LAB B03-L01: Private VM reaching the internet through Cloud NAT

## Goal
Build a custom VPC with two subnets, SSH into a VM that has no external IP through IAP, watch its outbound request fail, then add Cloud NAT and watch it succeed.

## What reading can't teach
That a route to the internet isn't enough, how the failure looks from inside the VM (a timeout, not an error), and that NAT takes effect with no change to the VM.

## Cost ceiling
Under USD 0.50 for one `e2-micro` VM and one Cloud NAT gateway running under 1 hour. Cloud NAT bills by the hour. [UNVERIFIED: check the pricing calculator for your region.] Verified: 2026-09-24.

## Time
45 minutes, including teardown.

## Prerequisites
- A sandbox project with a budget alert (LAB B01-L01).
- Owner or Network Admin plus Compute Instance Admin on the project.
- Cloud Shell.

## Steps (Cloud Shell)
Replace PROJECT_ID. Keep real IDs out of any file you commit.

```bash
export PROJECT_ID=PROJECT_ID
export REGION=us-central1
export REGION2=us-east1
export ZONE=us-central1-a
gcloud config set project "$PROJECT_ID"
gcloud services enable compute.googleapis.com iap.googleapis.com
```

1. Create a custom mode VPC and two subnets in different regions.
   ```bash
   gcloud compute networks create b03-vpc --subnet-mode=custom
   gcloud compute networks subnets create b03-subnet-a --network=b03-vpc --region="$REGION" --range=10.10.0.0/24
   gcloud compute networks subnets create b03-subnet-b --network=b03-vpc --region="$REGION2" --range=10.20.0.0/24
   ```
2. Allow SSH only from IAP's TCP forwarding range.
   ```bash
   gcloud compute firewall-rules create b03-allow-iap-ssh --network=b03-vpc \
     --direction=INGRESS --action=allow --rules=tcp:22 --source-ranges=35.235.240.0/20
   ```
3. Create a VM with no external IP.
   ```bash
   gcloud compute instances create b03-vm --zone="$ZONE" --machine-type=e2-micro \
     --subnet=b03-subnet-a --no-address
   ```
4. Try to reach the internet from the VM. Expect a timeout.
   ```bash
   gcloud compute ssh b03-vm --zone="$ZONE" --tunnel-through-iap \
     --command="curl -sS -m 5 -o /dev/null -w '%{http_code}\n' https://example.com || echo FAILED"
   ```
5. Add a Cloud Router and Cloud NAT in the VM's region.
   ```bash
   gcloud compute routers create b03-router --network=b03-vpc --region="$REGION"
   gcloud compute routers nats create b03-nat --router=b03-router --region="$REGION" \
     --auto-allocate-nat-external-ips --nat-all-subnet-ip-ranges
   ```
6. Repeat step 4. Wait a minute first if it still fails.

## Expected output
- Step 4 prints a curl timeout, then `FAILED`.
- Step 6 prints `200`.

What to notice: you changed nothing on the VM. The VM still has no external IP and still accepts no inbound traffic from the internet.

## Teardown
Delete in reverse order of creation.

```bash
gcloud compute instances delete b03-vm --zone="$ZONE" --quiet
gcloud compute routers nats delete b03-nat --router=b03-router --region="$REGION" --quiet
gcloud compute routers delete b03-router --region="$REGION" --quiet
gcloud compute firewall-rules delete b03-allow-iap-ssh --quiet
gcloud compute networks subnets delete b03-subnet-a --region="$REGION" --quiet
gcloud compute networks subnets delete b03-subnet-b --region="$REGION2" --quiet
gcloud compute networks delete b03-vpc --quiet
```

Confirm nothing billable remains:
```bash
gcloud compute instances list
gcloud compute routers list
```

## Fallback: delete the project
```bash
gcloud projects delete "$PROJECT_ID"
```
