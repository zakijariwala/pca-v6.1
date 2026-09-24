---
block: B01
title: Resource hierarchy and billing
pillars: [security, cost, operational-excellence]
exam_guide_refs: ["3.1", "1.1", "4.2"]
last_verified: 2026-09-24
sources:
  - https://docs.cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy
  - https://docs.cloud.google.com/resource-manager/docs/creating-managing-folders
  - https://docs.cloud.google.com/resource-manager/docs/creating-managing-organization
  - https://docs.cloud.google.com/resource-manager/docs/tags/tags-overview
  - https://docs.cloud.google.com/billing/docs/how-to/budgets
  - https://docs.cloud.google.com/billing/docs/how-to/budgets-programmatic-notifications
  - https://docs.cloud.google.com/docs/quotas/overview
  - https://docs.cloud.google.com/architecture/landing-zones
  - https://docs.cloud.google.com/architecture/blueprints/security-foundations
unverified: []
---
# B01 Resource hierarchy and billing

## Chunk 1: Organization, folders, projects, resources
Problem it solves: A company with hundreds of projects needs one place to set rules and one tree to reason about access.

Mental model: A tree. The organization is the root and maps to a company domain. Folders group projects under it, up to 10 levels deep. Projects hold resources. Think of a filesystem: org is `/`, folders are directories, projects are the files you work in.

```
Organization (example.com)
├── Folder: shared
│   └── Project: network-host
├── Folder: retail
│   ├── Folder: prod → projects
│   └── Folder: dev  → projects
```

Exam signals: "Central control across business units", "delegate administration per department."

Trap: Flattening everything under the org with no folders. Every policy then needs per-project setup.

Pillar tie-in: Operational excellence and security.

Check: How many levels deep can folders nest?
<details><summary>Answer</summary>Up to 10.</details>

## Chunk 2: Policy inheritance
Problem it solves: Setting the same permission on 400 projects by hand doesn't scale and drifts.

Mental model: Allow policies, deny policies, and organization policies flow down the tree. The effective policy on a resource combines what's set on it with everything inherited from its ancestors. For allow policies, a grant at the folder reaches every project below it. A child can't remove an allow grant it inherited.

Exam signals: "Grant the network team access to every project in the prod folder."

Trap: Granting a broad role at the org to fix one project's access problem. It lands on every project.

Pillar tie-in: Security. Grant at the lowest node that covers the need.

Check: A user holds Viewer on folder `retail`. Can a project owner under `retail` remove that access at the project level?
<details><summary>Answer</summary>No. Allow grants inherit and a lower level can't revoke them. Remove the grant at the folder, or use a deny policy.</details>

## Chunk 3: Identity root: Cloud Identity and Google Workspace
Problem it solves: Google Cloud needs a directory of users and groups before it can have an organization.

Mental model: The organization resource comes from a Google Workspace or Cloud Identity account tied to a domain you verify with a DNS TXT record. Once you sign up, verify the domain, and create a project with that account, Google provisions the organization. Cloud Identity Free covers identity without Workspace apps; it starts with 50 user licenses by default.

Companies that run Active Directory tend to sync it into Cloud Identity rather than managing two directories.

Exam signals: "Company uses Active Directory", "no organization node exists yet", "free trial with no folders."

Trap: Expecting folders and org policies in a personal or trial account. With no organization, those features don't exist.

Pillar tie-in: Security.

Check: What two steps give a company an organization resource?
<details><summary>Answer</summary>Sign up for Cloud Identity or Google Workspace and verify the domain, then create a project with that account. Google provisions the organization.</details>

## Chunk 4: Billing accounts and budgets
Problem it solves: Costs must land on the right cost center, and someone must hear about overspend before the invoice.

Mental model: A Cloud Billing account pays for projects linked to it. One billing account can pay for many projects. A project without an active billing link can't use paid services.

A budget watches spend against an amount and sends alerts at thresholds you set, on actual or forecasted cost. A budget doesn't stop spending. To act, you wire budget notifications to Pub/Sub and trigger automation, for example code that disables billing on a project.

Exam signals: "Alert finance at 90%", "stop a sandbox from running up costs."

Trap: Believing a budget caps spend. It only alerts. Capping needs Pub/Sub plus automation.

Pillar tie-in: Cost.

Check: A sandbox must stop costing money once it hits USD 100. Is a budget enough?
<details><summary>Answer</summary>No. A budget alerts only. Route its notifications to Pub/Sub and trigger code that disables billing or shuts down resources.</details>

## Chunk 5: Labels vs tags
Problem it solves: You need to slice cost by team and enforce policy by environment. One mechanism can't do both.

Mental model:

| | Labels | Tags |
|---|---|---|
| Shape | Key-value on a resource | Key-value defined once at org or project, bound to resources |
| Inheritance | None | Inherited down the hierarchy |
| Use in IAM conditions | No | Yes |
| Use in firewall policies | No | Yes, as sources and targets |
| In billing export | Yes | Yes, including inherited tags |

Labels are annotations for grouping and cost reporting. Tags carry policy weight.

Exam signals: "Grant access only to resources marked prod" points to tags. "Break down cost by team" points to labels, tags, or both.

Trap: Writing an IAM condition on a label. Labels can't drive conditions.

Pillar tie-in: Security and cost.

Check: You want a role binding that applies only to resources in the prod environment. Label or tag?
<details><summary>Answer</summary>Tag. IAM conditions can check tags; labels can't set conditions.</details>

## Chunk 6: Quotas and system limits
Problem it solves: A launch fails because a project hit a ceiling nobody planned for.

Mental model: A quota caps how much of a resource a project can use. Allocation quotas cap how much you hold, such as VM count. Rate quotas cap use over time, such as API calls per minute. You can request quota adjustments. System limits are fixed and can't change. The quota adjuster can watch usage and request increases for you as you approach a quota.

Exam signals: "Scale event next month", "hit a quota error during a spike."

Trap: Treating every ceiling as adjustable. System limits aren't; the design has to change.

Pillar tie-in: Reliability.

Check: What's the difference between a quota and a system limit?
<details><summary>Answer</summary>You can request an adjustment to a quota. A system limit is fixed.</details>

## Chunk 7: Landing zones
Problem it solves: Teams start building before anyone decides the folder tree, network, identity, and guardrails. Retrofitting those later is expensive.

Mental model: A landing zone (also called a cloud foundation) is the baseline of hierarchy, identity, networking, security controls, and logging that every workload lands on. Google's enterprise foundations blueprint describes one: a single organization, folders by environment, Shared VPC networks per environment, and private paths between on-prem and Google Cloud. It ships as Terraform in the `terraform-example-foundation` repository.

Exam signals: "Company is starting its Google Cloud adoption", "consistent governance across teams."

Trap: Letting each team create projects with no folder or policy plan, then trying to impose one.

Pillar tie-in: Operational excellence and security.

Check: A company has 3 business units, each with dev and prod. Sketch a folder layout.
<details><summary>Answer</summary>One option: org → folder per business unit → dev and prod folders under each → projects per app. A shared folder at the top holds network host and logging projects. Environment-first (org → prod/dev → business unit) also works; pick the axis policies differ on most.</details>
