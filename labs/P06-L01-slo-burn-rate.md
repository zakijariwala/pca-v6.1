# LAB P06-L01: SLO and burn-rate alert on Cloud Run, triggered on purpose

## Goal
Deploy a small Cloud Run service that can be told to fail. Define an availability SLO on it, add a burn-rate alert, then break the service and watch the alert fire.

## What reading can't teach
How the SLO UI turns request counts into a budget, how long a burn-rate alert takes to fire, and what the notification looks like.

## Cost ceiling
Under USD 0.20. One Cloud Run service that scales to zero, one source build, and a few hundred requests. [UNVERIFIED: check Cloud Run and Cloud Build pricing.] Verified: 2026-09-24.

## Time
60 minutes, including waiting for the alert.

## Prerequisites
- A sandbox project with a budget alert (LAB B01-L01).
- LAB B06-L01 done.
- Cloud Shell. SLO setup below uses the console; the rest uses gcloud.

## Steps (Cloud Shell)
Replace PROJECT_ID. Keep real IDs out of any file you commit.

```bash
export PROJECT_ID=PROJECT_ID
export REGION=us-central1
gcloud config set project "$PROJECT_ID"
gcloud services enable run.googleapis.com cloudbuild.googleapis.com artifactregistry.googleapis.com monitoring.googleapis.com
```

1. Write a service that returns 500 when `FAIL=1`.
   ```bash
   mkdir -p ~/p06 && cd ~/p06
   cat > main.py <<'EOF'
   import os
   from flask import Flask
   app = Flask(__name__)

   @app.route("/")
   def index():
       if os.environ.get("FAIL") == "1":
           return "broken on purpose", 500
       return "ok", 200
   EOF
   printf "flask\ngunicorn\n" > requirements.txt
   echo 'web: gunicorn -b :$PORT main:app' > Procfile
   ```
2. Deploy from source. Cloud Run builds it with buildpacks.
   ```bash
   gcloud run deploy p06-svc --source=. --region="$REGION" --allow-unauthenticated --set-env-vars=FAIL=0
   export URL=$(gcloud run services describe p06-svc --region="$REGION" --format="value(status.url)")
   for i in $(seq 1 100); do curl -s -o /dev/null "$URL"; done
   ```
   If your org blocks public access, drop `--allow-unauthenticated` and add `-H "Authorization: Bearer $(gcloud auth print-identity-token)"` to each curl.
3. Create the SLO (console): Monitoring → SLOs → pick the `p06-svc` Cloud Run service → Create SLO.
   - Metric: Availability. Evaluation: Request-based.
   - Compliance period: Rolling, 1 day. Goal: 99%.
4. Add a burn-rate alert on that SLO (console): on the SLO, Create SLO alert.
   - Lookback duration: 10 minutes. Burn rate threshold: 2.
   - Notification channel: your email.
5. Break the service and send traffic.
   ```bash
   gcloud run services update p06-svc --region="$REGION" --update-env-vars=FAIL=1
   for i in $(seq 1 300); do curl -s -o /dev/null -w '%{http_code}\n' "$URL"; sleep 1; done | sort | uniq -c
   ```
6. Watch the SLO page and your inbox. Then fix it.
   ```bash
   gcloud run services update p06-svc --region="$REGION" --update-env-vars=FAIL=0
   ```

## Expected output
- Step 2: 100 successful requests; the SLO page later shows about 100% compliance.
- Step 5: `300 500`.
- Step 6: the error budget drops, the burn-rate alert opens an incident, and an email arrives within minutes. After the fix, the incident closes.

What to notice: the alert fired on budget burn, not on a raw error count. 300 failures against a 1-day, 99% budget is a fast burn.

## Teardown
```bash
gcloud run services delete p06-svc --region="$REGION" --quiet
rm -rf ~/p06
```
In the console, delete the SLO alert policy (Monitoring → Alerting) and the SLO. Source deploys store images in an Artifact Registry repository named `cloud-run-source-deploy`; delete it if you're done with Cloud Run labs:
```bash
gcloud artifacts repositories delete cloud-run-source-deploy --location="$REGION" --quiet
```

## Fallback: delete the project
```bash
gcloud projects delete "$PROJECT_ID"
```
