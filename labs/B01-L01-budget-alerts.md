# LAB B01-L01: Budget alerts before anything else

## Goal
Create a budget on your sandbox project with alerts at 50%, 90%, and 100% of a small amount, and see where the alerts go.

## What reading can't teach
Who gets the email by default, how the budget appears in the console, and that nothing stops at 100%.

## Cost ceiling
USD 0. Budgets are free. Verified: 2026-09-24.

## Time
15 minutes.

## Prerequisites
- A sandbox project used for nothing else.
- Billing Account Administrator or Billing Account Costs Manager on the billing account. Without it, create the budget in the console as the account owner.
- Cloud Shell open in the sandbox project.
- Your billing account ID (format `XXXXXX-XXXXXX-XXXXXX`), from `gcloud billing projects describe "$PROJECT_ID"`.

## Steps (Cloud Shell)
Replace PROJECT_ID and BILLING_ACCOUNT_ID. Keep real IDs out of any file you commit.

```bash
export PROJECT_ID=PROJECT_ID
export BILLING_ACCOUNT_ID=BILLING_ACCOUNT_ID
gcloud config set project "$PROJECT_ID"
gcloud services enable billingbudgets.googleapis.com
```

1. Create the budget, scoped to the sandbox project.
   ```bash
   gcloud billing budgets create \
     --billing-account="$BILLING_ACCOUNT_ID" \
     --display-name="sandbox-guardrail" \
     --budget-amount=20USD \
     --filter-projects="projects/$PROJECT_ID" \
     --threshold-rule=percent=0.50 \
     --threshold-rule=percent=0.90 \
     --threshold-rule=percent=1.00 \
     --threshold-rule=percent=1.00,basis=forecasted-spend
   ```
   Use the currency of your billing account in `--budget-amount`.
2. List it.
   ```bash
   gcloud billing budgets list --billing-account="$BILLING_ACCOUNT_ID" \
     --format="table(displayName,amount.specifiedAmount.units,thresholdRules)"
   ```
3. In the console, open Billing → Budgets & alerts and find `sandbox-guardrail`. Note who receives alert emails.

## Expected output
Step 1 prints the new budget's resource name. Step 2 shows `sandbox-guardrail` with amount 20 and four threshold rules. The console shows current spend against the budget.

What to notice: the budget has no "stop" action. At 100% you get an email and spending continues. Capping needs Pub/Sub notifications plus automation (B01 chunk 4).

## Teardown
Keep this budget for the rest of the course. It's your safety net. To remove it later:

```bash
gcloud billing budgets list --billing-account="$BILLING_ACCOUNT_ID" --format="value(name)"
gcloud billing budgets delete BUDGET_NAME --billing-account="$BILLING_ACCOUNT_ID"
```

## Fallback: delete the project
Nothing billable exists in this lab. If you created a project for it and no longer need it:

```bash
gcloud projects delete "$PROJECT_ID"
```
