# LAB B13-L01: Cloud Build pipeline that runs terraform plan

## Goal
Run `terraform plan` in Cloud Build as a dedicated, least-privilege service account, first by hand, then (optional) on every commit to a GitHub repo.

## What reading can't teach
What a build step container is, where build logs go with a user-specified service account, and which roles a read-only plan needs.

## Cost ceiling
Under USD 0.10. A few short builds and a tiny state bucket. [UNVERIFIED: check Cloud Build pricing and free tier.] Verified: 2026-09-24.

## Time
45 minutes; 30 more for the optional trigger.

## Prerequisites
- A sandbox project with a budget alert (LAB B01-L01).
- LAB P02-L01 done.
- Cloud Shell.
- Optional part: a GitHub account and an empty repo you control.

## Steps (Cloud Shell)
Replace PROJECT_ID. Keep real IDs out of any file you commit.

```bash
export PROJECT_ID=PROJECT_ID
export REGION=us-central1
export BUILD_SA=b13-plan@$PROJECT_ID.iam.gserviceaccount.com
export STATE_BUCKET=$PROJECT_ID-b13-tfstate
gcloud config set project "$PROJECT_ID"
gcloud services enable cloudbuild.googleapis.com compute.googleapis.com
```

1. A build service account that can read the project and use the state bucket, nothing more.
   ```bash
   gcloud iam service-accounts create b13-plan
   gcloud projects add-iam-policy-binding "$PROJECT_ID" --member="serviceAccount:$BUILD_SA" --role=roles/viewer
   gcloud projects add-iam-policy-binding "$PROJECT_ID" --member="serviceAccount:$BUILD_SA" --role=roles/logging.logWriter
   gcloud storage buckets create "gs://$STATE_BUCKET" --location="$REGION" --uniform-bucket-level-access
   gcloud storage buckets update "gs://$STATE_BUCKET" --versioning
   gcloud storage buckets add-iam-policy-binding "gs://$STATE_BUCKET" \
     --member="serviceAccount:$BUILD_SA" --role=roles/storage.objectAdmin
   ```
   Viewer is broad for a real pipeline; it keeps the lab short. In production, grant only the read roles your resources need.
2. A small Terraform config and a build file.
   ```bash
   mkdir -p ~/b13 && cd ~/b13
   cat > main.tf <<EOF
   terraform {
     backend "gcs" {
       bucket = "$STATE_BUCKET"
       prefix = "b13"
     }
   }
   provider "google" {
     project = "$PROJECT_ID"
     region  = "$REGION"
   }
   resource "google_compute_network" "vpc" {
     name                    = "b13-vpc"
     auto_create_subnetworks = false
   }
   EOF
   cat > cloudbuild.yaml <<'EOF'
   steps:
   - id: init
     name: hashicorp/terraform:1.9
     args: ["init", "-input=false"]
   - id: plan
     name: hashicorp/terraform:1.9
     args: ["plan", "-input=false"]
   options:
     logging: CLOUD_LOGGING_ONLY
   EOF
   ```
3. Run the build as the service account.
   ```bash
   gcloud builds submit --config=cloudbuild.yaml \
     --service-account="projects/$PROJECT_ID/serviceAccounts/$BUILD_SA" .
   ```
4. Find the build and its logs.
   ```bash
   gcloud builds list --limit=1
   gcloud builds log "$(gcloud builds list --limit=1 --format='value(id)')"
   ```
5. Optional, on commit: push `main.tf` and `cloudbuild.yaml` to your GitHub repo. In the console, open Cloud Build → Repositories, connect GitHub, link the repo, then create a trigger on push to `main` that uses `cloudbuild.yaml` and the `b13-plan` service account. Push a change to `main.tf` and watch a build start. Keep the project ID out of the committed files: move it to a trigger substitution such as `_PROJECT_ID`.

If the build can't read its uploaded source, grant the service account `roles/storage.objectViewer` on the `gs://${PROJECT_ID}_cloudbuild` bucket and run step 3 again. The `hashicorp/terraform` image comes from Docker Hub.

## Expected output
- Step 3: two steps, `init` then `plan`. The plan ends with `Plan: 1 to add, 0 to change, 0 to destroy.`
- Step 4: the same output, read back from Cloud Logging.

What to notice: the plan ran with read-only project access. It can show changes but can't make them. Apply belongs in a separate, gated step with a different service account.

## Teardown
Nothing was applied, so no network exists.

```bash
gcloud storage rm -r "gs://$STATE_BUCKET"
gcloud projects remove-iam-policy-binding "$PROJECT_ID" --member="serviceAccount:$BUILD_SA" --role=roles/viewer
gcloud projects remove-iam-policy-binding "$PROJECT_ID" --member="serviceAccount:$BUILD_SA" --role=roles/logging.logWriter
gcloud iam service-accounts delete "$BUILD_SA" --quiet
rm -rf ~/b13
```
If you made a trigger, delete it and disconnect the repo in the console.

## Fallback: delete the project
```bash
gcloud projects delete "$PROJECT_ID"
```
