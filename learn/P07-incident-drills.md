---
block: P07
title: Incident drills
pillars: [operational-excellence, reliability]
exam_guide_refs: ["4.1", "4.3", "6.4", "6.6"]
last_verified: 2026-09-24
sources:
  - https://docs.cloud.google.com/nat/docs/troubleshooting
  - https://docs.cloud.google.com/nat/docs/monitoring
  - https://docs.cloud.google.com/policy-intelligence/docs/service-account-insights
  - https://docs.cloud.google.com/logging/docs/audit
  - https://docs.cloud.google.com/kubernetes-engine/docs/troubleshooting/oom-events
  - https://docs.cloud.google.com/kubernetes-engine/docs/troubleshooting/crashloopbackoff-events
  - https://docs.cloud.google.com/run/docs/configuring/vpc-direct-vpc
  - https://docs.cloud.google.com/sql/docs/postgres/configure-private-ip
  - https://docs.cloud.google.com/docs/quotas/troubleshoot
  - https://docs.cloud.google.com/docs/quotas/overview
unverified: []
---
# P07 Incident drills

This block is practice, not new material. Each incident gives a symptom and an alert. Work it before you open anything: write the commands you'd run first and why. Then open the hints one at a time. Output shown is illustrative and trimmed; real output varies.

Time each drill. Score yourself on two things: time to root cause, and whether you checked the obvious first.

## Chunk 1: The drill method
Problem it solves: Under pressure people skip steps and chase theories.

Mental model: Every incident, same loop.
1. Scope: what's broken, since when, for whom? What changed?
2. Stabilize: roll back or fail over if that's safe and fast.
3. Diagnose: cheapest check that rules out the most, first.
4. Fix and verify.
5. Postmortem in 5 lines: impact, root cause, detection, fix, prevention. Blameless.

Exam signals: "What should you do first?" → scope and recent changes, or restore service.

Trap: Diagnosing for an hour while a rollback would restore service in two minutes.

Pillar tie-in: Operational excellence.

Check: What goes in a 5-line postmortem?
<details><summary>Answer</summary>Impact, root cause, detection, fix, prevention.</details>

## Chunk 2: Incident 1, networking
Symptom: Since this morning's marketing launch, about 5% of calls from backend VMs to a partner API time out. The VMs have no external IPs. Nothing was deployed.

Alert: `Uptime check "partner-api-egress" failing from 2 of 6 probes.` Error rate on `orders-backend` is above threshold.

Check: What do you check first, and what would confirm the cause?
<details><summary>Hint 1</summary>The VMs reach the internet through Cloud NAT. Failures started with a traffic spike, not a deploy, and only some calls fail.</details>
<details><summary>Hint 2: what you'd see</summary>

```
$ # Metrics Explorer: router.googleapis.com/nat/dropped_sent_packets_count, grouped by reason
reason=OUT_OF_RESOURCES   rising since 09:05
```
</details>
<details><summary>Root cause, fix, postmortem</summary>

Root cause: NAT source port exhaustion. Traffic to one partner IP and port rose; each VM ran out of its static 64-port allocation.

Fix: enable dynamic port allocation and raise the minimum and maximum ports per VM, or add NAT IPs. Watch `OUT_OF_RESOURCES` fall to zero.

Postmortem:
- Impact: 5% of partner calls failed for 2 hours during the launch.
- Root cause: Cloud NAT port exhaustion under a traffic spike to a single destination.
- Detection: uptime check and error rate alert; no NAT metric alert existed.
- Fix: dynamic port allocation, higher port limits.
- Prevention: alert on `dropped_sent_packets_count` by reason; load test egress before launches.
</details>

## Chunk 3: Incident 2, IAM
Symptom: The quarterly reconciliation job failed at 02:00 with `403 Permission denied`. It ran fine last quarter. No code changed.

Alert: `Cloud Run job "quarterly-recon" execution failed.`

Check: What do you check first?
<details><summary>Hint 1</summary>"No code changed" and "ran fine last quarter." Look for changes to the identity, not the code. What does Admin Activity show about the job's service account?</details>
<details><summary>Hint 2: what you'd see</summary>

```
$ gcloud logging read 'protoPayload.methodName:"DisableServiceAccount"' --freshness=30d \
    --format="table(timestamp,protoPayload.authenticationInfo.principalEmail,protoPayload.resourceName)"
TIMESTAMP             PRINCIPAL_EMAIL           RESOURCE_NAME
2026-09-02T14:10:03Z  sec-cleanup@...           .../serviceAccounts/recon-runner@...
```
</details>
<details><summary>Root cause, fix, postmortem</summary>

