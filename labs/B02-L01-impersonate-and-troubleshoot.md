# LAB B02-L01: Impersonate a service account, then prove a denial

## Goal
Create a service account with read-only storage access, impersonate it from Cloud Shell, watch a write fail, and have Policy Troubleshooter explain why.

## What reading can't teach
The exact error text for a missing permission, the token creator step people forget, and how Policy Troubleshooter lays out its verdict.

## Cost ceiling
Under USD 0.01. One empty bucket and a few API calls. Verified: 2026-09-24.

## Time
30 minutes.

## Prerequisites
- A sandbox project with a budget alert (LAB B01-L01).
- Owner or equivalent on the sandbox project.
- Cloud Shell.

## Steps (Cloud Shell)
Replace PROJECT_ID. Keep real IDs out of any file you commit.

```bash
export PROJECT_ID=PROJECT_ID
export REGION=us-central1
export SA=b02-reader@$PROJECT_ID.iam.gserviceaccount.com
export ME=$(gcloud config get-value account)
export BUCKET=gs://$PROJECT_ID-b02-lab
gcloud config set project "$PROJECT_ID"
gcloud services enable iam.googleapis.com iamcredentials.googleapis.com policytroubleshooter.googleapis.com
```

1. Create a bucket and a service account with read-only access to it.
   ```bash
   gcloud storage buckets create "$BUCKET" --location="$REGION" --uniform-bucket-level-access
   gcloud iam service-accounts create b02-reader --display-name="B02 reader"
   gcloud storage buckets add-iam-policy-binding "$BUCKET" \
     --member="serviceAccount:$SA" --role=roles/storage.objectViewer
   ```
2. Let yourself impersonate it.
   ```bash
   gcloud iam service-accounts add-iam-policy-binding "$SA" \
     --member="user:$ME" --role=roles/iam.serviceAccountTokenCreator
   ```
   IAM changes can take a minute or two to apply.
3. Read as the service account. This should work.
   ```bash
   gcloud storage ls "$BUCKET" --impersonate-service-account="$SA"
   ```
4. Write as the service account. This should fail.
   ```bash
   echo hello > /tmp/b02.txt
   gcloud storage cp /tmp/b02.txt "$BUCKET/" --impersonate-service-account="$SA"
   ```
5. Ask Policy Troubleshooter why.
   ```bash
   gcloud policy-intelligence troubleshoot-policy iam \
     "//storage.googleapis.com/projects/_/buckets/$PROJECT_ID-b02-lab" \
     --principal-email="$SA" --permission=storage.objects.create
   ```

## Expected output
- Step 3 succeeds with an empty listing.
- Step 4 fails with a 403 naming `storage.objects.create`.
- Step 5 reports access not granted and lists the bucket's allow policy, showing `roles/storage.objectViewer` doesn't contain `storage.objects.create`.

What to notice: impersonation never wrote a key file. Check `ls ~/.config/gcloud` if you want proof.

## Teardown
```bash
gcloud storage rm -r "$BUCKET"
gcloud iam service-accounts delete "$SA" --quiet
```

## Fallback: delete the project
```bash
gcloud projects delete "$PROJECT_ID"
```
