# Lab M2L3 — Monitoring Plan and Maintenance Checklist

> **System:** QuickKart — e-commerce platform, launched two weeks ago  
> **What it does:** Processes customer orders, payments, and inventory in real time  
> **Audience:** DevOps and engineering teams, for incident handling and ongoing upkeep  
> **My role:** Systems Professional — define measurable monitoring goals, recommend alerting tools and escalation paths, plan recurring maintenance, and document incident response so the platform stays stable as it scales.

---

## Purpose

QuickKart is live and handling real money, so "it works" is no longer enough — it has to *keep* working and we have to *know* the moment it doesn't. This document gives DevOps and engineering a single reference for two things:

1. **What healthy looks like** — the metrics we watch, the thresholds that define normal, and how we get alerted when reality drifts out of range.
2. **What keeps it healthy** — the recurring maintenance that prevents slow-burn failures, and the incident workflow we follow when something breaks anyway.

---

# Section 1 — Monitoring plan

## 1. Critical system metrics and thresholds

These seven metrics cover availability, performance, capacity, and the revenue-critical paths (checkout and payments) that make QuickKart an e-commerce platform rather than a generic web app.

| **#** | **Metric** | **Threshold (alert if…)** | **Why it matters for QuickKart** |
| --- | --- | --- | --- |
| 1 | **Uptime / availability** | < **99.9%** monthly (≈ > 43 min downtime/month) | Every minute of downtime is lost orders and eroded trust on a two-week-old brand. |
| 2 | **API response time (p95 latency)** | Checkout & payment APIs **> 500 ms** p95 over 5 min | Slow checkout is the #1 driver of cart abandonment; p95 (not average) catches the tail users actually feel. |
| 3 | **Error rate (5xx)** | > **2%** of requests per hour | A rising 5xx rate is the earliest sign of a backend fault before customers start complaining. |
| 4 | **Payment success rate** | < **98%** of attempted transactions | Directly ties to revenue; a dip often means a payment-gateway or integration failure, not a code bug. |
| 5 | **CPU / memory usage** | > **80%** sustained for 10 min | Saturation precedes latency spikes and crashes; the warning window lets us scale before users notice. |
| 6 | **Database connection pool utilization** | > **85%** of max connections | Pool exhaustion stalls orders and inventory writes — a silent killer under traffic spikes (e.g., a promo). |
| 7 | **Log warnings / error-log volume** | Sudden spike (e.g., **> 3×** the rolling hourly baseline) | A surge in `WARN`/`ERROR` entries surfaces problems that haven't yet tripped a hard threshold. |

> **Note on thresholds:** these are starting values for a freshly launched system. As we collect two–four weeks of real baseline data, the Systems Professional revisits each threshold so alerts reflect *actual* normal traffic and we avoid both blind spots and false alarms.

## 2. Monitoring tools and alerting logic

### Tooling

| **Tool** | **What it watches** | **Role in the stack** |
| --- | --- | --- |
| **Datadog (APM)** | API response times, error rates, payment success rate, request traces | Primary application performance monitoring and end-to-end transaction tracing |
| **AWS CloudWatch** | Infrastructure metrics (CPU, memory), uptime checks, log ingestion | Source of truth for host/infra health and centralized logs |
| **Grafana** | Unified dashboards over Prometheus + CloudWatch metrics | Single-pane visualization for on-call and for the daily health review |

### Alerting logic

- **How alerts fire:** Each metric has a defined threshold (Section 1.1). When a metric crosses its threshold for the stated duration, the monitoring tool raises an alert tagged with a **severity** (High / Medium / Low) and the affected service.
- **Where alerts go:**
  - **Low / Medium** → posted to the **`#quickkart-alerts` Slack channel** for the on-call engineer to triage during working hours.
  - **High** → routed to **PagerDuty**, which pages the on-call DevOps/SRE engineer immediately, 24/7, *and* mirrors a notice to Slack for team visibility.
