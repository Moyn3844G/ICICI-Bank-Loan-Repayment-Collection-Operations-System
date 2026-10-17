# ICICI Bank Loan Repayment & Collection Operations System

![Industry](https://img.shields.io/badge/Industry-Banking-blue)
![Role](https://img.shields.io/badge/Role-SQL%20Developer-green)
![Runtime](https://img.shields.io/badge/Runtime-Google%20Colab-orange)
![Database](https://img.shields.io/badge/Database-SQLite-lightgrey)

A SQL-led internal control system for retail loan repayment and collection operations at **ICICI Bank**. This project consolidates loan accounts, instalment schedules, payment events, collection actions, and restructuring requests into a normalized SQLite database so operational teams can detect repayment gaps, classify delinquency, prioritize collections, and validate transaction-controlled adjustments.

---

## 📌 Project Overview

| Field | Value |
|-------|-------|
| **Product** | ICICI Bank Loan Repayment & Collection Operations System |
| **Industry** | Banking / FinTech |
| **Primary Role** | SQL Developer |
| **Runtime** | Google Colab (Python 3) |
| **Database** | SQLite (with foreign key enforcement) |
| **Review Date** | 1 August 2026 (deterministic as-of boundary) |
| **Dataset Seed** | 20260812 (reproducible) |

The system enables **Loan Operations** and **Collections Control** teams to:
- Identify missed, mismatched, or late repayments
- Detect duplicate payment references
- Flag stale collection follow-ups
- Surface aged restructuring requests
- Compare branch repayment performance
- Generate a prioritized daily action queue

---

## 🎯 Objectives

1. Create and validate a **normalized relational model** with seven connected core tables and enforced keys.
2. Consume **payment and collection updates** from a provided temporary internal API using `GET`, `POST`, `PUT`, `PATCH`, and `DELETE`.
3. Load validated JSON into SQLite with **constraints, indexes**, and system-managed operational tables.
4. Detect **six meaningful repayment and collection exceptions**.
5. Provide **thirteen operational SQL controls and reports** with deterministic expected outputs.
6. Demonstrate **COMMIT and ROLLBACK** integrity for approved and unapproved ledger adjustments.
7. Design a **seven-step cron-ready** collection-priority workflow and professional engineering change processes.

---

## 🗂️ Data Architecture

### Core Tables (7)

| Table | Primary Key | Purpose |
|-------|-------------|---------|
| `branches` | `branch_id` | Branch / city / region master |
| `customers` | `customer_id` | Borrower master |
| `loans` | `loan_id` | Loan account and pricing attributes |
| `repayment_schedule` | `loan_id + installment_no` | Scheduled EMI obligations **(composite key)** |
| `payments` | `payment_id` | Repayment events linked to a valid schedule row |
| `collection_actions` | `action_id` | Collection follow-up history |
| `restructuring_requests` | `request_id` | Restructuring requests and decisions |

### System-Managed Tables (3)

| Table | Purpose |
|-------|---------|
| `loan_ledger` | Successful repayment receipts + manual adjustments/waivers |
| `loan_exceptions` | Rule-generated operational exceptions and severity |
| `api_ingestion_runs` | Endpoint, row-count, timing, status, and error metadata |

### Dataset Volumes

- **24** branches
- **320** customers
- **420** loans
- **2,940** repayment schedule rows
- **3,089** payments
- **291** collection actions
- **72** restructuring requests

---

## 🔌 Temporary Internal API

A local API is started inside the Colab runtime (no external dependency). The SQL Developer consumes it and does **not** build the service.

| Activity | Method | Purpose | Expected Status |
|----------|--------|---------|-----------------|
| API-Q1 | `GET` | Fetch 5 SUCCESS UPI payments | 200 |
| API-Q2 | `POST` | Create a collection action | 201 |
| API-Q3 | `PUT` | Completely replace a payment record | 200 |
| API-Q4 | `PATCH` | Update selected collection-action fields | 200 |
| API-Q5 | `GET → DELETE → GET` | Verify deletion sequence | 200 → 200 → 404 |

All mutable practice records are reset before use so activities remain **rerunnable**.

---

## 🚨 Business Rules & Exception Engine

Six controlled exception categories are generated:

| Exception | Condition |
|-----------|-----------|
| `MISSED_INSTALLMENT` | Due instalment with no successful payment |
| `PAYMENT_AMOUNT_MISMATCH` | Received amount differs from due by > INR 1 |
| `LATE_PAYMENT` | Successful receipt > 5 days after due date |
| `DUPLICATE_PAYMENT_REFERENCE` | Same reference on multiple successful payments |
| `COLLECTION_FOLLOWUP_OVERDUE` | No collection action in last 10 days |
| `RESTRUCTURING_REQUEST_OVERDUE` | PENDING request older than 14 days |

### Delinquency Stages

- `CURRENT` (0 DPD)
- `DPD_1_30`
- `DPD_31_60`
- `DPD_61_90`
- `DPD_90_PLUS`

---

## 📊 SQL Analyses (Q1–Q13)

| ID | Business Output | SQL Coverage |
|----|-----------------|--------------|
| Q1 | Top 10 largest successful receipts | JOIN, WHERE, ORDER BY, LIMIT |
| Q2 | High-volume branches | GROUP BY, SUM, COUNT, HAVING |
| Q3 | Classify every instalment | CTE, LEFT JOIN, COALESCE, CASE |
| Q4 | Due instalments with no payment | LEFT JOIN, anti-join, NULL |
| Q5 | Overdue loans + open exception | INTERSECT, CTE |
| Q6 | Borrower risk report with DPD | UPPER, SUBSTR, CAST, COALESCE, CASE |
| Q7 | Branches above portfolio average | CTE, scalar subquery |
| Q8 | Top 30 priority overdue loans | Multiple CTEs, numeric calcs |
| Q9 | Rank branches within region | RANK, PARTITION BY |
| Q10 | Running total + 7-day moving average | SUM/AVG window frames |
| Q11 | Day-over-day comparison | LAG, CASE |
| Q12 | First & latest successful repayment | ROW_NUMBER, conditional aggregation |
| Q13 | Top 3 exception ranks per city | DENSE_RANK, PARTITION BY |

---

## 💳 Transaction Management

| Activity | Approved State | Required Proof |
|----------|----------------|----------------|
| **T1 — COMMIT** | INR 750 approved waiver on `L000001` instalment 7 | `LED-DEVELOPER-COMMIT` **remains** after COMMIT |
| **T2 — ROLLBACK** | INR 1,200 unapproved adjustment on `L000002` instalment 7 | `LED-DEVELOPER-ROLLBACK` is **absent** after ROLLBACK |

---

## ⏰ Workflow Automation

**Seven-step cron-ready flow:**

1. Scheduler / cron trigger
2. Internal API call
3. HTTP status and response schema validation
4. Raw JSON storage
5. SQLite loading
6. SQL report generation
7. Logging and monitoring

**AUTO-REPORT** returns the **top 20 due instalments** by outstanding amount, with customer, branch, DPD, delinquency stage, and latest collection-action date.

### Cron Expressions

| Scenario | Expression |
|----------|------------|
| Daily at 8 AM, 12 PM, 4 PM | `0 8,12,16 * * *` |
| Every weekday at 5 PM | `0 17 * * 1-5` |
| Every day at midnight | `0 0 * * *` |
| Every Sunday at 2 AM | `0 2 * * 0` |

> ⚠️ Google Colab cannot act as a persistent scheduler after its runtime stops.

---

## 🛠️ Engineering Practices

**Git Collaboration (8 steps):**  
`Requirement → Feature branch → SQL change → Test/validate → Commit → Push → Pull request/review → Approved merge`

**Safe Database Change (8 steps):**  
`Requirement → Backup/rollback plan → SQL change → Dev test → Peer review → Approved deploy → Monitor → Rollback if failed`

---

## ✅ Progress Dashboard (26 Activities)

The notebook tracks **26 validated activities**:

- **1** — ER design (`ER-1`)
- **5** — API activities (`API-Q1` to `API-Q5`)
- **13** — SQL analyses (`Q1` to `Q13`)
- **2** — Transactions (`T1`, `T2`)
- **3** — Automation (`AUTO-FLOW`, `AUTO-REPORT`, `AUTO-CRON`)
- **2** — Engineering (`ENG-GIT`, `ENG-CHANGE`)

**Expected final result:** `Passed activities: 26/26` ✅

---

## 🚀 How to Run

1. Open the notebook in **Google Colab**.
2. Use **Runtime → Run all**.
3. Wait for **`SETUP COMPLETE`**.
4. The SQL Developer workspaces contain complete SQL solutions.
5. The final cell shows the **progress dashboard** — verify **26/26 Passed**.
6. All outputs, diagrams, and validation logs are visible inline.

---

## 📁 Repository Structure
