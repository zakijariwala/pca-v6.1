---
block: P02
title: Terraform on GCP foundations
pillars: [operational-excellence, reliability]
exam_guide_refs: ["5.2", "2.3", "4.1"]
last_verified: 2026-09-24
sources:
  - https://docs.cloud.google.com/docs/terraform/authentication
  - https://docs.cloud.google.com/docs/terraform/resource-management/store-state
  - https://docs.cloud.google.com/docs/terraform/resource-management/import
  - https://cloud.google.com/docs/terraform/best-practices/security
  - https://cloud.google.com/blog/topics/developers-practitioners/using-google-cloud-service-account-impersonation-your-terraform-code
  - https://cloud.google.com/blog/products/devops-sre/using-the-cloud-foundation-toolkit-with-terraform
unverified: []
---
# P02 Terraform on GCP foundations

## Chunk 1: Why IaC, and the plan/apply loop
Problem it solves: Console-built infrastructure can't be reviewed, repeated, or rebuilt after a mistake.

Mental model: Terraform compares code (desired state) with the state file (what it built) and the real cloud. `plan` shows the diff. `apply` makes it real. `destroy` removes it. It's configuration management for cloud resources, like Ansible for servers but declarative.

```bash
terraform init
terraform plan -out=tfplan
terraform apply tfplan
terraform destroy
```

Exam signals: "Repeatable environments", "review infrastructure changes before they happen."

Trap: Running `apply` without saving and reviewing a plan.

Pillar tie-in: Operational excellence.

Check: What does `plan` compare?
<details><summary>Answer</summary>The configuration against the state file and the refreshed real infrastructure, and it shows what `apply` would change.</details>

## Chunk 2: google vs google-beta providers
Problem it solves: Some resource arguments exist only for preview features.

Mental model: The `google` provider covers GA features. The `google-beta` provider adds pre-GA features. You can use both in one configuration and pick per resource with the `provider` argument.

Exam signals: "Feature is in preview."

Trap: Switching the whole configuration to `google-beta` for one field. Scope it to the resource that needs it.

Pillar tie-in: Reliability.

Check: One resource needs a preview-only field. What's the narrow fix?
<details><summary>Answer</summary>Declare `google-beta` and set `provider = google-beta` on that resource only.</details>

## Chunk 3: Remote state in Cloud Storage
Problem it solves: A state file on a laptop gets lost, and two people applying at once corrupt it.

Mental model: The `gcs` backend stores state in a Cloud Storage bucket and locks it during operations, so two applies can't collide. Turn on object versioning so you can recover an earlier state. Restrict the bucket to the build system and a few admins: state can hold secrets in plain text.

```hcl
terraform {
  backend "gcs" {
    bucket = "BUCKET_NAME"
    prefix = "network"
  }
}
```

Exam signals: "Team collaboration on Terraform", "recover from a bad state."

Trap: Committing `terraform.tfstate` to Git.

Pillar tie-in: Reliability and security.

Check: Why turn on versioning for the state bucket?
<details><summary>Answer</summary>To restore a previous state version after corruption or a bad write.</details>

## Chunk 4: Authentication without keys
Problem it solves: Terraform needs broad permissions. A downloaded key with those permissions is a prime target.

Mental model: Create a dedicated service account for Terraform with only the roles it needs. Grant your user the Service Account Token Creator role on it. Then impersonate it: set `impersonate_service_account` in the provider block, and in the `gcs` backend block for state access. Your user account needs no direct access to the state bucket.

```hcl
provider "google" {
  project                     = "PROJECT_ID"
  impersonate_service_account = "terraform@PROJECT_ID.iam.gserviceaccount.com"
}
```

Exam signals: "Least privilege for IaC", "no service account keys."

Trap: `credentials = file("key.json")` in the provider block.

Pillar tie-in: Security.

Check: What role must your user hold on the Terraform service account to impersonate it?
<details><summary>Answer</summary>Service Account Token Creator.</details>

## Chunk 5: Variables, outputs, modules
Problem it solves: Copy-pasting the same network code for dev and prod drifts them apart.

Mental model: Variables are inputs, outputs are return values, and a module is a function: a folder of Terraform you call with inputs. Call one network module twice with different variables for dev and prod. Google's Cloud Foundation Toolkit publishes modules for common building blocks.

Exam signals: "Standardize infrastructure across teams", "reuse."

Trap: One giant root configuration for every environment with `count` tricks to toggle pieces.

Pillar tie-in: Operational excellence.

Check: What's the Terraform equivalent of a function with arguments?
<details><summary>Answer</summary>A module called with input variables.</details>

## Chunk 6: Importing and drift
Problem it solves: Resources built by hand before Terraform arrived, or changed by hand after.

Mental model: Import brings an existing resource under Terraform's management. You can run `terraform import` one resource at a time, or, with Terraform 1.5 and later, write `import` blocks that `plan` previews and `apply` executes. Drift is when the real resource differs from state because someone changed it outside Terraform. `plan` reveals drift; `apply` reverts it to code.

Resources inside a module each need their own import at their module address.

Exam signals: "Existing resources must be managed as code", "someone changed a firewall rule in the console."

Trap: Deleting and recreating a production resource to get it under Terraform.

Pillar tie-in: Operational excellence and reliability.

Check: An engineer opened port 22 in the console on a Terraform-managed firewall rule. What happens at the next `apply`?
<details><summary>Answer</summary>`plan` shows the drift and `apply` reverts the rule to what the code says, closing port 22.</details>
