---
block: B11
title: Security and compliance
pillars: [security]
exam_guide_refs: ["3.1", "3.2"]
last_verified: 2026-09-24
sources:
  - https://docs.cloud.google.com/kms/docs/key-management-service
  - https://docs.cloud.google.com/kms/docs/cmek
  - https://docs.cloud.google.com/kms/docs/destroy-restore
  - https://docs.cloud.google.com/storage/docs/encryption/customer-supplied-keys
  - https://docs.cloud.google.com/kms/docs/hsm
  - https://docs.cloud.google.com/kms/docs/ekm
  - https://docs.cloud.google.com/secret-manager/docs/overview
  - https://docs.cloud.google.com/vpc-service-controls/docs/overview
  - https://docs.cloud.google.com/iap/docs/concepts-overview
  - https://docs.cloud.google.com/armor/docs/cloud-armor-overview
  - https://docs.cloud.google.com/security-command-center/docs/security-command-center-overview
  - https://docs.cloud.google.com/binary-authorization/docs/overview
  - https://docs.cloud.google.com/assured-workloads/access-transparency/docs/overview
  - https://docs.cloud.google.com/assured-workloads/access-approval/docs/overview
  - https://docs.cloud.google.com/assured-workloads/docs/overview
  - https://docs.cloud.google.com/kms/docs/autokey-overview
  - https://docs.cloud.google.com/vpc-service-controls/docs/dry-run-mode
  - https://cloud.google.com/security/compliance/hipaa
unverified: []
---
# B11 Security and compliance

## Chunk 1: Encryption key options
Problem it solves: All data at rest is encrypted. The question is who controls the key.

Mental model:

| Option | Who holds the key | Fits |
|---|---|---|
| Google default encryption | Google | Most data |
| CMEK (Cloud KMS) | You manage it in Cloud KMS | Control rotation, disable, audit key use |
| Cloud KMS Autokey | Cloud KMS creates and assigns CMEKs for you | CMEK at scale without manual setup |
| Cloud HSM | Keys in FIPS 140-2 Level 3 HSMs Google runs | Regulation demands hardware-backed keys |
| Cloud EKM | Your external key manager, outside Google | Keys must never live in Google Cloud |
| CSEK | You supply the raw key with each request | Cloud Storage and Compute Engine only; Google doesn't keep the key |

Destroying a key version makes data encrypted with it unreadable. Destruction waits in a scheduled state, 30 days by default, during which you can restore it.

Exam signals: "Company must control and rotate keys" → CMEK. "Keys must stay outside Google" → EKM. "FIPS 140-2 Level 3" → Cloud HSM.

Trap: CSEK when the need is key control. With CSEK, lose the key and the data is gone, and few services support it.

Pillar tie-in: Security.

Check: A regulator requires encryption keys held outside Google's infrastructure. Which option?
<details><summary>Answer</summary>Cloud EKM.</details>

## Chunk 2: Secret Manager
Problem it solves: Passwords and API keys end up in code, env files, and images.

Mental model: A secret holds versions; each version stores the value. IAM controls who can read it, audit logs record access, and you can pin workloads to a version and roll back. Workloads fetch secrets at runtime with their service account.

Exam signals: "Database password in application config", "rotate API keys."

Trap: Storing secrets in Cloud Storage objects or environment variables baked into images.

Pillar tie-in: Security.

Check: A secret was leaked. How do versions help?
<details><summary>Answer</summary>Add a new version with a new value, move workloads to it, and disable the leaked version. Audit logs show who read it.</details>

## Chunk 3: VPC Service Controls
Problem it solves: Someone with valid credentials copies data from BigQuery or Cloud Storage to a project outside the company.

Mental model: A service perimeter wraps projects and the Google APIs they use. Requests crossing the perimeter get blocked unless an ingress or egress rule or an access level allows them. IAM says who can act; the perimeter says from where and to where data can move. Dry-run mode logs violations before you enforce.

Exam signals: "Prevent data exfiltration", "stolen credentials must not copy data out."

Trap: IAM alone against exfiltration. A valid identity with read access can still copy data out.

Pillar tie-in: Security.

