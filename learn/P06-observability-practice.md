---
block: P06
title: Observability in practice
pillars: [operational-excellence, reliability]
exam_guide_refs: ["6.2", "1.1", "6.1"]
last_verified: 2026-09-24
sources:
  - https://docs.cloud.google.com/logging/docs/view/logging-query-language
  - https://docs.cloud.google.com/logging/docs/logs-based-metrics
  - https://docs.cloud.google.com/logging/docs/routing/overview
  - https://docs.cloud.google.com/monitoring/alerts
  - https://docs.cloud.google.com/monitoring/uptime-checks
  - https://docs.cloud.google.com/monitoring/support/notification-options
  - https://docs.cloud.google.com/error-reporting/docs
  - https://docs.cloud.google.com/trace/docs/overview
  - https://docs.cloud.google.com/stackdriver/docs/managed-prometheus
  - https://docs.cloud.google.com/stackdriver/docs/solutions/slo-monitoring/alerting-on-budget-burn-rate
  - https://docs.cloud.google.com/profiler/docs/about-profiler
unverified: []
---
# P06 Observability in practice

## Chunk 1: Logging query language
Problem it solves: Find the five lines that matter among millions.

Mental model: Queries are field comparisons joined with `AND`, `OR`, `NOT`. Lines on separate rows are joined with `AND`. `=` matches the whole value, `:` matches substrings, `=~` matches a regex.

```
resource.type="cloud_run_revision"
severity>=ERROR
timestamp>="2026-09-24T00:00:00Z"
textPayload:"timeout"
```

Exam signals: "Find all errors from one service in the last hour."

Trap: Scrolling the logs explorer by time instead of filtering by resource and severity.

Pillar tie-in: Operational excellence.

Check: Write a filter for ERROR or worse logs from Cloud Run revisions.
<details><summary>Answer</summary>`resource.type="cloud_run_revision" AND severity>=ERROR`</details>

## Chunk 2: Log-based metrics and sinks
Problem it solves: Turn log events into numbers you can chart and alert on, and keep logs where other teams need them.

Mental model: A log-based metric counts matching entries (counter) or extracts a value into a distribution. Chart it and alert on it in Cloud Monitoring. Sinks route matching logs to a log bucket, BigQuery (for SQL analysis), Cloud Storage (for cheap long-term retention), or Pub/Sub (for streaming to other tools).

Exam signals: "Alert when 'payment failed' appears more than 10 times a minute" → counter log-based metric plus alert. "Analyze a year of logs with SQL" → sink to BigQuery.

Trap: Exporting logs to BigQuery just to count one message pattern.

Pillar tie-in: Operational excellence.

Check: Which log-based metric type holds values such as latency pulled from log lines?
<details><summary>Answer</summary>A distribution metric.</details>

## Chunk 3: Alerting policies and notification channels
Problem it solves: The right person hears about the right problem, once.

Mental model: An alerting policy has conditions (metric threshold, metric absence, or log match), notification channels (email, SMS, PagerDuty, Slack, webhooks, Pub/Sub), and documentation that appears in the notification. When a condition is met, Monitoring opens an incident and notifies the channels.

Exam signals: "Alerts go to email and are ignored" (EHR, Altostrat) → route to on-call tooling and cut noise.

Trap: Alerts on every CPU spike, sent to a shared inbox.

Pillar tie-in: Operational excellence.

Check: What three parts make up an alerting policy?
<details><summary>Answer</summary>Conditions, notification channels, and documentation.</details>

## Chunk 4: Uptime checks, Error Reporting, Trace
Problem it solves: Know about outages before users do, and find which code or hop is at fault.

Mental model: Public uptime checks probe a URL from several locations worldwide; private uptime checks cover internal endpoints. Error Reporting groups application errors and flags new ones. Cloud Trace shows distributed request latency across services.

Exam signals: "Detect the site is down from outside" → uptime check. "New exception type after a deploy" → Error Reporting.

Trap: Only internal metrics, no external probe. A DNS or certificate failure looks healthy from inside.

Pillar tie-in: Reliability.

Check: What catches an expired TLS certificate that internal metrics miss?
<details><summary>Answer</summary>A public uptime check against the HTTPS URL.</details>

## Chunk 5: Managed Service for Prometheus
Problem it solves: Teams run Prometheus at scale, and its storage and federation become a job of their own.

Mental model: Google Cloud Managed Service for Prometheus collects Prometheus and OpenTelemetry metrics and stores them in Google's backend. You keep PromQL and existing exporters; you drop the job of running and scaling Prometheus servers. It works across projects and clouds.

Exam signals: "Existing Prometheus dashboards and alerts" (Altostrat), "global view across clusters."

Trap: Rewriting every Prometheus alert as a Cloud Monitoring alert to migrate.

Pillar tie-in: Operational excellence.

Check: What query language do you keep with Managed Service for Prometheus?
<details><summary>Answer</summary>PromQL.</details>

## Chunk 6: SLO-based alerting and alert fatigue
Problem it solves: Pages that wake people for blips, and silence during slow bleeds.

Mental model: Alert on error budget burn rate. A fast-burn alert (high burn rate over a short lookback) pages; a slow-burn alert (lower rate over a longer lookback) opens a ticket. Burn rate of 1 uses the budget over the full compliance period; a burn rate of 10 would use it 10 times faster.

Exam signals: "Reduce alert noise while catching real incidents."

Trap: Static thresholds on error counts that fire at 3 a.m. for 2 failed requests.

Pillar tie-in: Reliability and operational excellence.

Check: A burn rate of 1 means what?
<details><summary>Answer</summary>You'd use the whole error budget by the end of the compliance period.</details>

## Chunk 7: Profiling (exam guide 6.2)
Problem it solves: CPU or memory cost is high and nobody knows which function spends it.

Mental model: Cloud Profiler samples, without pause, CPU and memory use of production code with low overhead and shows which functions consume the most. Use it for benchmarking and tuning before you add machines.

Exam signals: "Identify which code paths drive CPU cost in production."

Trap: Scaling up instances to fix a slow function.

Pillar tie-in: Performance and cost.

Check: Which tool shows which functions use the most CPU in production?
<details><summary>Answer</summary>Cloud Profiler.</details>
