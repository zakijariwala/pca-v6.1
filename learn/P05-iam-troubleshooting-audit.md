---
block: P05
title: IAM troubleshooting and audit
pillars: [security, operational-excellence]
exam_guide_refs: ["3.1", "3.2", "4.1"]
last_verified: 2026-09-24
sources:
  - https://docs.cloud.google.com/policy-intelligence/docs/troubleshoot-access
  - https://docs.cloud.google.com/policy-intelligence/docs/analyze-iam-policies
  - https://docs.cloud.google.com/policy-intelligence/docs/role-recommendations-overview
  - https://docs.cloud.google.com/policy-intelligence/docs/service-account-insights
  - https://docs.cloud.google.com/logging/docs/audit
  - https://docs.cloud.google.com/logging/docs/audit/configure-data-access
  - https://docs.cloud.google.com/logging/docs/buckets
  - https://docs.cloud.google.com/logging/quotas
  - https://docs.cloud.google.com/iam/docs/deny-access
unverified: []
---
# P05 IAM troubleshooting and audit

## Chunk 1: Two kinds of ticket
Problem it solves: Access tickets come in two shapes. Each has its own tool.

Mental model:

| Ticket | Question | Tool |
|---|---|---|
| "Why can't she?" | Does this principal have this permission on this resource? | Policy Troubleshooter |
| "Who can?" | Which principals can do X on Y? | Policy Analyzer |
| "Who did?" | Who changed or deleted this? | Cloud Audit Logs |

Exam signals: "Determine why", "list everyone with access to", "find who deleted."

Trap: Reading IAM policies by hand across the hierarchy. Inherited grants and groups hide the answer.

Pillar tie-in: Operational excellence.

Check: Which tool answers "who can read this BigQuery dataset?"
<details><summary>Answer</summary>Policy Analyzer.</details>

## Chunk 2: Policy Troubleshooter and Policy Analyzer
Problem it solves: Evidence-based answers to access questions.

Mental model: Policy Troubleshooter takes a principal, resource, and permission. It evaluates allow, deny, and principal access boundary policies and shows which ones decide the result. Policy Analyzer answers "which principals have what access to which resources" across a project, folder, or org. It uses the Cloud Asset API, so its data freshness is best-effort. You can export results to BigQuery.

Exam signals: "Auditor needs a list of everyone with Owner in the org" → Policy Analyzer.

Trap: Expecting Policy Analyzer to reflect a grant made seconds ago.

Pillar tie-in: Security.

Check: What API must be on for Policy Analyzer to work?
<details><summary>Answer</summary>The Cloud Asset API.</details>

## Chunk 3: Shrinking permissions: recommender and insights
Problem it solves: Grants pile up. Nobody knows which are still needed.

Mental model: The IAM recommender suggests removing or replacing roles based on permissions used over the last 90 days. Service account insights flag service accounts unused for 90 days so you can disable or delete them. Disable first, wait, then delete; a disabled account can be re-enabled if something breaks.

Exam signals: "Reduce excess permissions", "service account sprawl."

Trap: Deleting unused service accounts without disabling first. A quarterly job you forgot about breaks and the account is gone.

Pillar tie-in: Security.

Check: What observation window does the IAM recommender use?
<details><summary>Answer</summary>90 days of permission usage.</details>

## Chunk 4: Audit log types
Problem it solves: Knowing which log records what, and which ones you must turn on.

Mental model:

| Type | Records | Default |
|---|---|---|
| Admin Activity | Config changes and metadata writes by users | Always on; can't disable or exclude |
| System Event | Changes by Google systems, such as a MIG adding a VM | Always on; can't disable or exclude |
| Data Access | Reads of config or data, and user data writes | Off by default, except BigQuery |
| Policy Denied | Access denied by a security policy, such as VPC Service Controls | On by default; can't disable, but can exclude with filters |

Exam signals: "Track who read patient records" → turn on Data Access logs for that service. "Prove who changed the firewall" → Admin Activity.

Trap: Expecting Data Access logs for Cloud Storage reads without turning them on.

Pillar tie-in: Security and compliance.

Check: A regulator asks who read objects in a bucket last month. Data Access logs were never enabled for Cloud Storage. Can you answer?
<details><summary>Answer</summary>No. Data Access logs are off by default for Cloud Storage, so the reads weren't recorded. Turn them on now for future reads.</details>

## Chunk 5: Where logs live and for how long
Problem it solves: A 2-year audit requirement meets a 30-day default.

Mental model: Each project has two log buckets. `_Required` holds Admin Activity and System Event logs for 400 days; you can't change that. `_Default` holds most other logs for 30 days by default; you can change its retention. For longer retention, route logs with a sink to a custom log bucket with longer retention (lockable), Cloud Storage, or BigQuery. Aggregated sinks at the org or folder collect logs from every project below.

Exam signals: "Retain audit logs for 7 years", "central security team sees all projects' logs."

Trap: Raising `_Default` retention in every project one at a time, instead of an aggregated sink at the org.

Pillar tie-in: Security and compliance.

Check: How long does `_Required` keep Admin Activity logs?
<details><summary>Answer</summary>400 days.</details>

## Chunk 6: Answering "who deleted this bucket"
Problem it solves: The most common audit question, answered in minutes.

Mental model: Deleting a bucket is an admin action, so it lands in Admin Activity audit logs. Filter on the method name and resource:

```
protoPayload.methodName="storage.buckets.delete"
resource.labels.bucket_name="BUCKET_NAME"
```

`protoPayload.authenticationInfo.principalEmail` names who did it; the entry also shows the caller IP and time. If the principal is a service account, look for impersonation in `serviceAccountDelegationInfo`.

Exam signals: "Find who deleted", "investigate a configuration change."

Trap: Searching Data Access logs for a delete. Deletes are admin activity.

Pillar tie-in: Security.

Check: Which field names the principal in an audit log entry?
<details><summary>Answer</summary>`protoPayload.authenticationInfo.principalEmail`.</details>

## Chunk 7: Deny policies and org policy violations in practice
Problem it solves: Proving a guardrail works, and finding what breaks one.

Mental model: A deny policy blocks named permissions (format `service.googleapis.com/resource.verb`, such as `storage.googleapis.com/buckets.delete`) for named principals, whatever their roles. Creating one needs the Deny Admin role. Policy Troubleshooter shows when a deny policy is the reason. For org policy, a violation shows up as a failed API call with an error naming the constraint; the Policy Denied logs record VPC Service Controls blocks.

Exam signals: "Prevent anyone except the security team from deleting logs buckets."

Trap: Removing the permission from every role. A deny policy does it in one place and survives new grants.

Pillar tie-in: Security.

Check: In what format does a deny policy name permissions?
<details><summary>Answer</summary>`SERVICE.googleapis.com/RESOURCE.VERB`, for example `storage.googleapis.com/buckets.delete`.</details>