- **De-duplication & noise control:** alerts for the same root cause are grouped, and transient blips below the duration window are suppressed so the team isn't trained to ignore pages (avoiding alert fatigue).

### Escalation protocol

| **Severity** | **Example trigger** | **First action** | **Escalates to (if unresolved)** |
| --- | --- | --- | --- |
| **High** | Site down, payments failing, error rate > 2% | PagerDuty pages on-call **immediately** | Engineering Lead at **15 min**, then Engineering Manager at **30 min** |
| **Medium** | p95 latency > 500 ms, CPU > 80% sustained | Slack alert; on-call investigates within **30 min** | On-call escalates to a senior engineer if not understood within **1 hour** |
| **Low** | Log-warning spike, pool utilization > 85% | Slack alert; reviewed in the **daily health check** | Raised to Medium if the trend worsens |

---

# Section 2 — Maintenance checklist

## 1. Recurring maintenance schedule

Routine upkeep catches the slow-moving problems (disk growth, untested backups, missing patches) that monitoring thresholds alone won't flag until they become incidents.

| **Frequency** | **Task** | **Owner** | **Why** |
| --- | --- | --- | --- |
| **Daily** | Review application & error logs for new patterns | On-call DevOps | Catch emerging issues before they trip a threshold |
| **Daily** | Confirm automated backups completed successfully | DevOps | A failed backup is only dangerous if it goes unnoticed |
| **Daily** | Monitor resource usage (CPU, memory, disk) on the dashboard | On-call DevOps | Spot capacity trends early |
| **Weekly** | Check database growth & query performance | DBA / Backend | Inventory and order tables grow fast post-launch |
| **Weekly** | Review the past week's alert history | Systems Professional | Tune noisy thresholds; confirm real issues were actioned |
| **Weekly** | Test backup **restoration** in a staging environment | DevOps | A backup that can't be restored isn't a backup |
| **Monthly** | Apply OS / dependency **security patches** | DevOps / Security | Reduce exposure to known vulnerabilities |
| **Monthly** | Review usage & traffic trends, re-baseline thresholds | Systems Professional | Keep alerts aligned with real load as QuickKart grows |
| **Monthly** | Archive old logs and prune stale data | DevOps | Control storage cost and keep dashboards responsive |

## 2. Incident response workflow

When monitoring or maintenance surfaces a problem, the team follows this workflow so response is consistent and nothing is lost.

### Severity levels

| **Level** | **Definition** | **Example** | **Target response** |
| --- | --- | --- | --- |
| **High** | Customer-facing outage or revenue loss | Checkout down, payments failing, full outage | Acknowledge **< 15 min**, all-hands until resolved |
| **Medium** | Degraded performance, no full outage | Slow page loads, intermittent 5xx errors | Acknowledge **< 1 hour**, fix within the day |
| **Low** | Minor issue, no customer impact yet | Log-warning spike, single non-critical service flapping | Acknowledge **same day**, schedule a fix |

### Workflow steps

1. **Detect & alert** — Datadog / CloudWatch raises an alert; severity routes it to Slack or PagerDuty (Section 1.2).
2. **Acknowledge** — the **first responder (on-call DevOps / SRE)** acknowledges the page within the target time, owning the incident until handoff or resolution.
3. **Triage & communicate** — open a tracking ticket in **Jira (or ServiceNow)**; post a status thread in **Slack `#quickkart-incidents`**; for High incidents, notify stakeholders of impact and ETA.
4. **Mitigate & resolve** — apply a fix or rollback, restore service, and confirm via the dashboards that the metric is back in range.
5. **Escalate if needed** — if not resolved within the severity's window, escalate per the protocol in Section 1.2.
6. **Post-mortem (High & repeat Medium incidents)** — within **48 hours**, document a blameless post-mortem: timeline, root cause, customer impact, what fixed it, and **action items** to prevent recurrence. Store it in the shared incident log and review action items in the weekly alert review.
