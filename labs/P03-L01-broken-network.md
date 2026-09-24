# LAB P03-L01: Find three faults in a broken network

## Goal
Deploy a Terraform network with three planted faults. Using only diagnostic tools (Connectivity Tests, firewall rules logging, NAT status, `gcloud describe`), find and fix all three. You paste `main.tf` below, so the faults are visible in it. Don't read it for answers: treat it as a black box someone else wrote, and find each fault with the tools.

## What reading can't teach
Diagnosis order under uncertainty, and what each tool shows when the fault is real.

## Cost ceiling
Under USD 0.50 for one `e2-micro` VM and one Cloud NAT gateway running under 90 minutes. [UNVERIFIED: check the pricing calculator for your region.] Verified: 2026-09-24.

## Time
60 to 90 minutes, including teardown.

## Prerequisites
- A sandbox project with a budget alert (LAB B01-L01).
- LAB P02-L01 done, so the Terraform loop is familiar. This config runs as you, so you need Owner or Compute Admin on the sandbox project.
- Cloud Shell with Terraform.

## The symptoms
After deploy, the app owner reports:
1. "I can't SSH to `p03-vm` through IAP."
2. "Once I'm on the VM, `curl https://example.com` times out."
3. "Even after that's fixed, HTTPS to anything outside still fails, but HTTP works."

## Steps (Cloud Shell)
Replace PROJECT_ID. Keep real IDs out of any file you commit.

```bash
export PROJECT_ID=PROJECT_ID
export REGION=us-central1
export ZONE=us-central1-a
gcloud config set project "$PROJECT_ID"
gcloud services enable compute.googleapis.com iap.googleapis.com networkmanagement.googleapis.com
mkdir -p ~/p03 && cd ~/p03
cat > main.tf <<'EOF'
variable "project_id" { type = string }
variable "region"     { default = "us-central1" }
variable "zone"       { default = "us-central1-a" }

provider "google" {
  project = var.project_id
  region  = var.region
}

resource "google_compute_network" "vpc" {
  name                    = "p03-vpc"
  auto_create_subnetworks = false
}

resource "google_compute_subnetwork" "app" {
  name          = "p03-app"
  network       = google_compute_network.vpc.id
  ip_cidr_range = "10.30.0.0/24"
}

resource "google_compute_subnetwork" "batch" {
  name          = "p03-batch"
  network       = google_compute_network.vpc.id
  ip_cidr_range = "10.31.0.0/24"
}

resource "google_compute_firewall" "ssh" {
  name          = "p03-allow-ssh"
  network       = google_compute_network.vpc.id
  source_ranges = ["35.235.0.0/20"]
  allow {
    protocol = "tcp"
    ports    = ["22"]
  }
}

resource "google_compute_firewall" "egress_lockdown" {
  name               = "p03-legacy-egress"
  network            = google_compute_network.vpc.id
  direction          = "EGRESS"
  priority           = 900
  destination_ranges = ["0.0.0.0/0"]
  deny {
    protocol = "tcp"
    ports    = ["443"]
  }
}

resource "google_compute_router" "r" {
  name    = "p03-router"
  network = google_compute_network.vpc.id
}

resource "google_compute_router_nat" "nat" {
  name                               = "p03-nat"
  router                             = google_compute_router.r.name
  nat_ip_allocate_option             = "AUTO_ONLY"
  source_subnetwork_ip_ranges_to_nat = "LIST_OF_SUBNETWORKS"
  subnetwork {
    name                    = google_compute_subnetwork.batch.id
    source_ip_ranges_to_nat = ["ALL_IP_RANGES"]
  }
}

resource "google_compute_instance" "vm" {
  name         = "p03-vm"
  zone         = var.zone
  machine_type = "e2-micro"
  boot_disk {
    initialize_params { image = "debian-cloud/debian-12" }
  }
  network_interface {
    subnetwork = google_compute_subnetwork.app.id
  }
}
EOF
terraform init
terraform apply -auto-approve -var="project_id=$PROJECT_ID"
```

Now diagnose. Tools you may use:

```bash
# Simulate a path and name the blocking hop
gcloud network-management connectivity-tests create p03-test \
  --source-instance="projects/$PROJECT_ID/zones/$ZONE/instances/p03-vm" \
  --destination-ip-address=8.8.8.8 --destination-port=443 --protocol=TCP
gcloud network-management connectivity-tests describe p03-test

# Inspect firewall rules and NAT
gcloud compute firewall-rules list --filter="network:p03-vpc" \
  --format="table(name,direction,priority,sourceRanges.list(),destinationRanges.list())"
gcloud compute firewall-rules describe RULE_NAME
gcloud compute routers nats describe p03-nat --router=p03-router --region="$REGION"

# Turn on logging for a suspect rule
gcloud compute firewall-rules update RULE_NAME --enable-logging
```

Fix each fault in `main.tf`, then `terraform apply` again. Test with:
```bash
gcloud compute ssh p03-vm --zone="$ZONE" --tunnel-through-iap \
  --command="curl -sS -m 5 -o /dev/null -w '%{http_code}\n' https://example.com"
```

## Expected output
After all three fixes, the SSH command prints `200`.

<details><summary>Fault 1</summary>

The SSH rule allows `35.235.0.0/20`. IAP TCP forwarding comes from `35.235.240.0/20`. The implied deny ingress rule drops the IAP connection. Fix: set `source_ranges = ["35.235.240.0/20"]`.
</details>

<details><summary>Fault 2</summary>

Cloud NAT covers only `p03-batch`. The VM sits in `p03-app`, has no external IP, and so has no path to the internet. Fix: add `p03-app` to the NAT's subnetwork list, or use `ALL_SUBNETWORKS_ALL_IP_RANGES`.
</details>

<details><summary>Fault 3</summary>

`p03-legacy-egress` denies TCP 443 to `0.0.0.0/0` at priority 900. That beats the implied allow egress rule at 65535. HTTP on port 80 still works, which is the clue. Fix: remove the rule, or scope it to the destinations it was meant to block.
</details>

## Teardown
```bash
gcloud network-management connectivity-tests delete p03-test --quiet
cd ~/p03 && terraform destroy -auto-approve -var="project_id=$PROJECT_ID"
rm -rf ~/p03
gcloud compute instances list
gcloud compute routers list
```

## Fallback: delete the project
```bash
gcloud projects delete "$PROJECT_ID"
```
