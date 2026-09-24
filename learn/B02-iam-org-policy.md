---
block: B02
title: IAM and org policy
pillars: [security]
exam_guide_refs: ["3.1"]
last_verified: 2026-09-24
sources:
  - https://docs.cloud.google.com/iam/docs/roles-overview
  - https://docs.cloud.google.com/iam/docs/policy-types
  - https://docs.cloud.google.com/iam/docs/deny-overview
  - https://docs.cloud.google.com/iam/docs/conditions-overview
  - https://docs.cloud.google.com/iam/docs/service-account-impersonation
  - https://docs.cloud.google.com/iam/docs/workload-identity-federation
  - https://docs.cloud.google.com/organization-policy/restrict-service-accounts
  - https://docs.cloud.google.com/resource-manager/docs/secure-by-default-organizations
  - https://docs.cloud.google.com/policy-intelligence/docs/troubleshoot-access
unverified: []
---
# B02 IAM and org policy

## Chunk 1: Who, what, where
Problem it solves: Every access question on the exam reduces to three parts. Missing one gives the wrong grant.

Mental model: An allow policy binds a principal (who) to a role (what) on a resource (where). Principals include Google accounts, groups, service accounts, and federated identities. A role is a bundle of permissions such as `compute.instances.start`. You attach the policy at org, folder, project, or some individual resources.

Grant to groups, not users. When someone changes teams, you change group membership and every grant follows.

Exam signals: "New hires in the data team need access", "people change roles often."

Trap: Granting roles to individual user accounts. It works on day one and rots by month three.

Pillar tie-in: Security.

Check: Why grant roles to a group instead of each engineer?
<details><summary>Answer</summary>Access follows group membership. Onboarding and offboarding become one change in the directory, and audits read one binding instead of dozens.</details>

## Chunk 2: Basic, predefined, custom roles
Problem it solves: You need least privilege without hand-building permission lists for every job.

Mental model:

| Type | Examples | When |
|---|---|---|
| Basic | Owner, Editor, Viewer | Avoid in production; they span every service |
| Predefined | `roles/compute.instanceAdmin.v1` | Default choice; Google maintains them |
| Custom | Your own permission list | When no predefined role is narrow enough |

Google's guidance: in production, don't grant basic roles unless there's no alternative. Custom roles cost you upkeep: when Google adds permissions, predefined roles update and custom roles don't.

Exam signals: "Least privilege", "auditor needs read-only access to logs only."

Trap: Editor "to unblock them." Editor can change almost everything in the project.

Pillar tie-in: Security.

Check: An auditor must read logs and nothing else. Basic Viewer or a predefined logging role?
<details><summary>Answer</summary>A predefined logging role such as Logs Viewer. Viewer grants read access across every service in the project.</details>

## Chunk 3: Allow, deny, and principal access boundary
Problem it solves: Some permissions must never be used, no matter who grants what lower down.

Mental model: IAM evaluates three policy types. Principal access boundary policies say which resources a principal is eligible to reach. Deny policies block specific permissions for specific principals. Allow policies grant. Deny wins over allow: if a deny rule covers the permission, an allow grant can't override it.

Exam signals: "Ensure no one outside the security team can delete these keys, whatever roles they hold."

Trap: Trying to block access by removing allow grants everywhere. A new grant tomorrow reopens it. A deny policy at the right level holds.

Pillar tie-in: Security.

Check: A user has Owner on a project and a deny policy blocks `resourcemanager.projects.delete` for them. Can they delete the project?
<details><summary>Answer</summary>No. Deny policies take precedence over allow policies.</details>

## Chunk 4: Service accounts: keys vs impersonation
Problem it solves: Workloads need an identity. How that identity proves itself decides your breach risk.

Mental model: A service account is an identity for code. On Google Cloud compute, attach it to the resource and the metadata server hands out short-lived tokens. From outside Google Cloud, prefer impersonation or Workload Identity Federation. A service account key is a long-lived credential file; anyone holding the file is the service account until someone revokes it.

