---
block: P08
title: Capstone
pillars: [security, reliability, cost, performance, operational-excellence]
exam_guide_refs: ["5.1", "5.2", "6.3"]
last_verified: 2026-09-24
sources:
  - https://docs.cloud.google.com/docs/terraform/resource-management/store-state
  - https://docs.cloud.google.com/run/docs/configuring/vpc-direct-vpc
  - https://docs.cloud.google.com/sql/docs/postgres/configure-private-ip
  - https://docs.cloud.google.com/secret-manager/docs/overview
  - https://docs.cloud.google.com/armor/docs/cloud-armor-overview
  - https://docs.cloud.google.com/armor/docs/security-policy-overview
  - https://docs.cloud.google.com/load-balancing/docs/negs/serverless-neg-concepts
  - https://docs.cloud.google.com/load-balancing/docs/ssl-certificates/google-managed-certs
  - https://docs.cloud.google.com/run/docs/securing/ingress
  - https://docs.cloud.google.com/iap/docs/concepts-overview
  - https://docs.cloud.google.com/deploy/docs/deployment-strategies/canary
  - https://docs.cloud.google.com/stackdriver/docs/solutions/slo-monitoring/alerting-on-budget-burn-rate
unverified: []
---
# P08 Capstone

A build, not a lesson. You build one small production-style system in Terraform, in one project, from a clean state, over 4 to 6 sessions. Each milestone ends with a review against the checklist below. Every resource must earn its place: you should be able to say what it does and how it fails.

Cost warning: the load balancer, Cloud SQL, and Cloud NAT bill by the hour. Run `terraform destroy` at the end of every session and rebuild at the start of the next. That's practice in itself.

## Chunk 1: The target system
Problem it solves: A capstone needs a fixed target so scope doesn't creep.

Mental model:

```
Users → global external Application LB (+ Cloud Armor)
        ├── /       → Cloud Run service (app)
        └── /admin  → same service, behind IAP
Cloud Run → Direct VPC egress → private subnet → Cloud SQL (private IP, HA)
Secrets in Secret Manager. Images in Artifact Registry.
Cloud Build → Cloud Deploy (canary) → Cloud Run
Monitoring: SLO + burn-rate alert, budget alert.
```

Pick any small app you can containerize; a to-do API is enough. Use Cloud Run unless P04 went well and you want GKE Autopilot.

Exam signals: This shape covers sections 2, 3, 5, and 6 of the exam guide.

Trap: Adding services the target doesn't need. Scope is the lesson.

Pillar tie-in: All five core pillars.

Check: Why does the database have no public IP?
<details><summary>Answer</summary>Nothing outside the VPC needs it. The app reaches it through Direct VPC egress; a public IP only adds attack surface.</details>

## Chunk 2: M1 Network
Build: custom VPC, one private subnet, Cloud Router and Cloud NAT, network firewall policy with only the rules you need, remote state in a versioned bucket, Terraform run through an impersonated service account.

Review checklist:
- [ ] No default network; custom mode with planned ranges.
- [ ] No firewall rule open to `0.0.0.0/0` on SSH or RDP.
- [ ] State bucket has versioning and restricted access.
- [ ] No service account key anywhere.
- [ ] `terraform destroy` then `apply` rebuilds it without errors.

Check: What fails if you forget Cloud NAT and the app calls an external API?
<details><summary>Answer</summary>Outbound calls from private resources to the internet time out; there's no external IP or NAT path.</details>

## Chunk 3: M2 Compute
Build: Artifact Registry repository, container image pushed by Cloud Build, Cloud Run service with its own least-privilege service account and Direct VPC egress into the subnet. Ingress set to `internal-and-cloud-load-balancing`.

Review checklist:
- [ ] Runtime service account isn't the default compute account.
- [ ] Image referenced by digest or immutable tag.
- [ ] Minimum and maximum instances set on purpose.
- [ ] Service not reachable on its `run.app` URL from the internet if ingress is restricted.

Check: Why give the service its own service account?
<details><summary>Answer</summary>Least privilege and clear audit trails. The default compute account often carries broad roles and is shared by other resources.</details>

## Chunk 4: M3 Data
Build: Cloud SQL (PostgreSQL) with private IP only and HA, database password in Secret Manager, app reads the secret at runtime with its service account.

Review checklist:
- [ ] `--no-assign-ip`; private services access configured.
- [ ] HA (regional) availability type; backups and point-in-time recovery on.
- [ ] Secret Accessor granted to the app's service account only, on that one secret.
- [ ] No credentials in Terraform variables files, images, or environment variables in plain text.

Check: The secret must rotate. What changes in the app?
<details><summary>Answer</summary>Nothing if it reads the latest secret version at startup or on a refresh; add a new version and roll a new revision.</details>

## Chunk 5: M4 Edge and security
Build: global external Application Load Balancer with a serverless NEG to Cloud Run, managed TLS certificate, Cloud Armor policy with preconfigured WAF rules and a rate limit, IAP on the `/admin` path.

Review checklist:
- [ ] HTTPS only; HTTP redirects.
- [ ] Cloud Armor in preview mode first, then enforced after checking logs.
- [ ] IAP access granted to a group, not individuals.
- [ ] Cloud Run ingress blocks direct traffic that bypasses the load balancer.

Check: Why run Cloud Armor rules in preview mode first?
<details><summary>Answer</summary>To see what they would block in logs before they block real users.</details>

## Chunk 6: M5 Ops
Build: Cloud Build trigger on push; Cloud Deploy pipeline with a canary on Cloud Run for prod; availability SLO with a burn-rate alert; budget alert; a documented teardown.

Review checklist:
- [ ] Separate service accounts for build and deploy, each least-privilege.
- [ ] Canary phases defined; rollback tested once on purpose.
- [ ] SLO alert fires in a deliberate break test (P06 lab method).
- [ ] Budget alert at 50/90/100% on the project.
- [ ] `terraform destroy` leaves nothing billable; `gcloud` list commands confirm it.

Check: What proves the canary protects users?
<details><summary>Answer</summary>A deliberate bad release: the canary gets a small share of traffic, the SLO alert fires, and you roll back before promoting to 100%.</details>

## Chunk 7: Exit review
For every resource in your Terraform, write one line: what it does, and how it fails. Examples:

| Resource | Does | Fails when |
|---|---|---|
| Cloud NAT | Egress for private resources | Ports run out under load |
| Cloud SQL HA | Survives a zone failure | Region outage; connection storms after failover |
| Cloud Armor | Blocks attacks at the edge | Rules too broad block real users |

Exit: you can explain every resource and its failure mode without notes.

Check: Name one failure mode for the load balancer's managed certificate.
<details><summary>Answer</summary>Provisioning stays pending if DNS doesn't point the domain at the load balancer's IP.</details>
