# RavenStack: B2B SaaS Churn & Subscription Analytics

## One‑Line Pitch
Proactively protecting Annual Recurring Revenue (ARR) by surfacing high‑risk accounts, linking product quality and support velocity directly to customer churn.

---

## The Business Problem
RavenStack, a stealth‑mode B2B SaaS startup, faced a “leaky bucket” scenario: rapid customer acquisition but equally rapid churn. Revenue growth was fragile because:
- Operational churn risks were hidden.
- Product bugs and system exceptions degraded user experience.
- Support bottlenecks slowed resolution and drove cancellations.

**Objective:** Build a complete, enterprise‑grade Power BI solution to:
1. **Monitor Financial Health** – Track MRR, ARR, NRR, GRR.
2. **Diagnose Friction** – Correlate product bugs and support delays with churn.
3. **Drive Action** – Generate an automated “Accounts at Risk” queue for proactive intervention.

---

## Data Architecture & ETL

**Dataset:** Synthetic relational database, 33,100 event logs across 5 tables.

- **Accounts (500)** – Customer demographics, plan tier, churn flag.
- **Subscriptions (5,000)** – MRR/ARR, upgrades/downgrades, churn status.
- **Feature Usage (25,000)** – Sessions, duration, errors, beta flags.
- **Support Tickets (2,000)** – SLA times, priority, satisfaction, escalations.
- **Churn Events (600)** – Cancellation reasons, refunds, feedback.

**ETL with Power Query:**
- Text sanitization (Text.Trim, Text.Clean).
- Null preservation for unresolved tickets.
- Binary normalization of Yes/No flags.

**Star Schema Model:**
- Central **Accounts** dimension.
- Fact tables: Subscriptions, Feature Usage, Support Tickets, Churn Events.
- Strict 1‑to‑Many relationships, single filter direction.
- Central **DateTable = CALENDARAUTO()** for unified time intelligence.

---

## The Analytics Engine (DAX Highlights)

### 1. Predictive Risk Model
Penalty‑based scoring system combining product instability and support friction:
- >50 errors → +40 points
- >5 tickets → +30 points
- CSAT <3 stars → +30 points

```dax
Predictive Risk Score = 
VAR TotalErrors = SUM(feature_usage[error_count])
VAR TotalTickets = COUNT(support_tickets[ticket_id])
VAR AvgSatis = AVERAGE(support_tickets[satisfaction_score])
VAR ErrorPenalty   = IF(TotalErrors > 50, 40, 0)
VAR TicketPenalty  = IF(TotalTickets > 5, 30, 0)
VAR CSATPenalty    = IF(NOT(ISBLANK(AvgSatis)) && AvgSatis < 3.0, 30, 0)
RETURN ErrorPenalty + TicketPenalty + CSATPenalty

```
Dynamic context transition ensures accurate account counts:
```
dax
Accounts at Risk =
COUNTROWS(
    FILTER(VALUES(accounts[account_id]), [Predictive Risk Score] >= 70)
)
```
---
### 2. User Journey Funnel
Drop‑off rates from sign‑up to retention:
```dax
SignUps = DISTINCTCOUNT(accounts[account_id])
ActiveUsers = CALCULATE(DISTINCTCOUNT(accounts[account_id]), FILTER(feature_usage, feature_usage[usage_count] > 0))
SupportAccounts = CALCULATE(DISTINCTCOUNT(accounts[account_id]), FILTER(support_tickets, support_tickets[ticket_id] <> BLANK()))
RetainedAccounts = CALCULATE(DISTINCTCOUNT(accounts[account_id]), accounts[churn_flag] = FALSE)
```
Conversions:

Sign‑Up → Active

Active → Support

Support → Retained

---

### 3. Core SaaS Financials
```
dax
Active_mrr = CALCULATE(SUM(subscriptions[mrr_amount]), accounts[churn_flag] = FALSE())
Net Revenue Retention % = DIVIDE([mrr] + [upgradedmrr] - [downgradedmrr] - [churnedmrr], [mrr], 0)
```
---

## Dashboard Suite & Insights
### 1. Executive Dashboard (Financial Health)
KPIs: $8.98M Active MRR | $107.8M ARR | 96.09% NRR | 22% Logo Churn.

Insight: High NRR but high logo churn → dependency on Enterprise expansions, while Basic/Pro quietly bleed out.

Action: Launch small‑account retention programs.

---

### 2. Product Engagement Dashboard
KPIs: 251K sessions | 14K errors.

Insight: Funnel bottleneck → 98.4% of users forced into support, retention drops to 78%. Feature_26 & feature_9 unstable.

Action: Patch error‑prone modules, improve onboarding/self‑serve.

---

### 3. Customer Stability & Churn Predictor
KPIs: Avg Risk Score 70 | Resolution Time 35.86 hrs | Escalation 4.75%.

Insight: Product instability + slow support → predictable churn.

Action: Use “Accounts at Risk” table for proactive save‑calls.

---

## Execution Playbook (FY2026)
Engineering Triage: Halt new features, hotfix feature_26 & feature_32.

Support SLA: Reduce resolution time from 35.8 hrs → <12 hrs.

Automated Outreach: Integrate Predictive Risk Score into CRM alerts.

---

## Dataset Credit
Synthetic B2B SaaS dataset inspired by River @ Rivalytics.