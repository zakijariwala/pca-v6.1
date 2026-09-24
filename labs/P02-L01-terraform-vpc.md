# LAB P02-L01: Rebuild the B03 network in Terraform with remote state

## Goal
Build the B03 network (custom VPC, two subnets, IAP SSH firewall rule, Cloud Router, Cloud NAT) with Terraform. Store state in a versioned Cloud Storage bucket. Authenticate by impersonating a Terraform service account, with no key.

## What reading can't teach
The init/plan/apply loop on real resources, what the state lock looks like, and what `plan` prints when someone changes a resource by hand.

## Cost ceiling
Under USD 0.30 for Cloud NAT running under 1 hour, plus pennies of storage. Cloud NAT bills by the hour. [UNVERIFIED: check the pricing calculator for your region.] Verified: 2026-09-24.

## Time
60 minutes, including teardown.

## Prerequisites
- A sandbox project with a budget alert (LAB B01-L01).
- Owner on the sandbox project, to set up the Terraform service account.
- LAB B03-L01 done, so you know what you're building.
- Cloud Shell. Check that Terraform is there with `terraform version`; install it if not.

## Steps (Cloud Shell)
Replace PROJECT_ID. Keep real IDs out of any file you commit.

```bash
export PROJECT_ID=PROJECT_ID
export REGION=us-central1
export TF_SA=terraform@$PROJECT_ID.iam.gserviceaccount.com
export ME=$(gcloud config get-value account)
export STATE_BUCKET=$PROJECT_ID-tfstate
gcloud config set project "$PROJECT_ID"
gcloud services enable compute.googleapis.com iamcredentials.googleapis.com
```

1. Create the Terraform service account with network roles, and let yourself impersonate it.
   ```bash
   gcloud iam service-accounts create terraform --display-name="Terraform"
   for role in roles/compute.networkAdmin roles/compute.securityAdmin; do
     gcloud projects add-iam-policy-binding "$PROJECT_ID" --member="serviceAccount:$TF_SA" --role="$role"
   done
   gcloud iam service-accounts add-iam-policy-binding "$TF_SA" \
     --member="user:$ME" --role=roles/iam.serviceAccountTokenCreator
   ```
2. Create the state bucket with versioning, and give the service account access to it.
   ```bash
   gcloud storage buckets create "gs://$STATE_BUCKET" --location="$REGION" --uniform-bucket-level-access
   gcloud storage buckets update "gs://$STATE_BUCKET" --versioning
   gcloud storage buckets add-iam-policy-binding "gs://$STATE_BUCKET" \
     --member="serviceAccount:$TF_SA" --role=roles/storage.objectAdmin
   ```
3. Write the configuration.
   ```bash
   mkdir -p ~/p02 && cd ~/p02
   cat > backend.tf <<EOF
   terraform {
     backend "gcs" {
       bucket                      = "$STATE_BUCKET"
       prefix                      = "p02/network"
       impersonate_service_account = "$TF_SA"
     }
   }
   EOF
   cat > main.tf <<'EOF'
   variable "project_id" { type = string }
   variable "region"     { type = string }
   variable "tf_sa"      { type = string }

   provider "google" {
     project                     = var.project_id
     region                      = var.region
     impersonate_service_account = var.tf_sa
   }

   resource "google_compute_network" "vpc" {
     name                    = "p02-vpc"
     auto_create_subnetworks = false
   }

   resource "google_compute_subnetwork" "a" {
     name                     = "p02-subnet-a"
     network                  = google_compute_network.vpc.id
     region                   = var.region
     ip_cidr_range            = "10.10.0.0/24"
     private_ip_google_access = true
   }

   resource "google_compute_subnetwork" "b" {
     name          = "p02-subnet-b"
     network       = google_compute_network.vpc.id
     region        = "us-east1"
     ip_cidr_range = "10.20.0.0/24"
   }

   resource "google_compute_firewall" "iap_ssh" {
     name          = "p02-allow-iap-ssh"
     network       = google_compute_network.vpc.id
     direction     = "INGRESS"
     source_ranges = ["35.235.240.0/20"]
     allow {
       protocol = "tcp"
       ports    = ["22"]
     }
   }

   resource "google_compute_router" "router" {
     name    = "p02-router"
     network = google_compute_network.vpc.id
     region  = var.region
   }

   resource "google_compute_router_nat" "nat" {
     name                               = "p02-nat"
     router                             = google_compute_router.router.name
     region                             = var.region
     nat_ip_allocate_option             = "AUTO_ONLY"
     source_subnetwork_ip_ranges_to_nat = "ALL_SUBNETWORKS_ALL_IP_RANGES"
   }

   output "network" { value = google_compute_network.vpc.name }
   EOF
   cat > terraform.tfvars <<EOF
   project_id = "$PROJECT_ID"
   region     = "$REGION"
   tf_sa      = "$TF_SA"
   EOF
   ```
4. Initialize, plan, apply.
   ```bash
   terraform init
   terraform plan -out=tfplan
   terraform apply tfplan
   ```
5. Check the state landed in the bucket.
   ```bash
   gcloud storage ls "gs://$STATE_BUCKET/p02/network/"
   ```
6. Create drift: change the firewall rule outside Terraform, then plan.
   ```bash
   gcloud compute firewall-rules update p02-allow-iap-ssh --rules=tcp:22,tcp:3389
   terraform plan
   ```
7. Apply to revert the drift.
   ```bash
   terraform apply -auto-approve
   ```

## Expected output
- Step 4: `Apply complete! Resources: 6 added, 0 changed, 0 destroyed.`
- Step 5: `default.tfstate` in the bucket.
- Step 6: the plan shows an in-place update on `google_compute_firewall.iap_ssh`, removing port 3389.
- Step 7: `1 changed`.

What to notice: `terraform.tfvars` and the state hold your project ID. Keep both out of Git.

## Teardown
```bash
cd ~/p02
terraform destroy -auto-approve
gcloud storage rm -r "gs://$STATE_BUCKET"
gcloud iam service-accounts delete "$TF_SA" --quiet
rm -rf ~/p02
```

Confirm nothing billable remains:
```bash
gcloud compute routers list
gcloud compute networks list
```

## Fallback: delete the project
```bash
gcloud projects delete "$PROJECT_ID"
```
