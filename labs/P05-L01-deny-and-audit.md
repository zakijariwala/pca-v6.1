# LAB P05-L01: Grant, deny, trace, and audit

## Goal
Give a test service account Storage Admin, block one permission with a deny policy, prove the block with Policy Troubleshooter, then delete a bucket yourself and find the deletion in the audit logs.

## What reading can't teach
How a deny beats an allow in practice, how Policy Troubleshooter shows it, and the shape of a real audit log entry.

## Cost ceiling
USD 0. Empty buckets and API calls. Verified: 2026-09-24.

## Time
45 minutes.

## Prerequisites
- A sandbox project with a budget alert (LAB B01-L01).
- Owner on the project, plus Deny Admin (`roles/iam.denyAdmin`) on the project. Owner doesn't include Deny Admin; grant it to yourself first:
  ```bash
  gcloud projects add-iam-policy-binding PROJECT_ID --member="user:$(gcloud config get-value account)" --role=roles/iam.denyAdmin
  ```
  If your organization blocks that grant, do steps 1, 5, and 6 only.
- LAB B02-L01 done.
- Cloud Shell.

## Steps (Cloud Shell)
Replace PROJECT_ID. Keep real IDs out of any file you commit.

```bash
export PROJECT_ID=PROJECT_ID
export REGION=us-central1
export SA=p05-admin@$PROJECT_ID.iam.gserviceaccount.com
export ME=$(gcloud config get-value account)
gcloud config set project "$PROJECT_ID"
gcloud services enable iam.googleapis.com iamcredentials.googleapis.com policytroubleshooter.googleapis.com
```

1. A service account with Storage Admin on the project, which you can impersonate, and two buckets.
   ```bash
   gcloud iam service-accounts create p05-admin
   gcloud projects add-iam-policy-binding "$PROJECT_ID" --member="serviceAccount:$SA" --role=roles/storage.admin
   gcloud iam service-accounts add-iam-policy-binding "$SA" --member="user:$ME" --role=roles/iam.serviceAccountTokenCreator
   gcloud storage buckets create "gs://$PROJECT_ID-p05-a" "gs://$PROJECT_ID-p05-b" --location="$REGION"
   ```
2. Deny bucket deletion for that service account.
   ```bash
   cat > /tmp/deny.json <<EOF
   {
     "displayName": "No bucket deletes for p05-admin",
     "rules": [{
       "denyRule": {
         "deniedPrincipals": ["principal://iam.googleapis.com/projects/-/serviceAccounts/$SA"],
         "deniedPermissions": ["storage.googleapis.com/buckets.delete"]
       }
     }]
   }
   EOF
   gcloud iam policies create p05-no-bucket-delete \
     --attachment-point="cloudresourcemanager.googleapis.com/projects/$PROJECT_ID" \
     --kind=denypolicies --policy-file=/tmp/deny.json
   ```
3. Try to delete bucket A as the service account. Wait a few minutes after step 2 for the policy to apply.
   ```bash
   gcloud storage buckets delete "gs://$PROJECT_ID-p05-a" --impersonate-service-account="$SA"
   ```
4. Ask Policy Troubleshooter why.
   ```bash
   gcloud policy-intelligence troubleshoot-policy iam \
     "//storage.googleapis.com/projects/_/buckets/$PROJECT_ID-p05-a" \
     --principal-email="$SA" --permission=storage.buckets.delete
   ```
5. Delete bucket B as yourself.
   ```bash
   gcloud storage buckets delete "gs://$PROJECT_ID-p05-b"
   ```
6. Find who deleted bucket B. Allow a minute for the log to arrive.
   ```bash
   gcloud logging read \
     "protoPayload.methodName=\"storage.buckets.delete\" AND resource.labels.bucket_name=\"$PROJECT_ID-p05-b\"" \
     --freshness=1h \
     --format="table(timestamp,protoPayload.authenticationInfo.principalEmail,protoPayload.requestMetadata.callerIp)"
   ```
7. Try on your own: find the step 1 grant of `roles/storage.admin`. Hint: the method is `SetIamPolicy`.

## Expected output
- Step 3: permission denied for `storage.buckets.delete`, even though the service account holds Storage Admin.
- Step 4: access not granted; the deny policy `p05-no-bucket-delete` is listed as the reason, alongside the allow policy that would have granted it.
- Step 6: one row with the time, your account, and your Cloud Shell IP.

What to notice: step 6 used Admin Activity logs, which are always on. You enabled nothing to get that answer.

## Teardown
```bash
gcloud iam policies delete p05-no-bucket-delete \
  --attachment-point="cloudresourcemanager.googleapis.com/projects/$PROJECT_ID" --kind=denypolicies
gcloud storage buckets delete "gs://$PROJECT_ID-p05-a"
gcloud projects remove-iam-policy-binding "$PROJECT_ID" --member="serviceAccount:$SA" --role=roles/storage.admin
gcloud iam service-accounts delete "$SA" --quiet
```

## Fallback: delete the project
```bash
gcloud projects delete "$PROJECT_ID"
```
