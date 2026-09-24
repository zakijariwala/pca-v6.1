# LAB B11-L01: CMEK on a bucket, then disable the key

## Goal
Protect a bucket with a customer-managed encryption key (CMEK) in Cloud KMS, read an object, disable the key version, watch reads fail, then re-enable it.

## What reading can't teach
The Cloud Storage service agent permission step, the exact error when a key is disabled, and that "you control the key" means you can cut off your own access.

## Cost ceiling
Under USD 0.20. One software key version bills monthly while it exists and stays billed until destruction completes. Key operations cost fractions of a cent. [UNVERIFIED: check Cloud KMS pricing.] Verified: 2026-09-24.

## Time
30 minutes.

## Prerequisites
- A sandbox project with a budget alert (LAB B01-L01).
- Owner, or Cloud KMS Admin plus Storage Admin.
- Cloud Shell.

## Steps (Cloud Shell)
Replace PROJECT_ID. Keep real IDs out of any file you commit.

```bash
export PROJECT_ID=PROJECT_ID
export REGION=us-central1
export BUCKET=gs://$PROJECT_ID-b11-lab
export KEY=projects/$PROJECT_ID/locations/$REGION/keyRings/b11-ring/cryptoKeys/b11-key
gcloud config set project "$PROJECT_ID"
gcloud services enable cloudkms.googleapis.com
```

1. Create a key ring and key in the same region as the bucket.
   ```bash
   gcloud kms keyrings create b11-ring --location="$REGION"
   gcloud kms keys create b11-key --keyring=b11-ring --location="$REGION" --purpose=encryption
   ```
2. Let the Cloud Storage service agent use the key.
   ```bash
   gcloud storage service-agent --authorize-cmek="$KEY"
   ```
3. Create the bucket with the key as its default, and upload an object.
   ```bash
   gcloud storage buckets create "$BUCKET" --location="$REGION" \
     --uniform-bucket-level-access --default-encryption-key="$KEY"
   echo "patient record 7" > /tmp/record.txt
   gcloud storage cp /tmp/record.txt "$BUCKET/record.txt"
   gcloud storage objects describe "$BUCKET/record.txt" --format="value(kms_key)"
   gcloud storage cat "$BUCKET/record.txt"
   ```
4. Disable key version 1, then read again.
   ```bash
   gcloud kms keys versions disable 1 --key=b11-key --keyring=b11-ring --location="$REGION"
   gcloud storage cat "$BUCKET/record.txt"
   ```
5. Re-enable it and read again.
   ```bash
   gcloud kms keys versions enable 1 --key=b11-key --keyring=b11-ring --location="$REGION"
   gcloud storage cat "$BUCKET/record.txt"
   ```

## Expected output
- Step 3: the object's `kms_key` shows your key (with a version suffix); `cat` prints the text.
- Step 4: the read fails with an error about the key being disabled or unusable. Allow a minute or two for the disable to take effect.
- Step 5: the read works again.

What to notice: your role on the bucket didn't change. Losing access to the key was enough. Destroying the version, not only disabling it, would make the data unrecoverable once the scheduled destruction period ends.

## Teardown
Delete the bucket first, then schedule the key version for destruction. Key rings and keys can't be deleted; only key versions are destroyed.

```bash
gcloud storage rm -r "$BUCKET"
gcloud kms keys versions destroy 1 --key=b11-key --keyring=b11-ring --location="$REGION"
gcloud kms keys versions list --key=b11-key --keyring=b11-ring --location="$REGION"
```
The version shows `DESTROY_SCHEDULED`. The default wait before destruction is 30 days.

## Fallback: delete the project
```bash
gcloud projects delete "$PROJECT_ID"
```
