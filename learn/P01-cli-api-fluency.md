---
block: P01
title: CLI and API fluency
pillars: [operational-excellence, security]
exam_guide_refs: ["5.2"]
last_verified: 2026-09-24
sources:
  - https://docs.cloud.google.com/sdk/docs/scripting-gcloud
  - https://docs.cloud.google.com/sdk/gcloud/reference/topic/formats
  - https://docs.cloud.google.com/sdk/gcloud/reference/topic/filters
  - https://docs.cloud.google.com/docs/authentication/application-default-credentials
  - https://docs.cloud.google.com/docs/authentication/use-service-account-impersonation
  - https://docs.cloud.google.com/storage/docs/gsutil-transition-to-gcloud
  - https://docs.cloud.google.com/shell/docs/how-cloud-shell-works
unverified: []
---
# P01 CLI and API fluency

## Chunk 1: gcloud configurations
Problem it solves: You work across several projects and accounts. Retyping `--project` on every command invites the one typo that deletes the wrong thing.

Mental model: A named configuration is a profile: account, project, default region and zone. Think of it as a set of shell environment variables you switch as a unit.

```bash
gcloud config configurations create dev
gcloud config set project PROJECT_ID
gcloud config set compute/region us-central1
gcloud config configurations activate dev
gcloud config configurations list
```

A `--project` flag on one command overrides the active configuration for that command only.

Exam signals: "Engineers work across many projects", "avoid running commands against production by mistake."

Trap: Relying on the active configuration inside scripts. Scripts should pass `--project` so they don't depend on whoever ran `activate` last.

Pillar tie-in: Operational excellence. Explicit targets make runs repeatable.

Check: A script ran against prod when you meant dev. What change makes that impossible to repeat?
<details><summary>Answer</summary>Pass `--project` in the script, or activate a named configuration at the top of the script. Don't depend on the shell's current active configuration.</details>

## Chunk 2: --format, --filter, projections
Problem it solves: Default gcloud output is for humans. Scripts need stable, parseable output.

Mental model: `--filter` picks which resources come back. `--format` picks how they print. A projection inside `--format` picks which fields.

| Need | Flag |
|---|---|
| Only running VMs | `--filter="status=RUNNING"` |
| One value for a variable | `--format="value(name)"` |
| CSV for a spreadsheet | `--format="csv(name,zone.basename(),status)"` |
| Full detail for jq | `--format=json` |

Supported formats include csv, json, table, value, and yaml. Run `gcloud topic formats`, `gcloud topic filters`, and `gcloud topic projections` for the full grammar.

Exam signals: "Script", "report", "automate a list of resources."

Trap: Piping table output through `grep` and `awk`. Column widths change and the script breaks.

Pillar tie-in: Operational excellence.

Check: You need just the names of stopped VMs, one per line, to feed a loop. Which two flags?
<details><summary>Answer</summary>`--filter="status=TERMINATED"` and `--format="value(name)"`.</details>

## Chunk 3: APIs must be enabled
Problem it solves: A command fails with a permission-looking error when the real cause is a disabled API.

Mental model: Each Google Cloud service is an API you switch on per project, like loading a kernel module before you use the device. A new project has few APIs on.

```bash
gcloud services list --enabled
gcloud services enable compute.googleapis.com
```

Enabling an API needs a role that holds `serviceusage.services.enable`, such as Service Usage Admin.

Exam signals: "Error says the API has not been used in project" or "is disabled".

Trap: Granting the user a bigger IAM role when the API is off. More permissions won't turn the API on.

Pillar tie-in: Security. Only the APIs a project needs stay on, which shrinks the attack surface.

Check: A new project returns "API not enabled" for `gcloud compute instances list`. What do you run?
<details><summary>Answer</summary>`gcloud services enable compute.googleapis.com` in that project, with an account allowed to enable services.</details>

## Chunk 4: Credentials: user, ADC, impersonation
Problem it solves: gcloud and your code look for credentials in different places. Mixing them up causes "works in my shell, fails in my script."

Mental model:

| Credential | Used by | Set with |
|---|---|---|
| gcloud user credentials | gcloud commands | `gcloud auth login` |
| Application Default Credentials (ADC) | Client libraries and tools like Terraform | `gcloud auth application-default login`, or an attached service account |
| Impersonation | Either, acting as a service account for a short time | `--impersonate-service-account=SA_EMAIL` |

ADC search order:
1. The file named in `GOOGLE_APPLICATION_CREDENTIALS`.
2. The local file written by `gcloud auth application-default login`.
3. The metadata server, which serves the attached service account on Google Cloud compute.

Impersonation needs the Service Account Token Creator role on the target service account. The credentials it creates are short-lived and never land on disk as a key.

Exam signals: "Avoid service account keys", "developer needs to test with production permissions temporarily."

Trap: Downloading a service account key so a laptop script can run. Impersonation does the same job without a long-lived secret.

Pillar tie-in: Security.

Check: A developer runs `gcloud auth login` and their Python script still fails to authenticate. Why?
<details><summary>Answer</summary>`gcloud auth login` sets gcloud's own credentials. Client libraries read ADC. Run `gcloud auth application-default login`, with impersonation where possible.</details>

## Chunk 5: gcloud storage, bq, kubectl
Problem it solves: Three services ship their own CLI conventions. You need to know which tool to reach for.

Mental model:

| Tool | For | Note |
|---|---|---|
| `gcloud storage` | Cloud Storage | Google's recommended CLI for Cloud Storage |
| `gsutil` | Cloud Storage | Legacy, minimal maintenance; lacks newer features such as soft delete and managed folders |
| `bq` | BigQuery | Queries, datasets, loads |
| `kubectl` | GKE workloads | Get credentials first with `gcloud container clusters get-credentials` |

Google plans to remove gsutil from the Google Cloud CLI install package after March 2027.

Exam signals: "Copy large files faster" points to `gcloud storage`. The exam guide still names gsutil under section 5.2, so recognize it.

Trap: Writing new automation on gsutil.

Pillar tie-in: Performance. `gcloud storage` picks parallel transfer settings without extra flags.

Check: Which tool do you use for a new script that syncs a directory to a bucket, and why?
<details><summary>Answer</summary>`gcloud storage rsync`. gsutil is legacy, receives minimal maintenance, and is leaving the default install.</details>

## Chunk 6: Release tracks and Cloud Shell limits
Problem it solves: Some features exist only under `gcloud alpha` or `gcloud beta`, and Cloud Shell isn't a server.

Mental model: `gcloud alpha` and `gcloud beta` expose pre-GA surfaces. They can change without notice. Production automation should stick to GA commands where one exists.

Cloud Shell gives you an ephemeral VM with 5 GB of persistent `$HOME`. The VM stops after 40 minutes idle and sessions end after 12 hours. Anything outside `$HOME` resets.

For long work or local tooling, install the Google Cloud CLI on your own machine.

Exam signals: "Needs a feature only available in preview" points to beta or alpha. "Must be supported in production" points to GA.

Trap: Building a production pipeline on an alpha command.

Pillar tie-in: Reliability.

Check: Why shouldn't a nightly job run from Cloud Shell?
<details><summary>Answer</summary>Cloud Shell's VM stops when idle and sessions cap at 12 hours. It's a workstation, not a scheduler. Use Cloud Scheduler with Cloud Run jobs or a CI system.</details>
