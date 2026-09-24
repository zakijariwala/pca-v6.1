# LAB B07-L01: Lifecycle rule and signed URL

## Goal
Add a lifecycle configuration to a bucket, then give time-limited access to one private object with a signed URL created through service account impersonation, with no key file.

## What reading can't teach
That lifecycle actions run later, not on save; what a signed URL looks like; and what an expired one returns.

## Cost ceiling
Under USD 0.01. One tiny object in a regional bucket. Verified: 2026-09-24.

## Time
30 minutes.

## Prerequisites
- A sandbox project with a budget alert (LAB B01-L01).
- LAB B02-L01 done, so you know impersonation.
- Cloud Shell.

## Steps (Cloud Shell)
Replace PROJECT_ID. Keep real IDs out of any file you commit.

```bash
export PROJECT_ID=PROJECT_ID
export REGION=us-central1
export BUCKET=gs://$PROJECT_ID-b07-lab
export SA=b07-signer@$PROJECT_ID.iam.gserviceaccount.com
export ME=$(gcloud config get-value account)
gcloud config set project "$PROJECT_ID"
gcloud services enable iamcredentials.googleapis.com
```

1. Create a private bucket and upload an object.
   ```bash
   gcloud storage buckets create "$BUCKET" --location="$REGION" --uniform-bucket-level-access
   echo "invoice 42" > /tmp/invoice.txt
   gcloud storage cp /tmp/invoice.txt "$BUCKET/invoice.txt"
   ```
2. Add a lifecycle configuration: move to Nearline at 30 days, delete at 365 days.
   ```bash
   cat > /tmp/lifecycle.json <<'EOF'
   {
     "rule": [
       {"action": {"type": "SetStorageClass", "storageClass": "NEARLINE"}, "condition": {"age": 30}},
       {"action": {"type": "Delete"}, "condition": {"age": 365}}
     ]
   }
   EOF
   gcloud storage buckets update "$BUCKET" --lifecycle-file=/tmp/lifecycle.json
   gcloud storage buckets describe "$BUCKET" --format="yaml(lifecycle_config)"
   ```
3. Create a signer service account with read access to the bucket, and let yourself impersonate it.
   ```bash
   gcloud iam service-accounts create b07-signer
   gcloud storage buckets add-iam-policy-binding "$BUCKET" \
     --member="serviceAccount:$SA" --role=roles/storage.objectViewer
   gcloud iam service-accounts add-iam-policy-binding "$SA" \
     --member="user:$ME" --role=roles/iam.serviceAccountTokenCreator
   ```
4. Confirm the object isn't public.
   ```bash
   curl -s -o /dev/null -w '%{http_code}\n' "https://storage.googleapis.com/$PROJECT_ID-b07-lab/invoice.txt"
   ```
5. Sign a URL valid for 2 minutes and fetch it.
   ```bash
   URL=$(gcloud storage sign-url "$BUCKET/invoice.txt" --duration=2m \
     --impersonate-service-account="$SA" --format="value(signed_url)")
   curl -s "$URL"
   ```
6. Wait 3 minutes, then fetch again.
   ```bash
   curl -s -o /dev/null -w '%{http_code}\n' "$URL"
   ```

## Expected output
- Step 2: the lifecycle config lists both rules. The object stays in Standard: rules act on objects 30 days old, and config changes can take up to 24 hours to take effect.
- Step 4: `403`.
- Step 5: `invoice 42`.
- Step 6: an error code (`400` for an expired signature).

If `--format="value(signed_url)"` prints nothing, drop the `--format` flag and copy the URL from the table. IAM changes can take a minute or two to apply.

## Teardown
```bash
gcloud storage rm -r "$BUCKET"
gcloud iam service-accounts delete "$SA" --quiet
```

## Fallback: delete the project
```bash
gcloud projects delete "$PROJECT_ID"
```