Check: What does VPC Service Controls add on top of IAM?
<details><summary>Answer</summary>A perimeter that blocks data from moving to or from resources outside it, even for principals IAM allows.</details>

## Chunk 4: Identity-Aware Proxy and Cloud Armor
Problem it solves: Protect apps without a VPN, and stop web attacks at the edge.

Mental model: IAP puts an identity check in front of HTTPS apps and TCP access (SSH, RDP): users sign in, IAM and context decide access. It's zero trust access without a VPN. Cloud Armor attaches to external Application Load Balancers and gives DDoS protection, WAF rules including preconfigured rules for OWASP Top 10 risks, rate limiting, and IP or geo rules.

Exam signals: "Admins reach internal tools without a VPN" → IAP. "SQL injection attempts", "block traffic from a country", "DDoS" → Cloud Armor.

Trap: Cloud Armor to control which employees reach an admin page. That's identity: IAP.

Pillar tie-in: Security.

Check: A web app faces SQL injection attempts. What do you attach, and to what?
<details><summary>Answer</summary>A Cloud Armor security policy with preconfigured WAF rules, attached to the backend service of the external Application Load Balancer.</details>

## Chunk 5: Security Command Center and Binary Authorization
Problem it solves: See misconfigurations and threats across the org, and stop unapproved images from running.

Mental model: Security Command Center finds vulnerabilities (misconfigurations, public exposure, leaked credentials) and threats, and exports findings to BigQuery and Pub/Sub. Tiers are Standard and Premium; the Enterprise tier shuts down on May 21, 2027, and moves to Premium. Binary Authorization enforces, at deploy time on GKE and Cloud Run, that images meet a policy, such as carrying attestations from your build pipeline.

Exam signals: "Central view of security posture" → Security Command Center. "Only images built by our CI may run" → Binary Authorization.

Trap: Scanning images after they're running instead of blocking them at deploy.

Pillar tie-in: Security.

Check: Which service blocks a container that lacks a build attestation from deploying to GKE?
<details><summary>Answer</summary>Binary Authorization.</details>

## Chunk 6: Controlling Google's own access
Problem it solves: Regulated customers must see, and sometimes approve, access by Google staff.

Mental model: Access Transparency logs record actions Google personnel take on your content. Access Approval requires your explicit approval before Google staff access your data. Assured Workloads applies control packages for regimes such as FedRAMP and data residency, including limits on where data lives and which personnel can support it.

Exam signals: "Know when Google support accesses data" → Access Transparency. "Approve each access" → Access Approval. "Government or sovereign workloads" → Assured Workloads.

Trap: Cloud Audit Logs as the answer for Google staff access. Access Transparency covers that.

Pillar tie-in: Security and compliance.

Check: A bank must approve any Google staff access to its data in advance. What feature?
<details><summary>Answer</summary>Access Approval.</details>

## Chunk 7: Compliance mapping
Problem it solves: Map a regulation to concrete controls.

Mental model:

| Requirement | Controls |
|---|---|
| Health data (HIPAA) | A Business Associate Agreement with Google where required; covered services only; CMEK, audit logs, VPC Service Controls |
| Card data (PCI DSS) | Isolate the cardholder environment in its own projects; Cloud Armor, VPC SC, Sensitive Data Protection for tokenizing card numbers |
| Personal data in the EU (GDPR) | Resource location org policy, EU regions, Sensitive Data Protection, access controls, deletion processes |
| Sovereignty | Assured Workloads, EKM, Access Approval |

Compliance is shared: Google certifies the platform; you configure your workloads to comply.

Exam signals: "Health records", "credit card numbers", "EU residents' data."

Trap: "Google Cloud is HIPAA compliant, so we're done." No HHS-recognized HIPAA certification exists. HIPAA compliance is shared: the customer configures the workload and puts a BAA in place where required.

Pillar tie-in: Security.

Check: Name three controls for a workload that stores EU personal data and must keep it in the EU.
<details><summary>Answer</summary>Any three of: resource location org policy restricted to EU, EU regions, Sensitive Data Protection for discovery and de-identification, least-privilege IAM, CMEK, VPC Service Controls, audit logs.</details>
