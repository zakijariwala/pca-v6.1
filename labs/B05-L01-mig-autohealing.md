# LAB B05-L01: Regional MIG behind a load balancer, with autohealing

## Goal
Run a regional managed instance group behind a global external Application Load Balancer. Break the app on one VM and watch the MIG recreate it.

## What reading can't teach
How long a health-check-driven repair takes, what the VM's `currentAction` shows during it, and that the load balancer keeps serving from the healthy VM.

## Cost ceiling
Under USD 1.00 for two `e2-micro` VMs and one load balancer forwarding rule, under 90 minutes. Forwarding rules bill by the hour. [UNVERIFIED: check the pricing calculator for your region.] Verified: 2026-09-24.

## Time
60 to 90 minutes, including teardown. The load balancer takes several minutes to start serving.

## Prerequisites
- A sandbox project with a budget alert (LAB B01-L01).
- A `default` VPC network in the project. If it doesn't exist, create a custom VPC and add `--network` and `--subnet` flags.
- Cloud Shell.

## Steps (Cloud Shell)
Replace PROJECT_ID. Keep real IDs out of any file you commit.

```bash
export PROJECT_ID=PROJECT_ID
export REGION=us-central1
gcloud config set project "$PROJECT_ID"
gcloud services enable compute.googleapis.com iap.googleapis.com
```

1. Instance template with a startup script that serves the hostname.
   ```bash
   gcloud compute instance-templates create b05-tpl --machine-type=e2-micro \
     --image-family=debian-12 --image-project=debian-cloud --tags=b05-web \
     --metadata=startup-script='#! /bin/bash
   apt-get update && apt-get install -y nginx
   echo "served by $(hostname)" > /var/www/html/index.html'
   ```
2. Firewall: health checks and load balancer traffic, plus IAP SSH.
   ```bash
   gcloud compute firewall-rules create b05-allow-hc --network=default --direction=INGRESS \
     --rules=tcp:80 --source-ranges=130.211.0.0/22,35.191.0.0/16 --target-tags=b05-web
   gcloud compute firewall-rules create b05-allow-iap --network=default --direction=INGRESS \
     --rules=tcp:22 --source-ranges=35.235.240.0/20 --target-tags=b05-web
   ```
3. Health check and a regional MIG with autohealing. The initial delay covers nginx install time.
   ```bash
   gcloud compute health-checks create http b05-hc --port=80 --request-path=/
   gcloud compute instance-groups managed create b05-mig --region="$REGION" \
     --template=b05-tpl --size=2 --health-check=b05-hc --initial-delay=180
   gcloud compute instance-groups managed set-named-ports b05-mig --region="$REGION" --named-ports=http:80
   ```
4. Load balancer: backend service, URL map, proxy, forwarding rule.
   ```bash
   gcloud compute backend-services create b05-bes --global --protocol=HTTP --port-name=http \
     --health-checks=b05-hc --load-balancing-scheme=EXTERNAL_MANAGED
   gcloud compute backend-services add-backend b05-bes --global \
     --instance-group=b05-mig --instance-group-region="$REGION"
   gcloud compute url-maps create b05-map --default-service=b05-bes
   gcloud compute target-http-proxies create b05-proxy --url-map=b05-map
   gcloud compute forwarding-rules create b05-fr --global --load-balancing-scheme=EXTERNAL_MANAGED \
     --target-http-proxy=b05-proxy --ports=80
   export LB_IP=$(gcloud compute forwarding-rules describe b05-fr --global --format="value(IPAddress)")
   ```
5. Wait until the load balancer answers, then watch it alternate between VMs.
   ```bash
   for i in $(seq 1 10); do curl -s -m 3 "http://$LB_IP/"; sleep 1; done
   ```
6. Break nginx on one VM.
   ```bash
   gcloud compute instance-groups managed list-instances b05-mig --region="$REGION"
   gcloud compute ssh INSTANCE_NAME --zone=INSTANCE_ZONE --tunnel-through-iap --command="sudo systemctl stop nginx"
   ```
7. Watch the repair and keep curling.
   ```bash
   watch -n 10 "gcloud compute instance-groups managed list-instances b05-mig --region=$REGION \
     --format='table(instance.basename(),zone.basename(),status,currentAction,instanceHealth[0].detailedHealthState)'"
   ```

## Expected output
- Step 5: responses alternate between two `served by b05-mig-xxxx` hostnames. Early 502s mean the load balancer isn't ready; wait.
- Step 7: the broken VM's health turns `UNHEALTHY`, then `currentAction` shows `RECREATING`, then `VERIFYING`, then `NONE` and `HEALTHY`. The VM keeps its name.
- Curl keeps returning 200 from the healthy VM throughout.

## Teardown
Delete in reverse order of creation.

```bash
gcloud compute forwarding-rules delete b05-fr --global --quiet
gcloud compute target-http-proxies delete b05-proxy --quiet
gcloud compute url-maps delete b05-map --quiet
gcloud compute backend-services delete b05-bes --global --quiet
gcloud compute instance-groups managed delete b05-mig --region="$REGION" --quiet
gcloud compute health-checks delete b05-hc --quiet
gcloud compute firewall-rules delete b05-allow-hc b05-allow-iap --quiet
gcloud compute instance-templates delete b05-tpl --quiet
```

Confirm nothing billable remains:
```bash
gcloud compute forwarding-rules list
gcloud compute instances list
```

## Fallback: delete the project
```bash
gcloud projects delete "$PROJECT_ID"
```