Organizations created on or after May 3, 2024 enforce `iam.disableServiceAccountKeyCreation` by default as part of Google's security baseline.

Exam signals: "Avoid long-lived credentials", "a key was committed to a public repo."

Trap: Any answer that downloads a service account key when an attached service account, impersonation, or federation works. That's the red flag the exam wants you to spot.

Pillar tie-in: Security.

Check: A Compute Engine VM needs to write to a bucket. Key file or attached service account?
<details><summary>Answer</summary>Attached service account with a role scoped to the bucket. The VM gets short-lived tokens from the metadata server; no key exists to leak.</details>

## Chunk 5: Workload Identity Federation
Problem it solves: A GitHub Actions pipeline or an AWS workload needs to call Google Cloud APIs without a stored key.

Mental model: You trust an external identity provider (AWS, Azure, an OIDC or SAML provider, a CI system) through a workload identity pool. The workload presents its own token, exchanges it at Google's Security Token Service for a short-lived Google token, and can impersonate a service account. No Google secret is stored outside Google Cloud.

Exam signals: "Multicloud workload", "CI/CD in GitHub deploys to Google Cloud", "no keys."

Trap: Creating a service account key and pasting it into the CI secret store.

Pillar tie-in: Security.

Check: A deployment pipeline in another cloud needs to push images to Artifact Registry. What's the keyless option?
<details><summary>Answer</summary>Workload Identity Federation: trust the other cloud's identity provider, exchange its token for a short-lived Google token, and impersonate a service account with Artifact Registry write access.</details>

## Chunk 6: IAM Conditions
Problem it solves: A grant should apply only at certain times, to certain resources, or to resources with certain tags.

Mental model: A condition is an expression on a role binding. It uses attributes such as `request.time`, `resource.name`, and resource tags. The binding applies only when the expression is true.

Exam signals: "Temporary access until Friday", "contractors only on resources tagged dev."

Trap: Creating a separate project to isolate access that a tag-based condition would handle.

Pillar tie-in: Security.

Check: How do you give an on-call engineer elevated access that expires at the end of the shift?
<details><summary>Answer</summary>A role binding with an IAM condition on `request.time` that ends at the shift end.</details>

## Chunk 7: Organization Policy Service
Problem it solves: IAM controls who can act. Some things nobody should do, such as creating external IPs or service account keys.

Mental model: An organization policy sets a constraint on resource configuration, independent of IAM. It inherits down the hierarchy like other policies. Examples: disable service account key creation, restrict resource locations, restrict VMs with external IPs. IAM answers "who can"; org policy answers "what's allowed at all."

Exam signals: "Data must stay in the EU", "no public IPs anywhere in prod."

Trap: Solving a "never allowed" rule with IAM. An admin with the right role can still do it. An org policy blocks the configuration itself.

Pillar tie-in: Security and compliance.

Check: Compliance says no resource may be created outside the EU. IAM or org policy?
<details><summary>Answer</summary>Org policy, with the resource locations constraint set at the org or folder.</details>

## Chunk 8: Proving a deny with Policy Troubleshooter
Problem it solves: "Why can't she do X?" You need evidence, not guesses.

Mental model: Policy Troubleshooter takes a principal, a resource, and a permission. It checks allow, deny, and principal access boundary policies, then says whether access is granted and which policies decide it.

```bash
gcloud policy-intelligence troubleshoot-policy iam RESOURCE \
  --principal-email=EMAIL --permission=PERMISSION
```

Exam signals: "Determine why a user can't access a resource."

Trap: Granting a bigger role to test whether access was the problem.

Pillar tie-in: Security and operational excellence.

Check: What three inputs does Policy Troubleshooter need?
<details><summary>Answer</summary>A principal, a resource, and a permission.</details>