Root cause: a cleanup drive disabled service accounts flagged as unused for 90 days. The quarterly job's account ran every 90+ days, so it looked unused.

Fix: re-enable the service account (disabled accounts can be re-enabled), rerun the job.

Postmortem:
- Impact: quarterly reconciliation delayed by one day.
- Root cause: service account disabled by an unused-account cleanup; the job's schedule is longer than the 90-day insight window.
- Detection: job failure alert.
- Fix: re-enabled the account.
- Prevention: label service accounts with owner and schedule; exclude known low-frequency jobs from cleanup; notify owners before disabling.
</details>

## Chunk 4: Incident 3, GKE
Symptom: After the 15:00 release of `checkout`, its Pods restart every few minutes. Latency spikes with each restart.

Alert: `checkout: container restarts > 5 in 10 minutes.`

Check: What are your first two commands?
<details><summary>Hint 1</summary>Restarts started with a release. Look at the Pod's last state and exit code before reading app logs.</details>
<details><summary>Hint 2: what you'd see</summary>

```
$ kubectl describe pod checkout-7c9f... | grep -A4 "Last State"
    Last State:     Terminated
      Reason:       OOMKilled
      Exit Code:    137
```
</details>
<details><summary>Root cause, fix, postmortem</summary>

Root cause: the new version loads a larger product cache at startup and exceeds its memory limit. The kernel OOM killer ends the container (exit 137).

Fix now: `kubectl rollout undo deployment/checkout`. Lasting fix: raise the memory request and limit to match measured use, or shrink the cache; use VPA recommendations to size it.

Postmortem:
- Impact: elevated checkout latency for 40 minutes.
- Root cause: memory use of the new release exceeded the container limit.
- Detection: restart-count alert.
- Fix: rollback, then a release with correct memory settings.
- Prevention: load test releases with production-sized data; canary through Cloud Deploy with an SLO gate.
</details>

## Chunk 5: Incident 4, Cloud SQL
Symptom: A new Cloud Run service can't connect to the orders database. Existing GKE services connect fine. The database has a private IP only.

Alert: `orders-api-v2: 100% of requests returning 500.` Logs show `connection timed out` to `10.x.x.x:5432`.

Check: Why can GKE reach the database and the new Cloud Run service can't?
<details><summary>Hint 1</summary>GKE nodes sit in the VPC. Where does a Cloud Run service send traffic to private IPs by default?</details>
<details><summary>Hint 2: what you'd see</summary>

```
$ gcloud run services describe orders-api-v2 --region=REGION --format="yaml(spec.template.metadata.annotations)"
annotations:
  autoscaling.knative.dev/maxScale: '20'
# no VPC network or connector annotations
```
</details>
<details><summary>Root cause, fix, postmortem</summary>

Root cause: the Cloud Run service has no VPC egress configured, so it can't reach the private IP.

Fix: redeploy with Direct VPC egress into the same VPC (or a Serverless VPC Access connector), and allow the traffic in firewall rules.

Postmortem:
- Impact: new API unavailable for 25 minutes after launch.
- Root cause: missing VPC egress on the Cloud Run service.
- Detection: error rate alert.
- Fix: redeployed with Direct VPC egress.
- Prevention: deploy from a Terraform module that sets VPC egress; add a connectivity smoke test to the pipeline.
</details>

## Chunk 6: Incident 5, quota and billing
Symptom: During a traffic spike, the web tier's managed instance group stops growing at 24 VMs. Latency climbs. The autoscaler's target is well above 24.

Alert: `web-mig: autoscaler recommended size 40, current size 24.`

Check: What limits growth that isn't the autoscaler?
<details><summary>Hint 1</summary>The autoscaler asked for more. Something refused to create them. Check the MIG's errors and the project's regional quotas.</details>
<details><summary>Hint 2: what you'd see</summary>

```
$ gcloud compute instance-groups managed list-errors web-mig --region=REGION --limit=3
ERROR: Quota 'CPUS' exceeded. Limit: 48.0 in region REGION.
```
</details>
<details><summary>Root cause, fix, postmortem</summary>

Root cause: the regional CPU quota caps the project at 48 vCPUs, which is 24 two-vCPU VMs.

Fix now: request a quota increase; shift some load to another region if the design allows. Lasting fix: plan quota ahead of expected peaks; consider the quota adjuster.

Postmortem:
- Impact: high latency for 1 hour at peak.
- Root cause: regional CPU quota reached; autoscaler couldn't add VMs.
- Detection: latency alert; the size gap wasn't alerted.
- Fix: quota increase.
- Prevention: alert on quota usage above 80%; review quotas before launches; enable the quota adjuster.

Related: if a project's billing account stops being billable, the project can't use paid services. Check billing status when many services fail at once.
</details>
