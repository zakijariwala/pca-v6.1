# LAB B00-L00: <title>

## Goal
<One sentence: what you build and what you observe.>

## What reading can't teach
<The behavior you only see by running it: an error, a latency, a default, a failure mode.>

## Cost ceiling
<Max expected spend in USD, with the billable resources listed. Verified: YYYY-MM-DD.>

## Time
<Minutes, including teardown.>

## Prerequisites
- A sandbox project used for nothing else. Never run labs in a project that holds real workloads.
- A budget with an alert set at or below the cost ceiling on the sandbox project's billing account.
- Cloud Shell open in that project.
- <APIs to enable, roles needed.>

## Steps (Cloud Shell)
Replace PROJECT_ID with your sandbox project ID. Keep real IDs out of any file you commit.

```bash
export PROJECT_ID=PROJECT_ID
export REGION=<region>
gcloud config set project "$PROJECT_ID"
```

1. <Step.>
   ```bash
   <command using $PROJECT_ID>
   ```

## Expected output
<What each step prints or shows in the console. Name the field or line to check.>

## Teardown
Run every step. Then confirm nothing billable remains.

```bash
<delete commands, reverse order of creation>
```

## Fallback: delete the project
If teardown fails or you lose track of resources, delete the whole sandbox project. This shuts down all billing in it and schedules the project for deletion.

```bash
gcloud projects delete "$PROJECT_ID"
```
