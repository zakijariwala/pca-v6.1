# LAB B06-L01: Deploy a container to Cloud Run

## Goal
Deploy Google's sample container to Cloud Run, call it, push a second revision, split traffic between revisions, and watch it scale to zero.

## What reading can't teach
How fast a deploy is, what a revision is in practice, and what a cold start feels like after idle.

## Cost ceiling
Under USD 0.10. A few requests on a service that scales to zero. [UNVERIFIED: check Cloud Run pricing and free tier for your region.] Verified: 2026-09-24.

## Time
30 minutes.

## Prerequisites
- A sandbox project with a budget alert (LAB B01-L01).
- Cloud Shell.
- If your organization blocks public (`allUsers`) access, skip `--allow-unauthenticated` and use the authenticated curl shown in step 2.

## Steps (Cloud Shell)
Replace PROJECT_ID. Keep real IDs out of any file you commit.

```bash
export PROJECT_ID=PROJECT_ID
export REGION=us-central1
gcloud config set project "$PROJECT_ID"
gcloud services enable run.googleapis.com
```

1. Deploy the sample.
   ```bash
   gcloud run deploy b06-hello --image=us-docker.pkg.dev/cloudrun/container/hello \
     --region="$REGION" --allow-unauthenticated
   export URL=$(gcloud run services describe b06-hello --region="$REGION" --format="value(status.url)")
   ```
2. Call it.
   ```bash
   curl -s "$URL" | head -20
   # If public access is blocked:
   curl -s -H "Authorization: Bearer $(gcloud auth print-identity-token)" "$URL" | head -20
   ```
3. Push a second revision with an environment variable, and keep all traffic on the first.
   ```bash
   gcloud run deploy b06-hello --image=us-docker.pkg.dev/cloudrun/container/hello \
     --region="$REGION" --set-env-vars=COLOR=blue --no-traffic
   gcloud run revisions list --service=b06-hello --region="$REGION"
   ```
4. Split traffic 50/50.
   ```bash
   gcloud run services update-traffic b06-hello --region="$REGION" \
     --to-revisions=REVISION_1=50,REVISION_2=50
   gcloud run services describe b06-hello --region="$REGION" --format="yaml(status.traffic)"
   ```
5. Leave it idle for 15 minutes, then time a request.
   ```bash
   time curl -s -o /dev/null "$URL"
   time curl -s -o /dev/null "$URL"
   ```

## Expected output
- Step 1: a `Service URL` ending in `.run.app`.
- Step 2: an HTML page from the sample.
- Step 3: two revisions; the new one shows 0% traffic.
- Step 4: 50/50 split in `status.traffic`.
- Step 5: the first call takes longer than the second. That gap is the cold start after scale to zero.

## Teardown
```bash
gcloud run services delete b06-hello --region="$REGION" --quiet
gcloud run services list --region="$REGION"
```

## Fallback: delete the project
```bash
gcloud projects delete "$PROJECT_ID"
```
