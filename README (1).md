# BookLeaf Publishing — Author Royalty Management & Automation Portal

**Assessment 2** | No-Code Application Development with Workflow Automation

[![Bubble.io](https://img.shields.io/badge/Bubble.io-Free%20Plan-7B61FF?style=flat-square)](https://apandey8957.bubbleapps.io/version-test/)
[![n8n](https://img.shields.io/badge/n8n-Workflow%20Automation-EA4B71?style=flat-square)](https://n8n.io)

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Problem Statement](#2-problem-statement)
3. [Project Objectives](#3-project-objectives)
4. [Technology Stack](#4-technology-stack)
5. [Application Access](#5-application-access)
6. [Test Account Information](#6-test-account-information)
7. [Application Features](#7-application-features)
8. [Database Structure](#8-database-structure)
9. [n8n Workflow Documentation](#9-n8n-workflow-documentation)
10. [Bubble and n8n Integration](#10-bubble-and-n8n-integration)
11. [Setup and Configuration](#11-setup-and-configuration)
12. [Testing and Validation](#12-testing-and-validation)
13. [Demo Video](#13-demo-video)
14. [Assumptions and Limitations](#14-assumptions-and-limitations)
15. [Project Completion Summary](#15-project-completion-summary)

---

## 1. Project Overview

BookLeaf Publishing – Author Royalty Management Portal is a web-based application built with **Bubble.io** for the front-end and database, and **n8n** for backend workflow automation.

The application gives BookLeaf Publishing a single platform to manage all 10 authors, their 18 books, royalty earnings, payout history, and automated royalty summary notifications. It has two separate access roles — **Author** and **Admin** — each with their own dashboard and permissions.

The notification feature uses an asynchronous architecture: Bubble creates a notification log entry, sends a POST request to an n8n webhook, and n8n runs an **automated pipeline** that validates the request, calculates royalty totals, generates a personalised royalty summary email, then calls back to Bubble's API to record the result.

---

## 2. Problem Statement

Publishing companies working with many authors face a recurring challenge: keeping each author accurately informed about their royalty status. Manually calculating, compiling, and sending individual royalty summaries is time-consuming and prone to error.

BookLeaf Publishing needed a centralised system where:

- Authors can self-serve and check their own royalty information at any time.
- Admins can view all author and book data from a single dashboard.
- Royalty summary notifications can be generated and sent — individually or in bulk — with minimal manual effort.
- Every notification attempt is logged with its full lifecycle status for audit and troubleshooting.

---

## 3. Project Objectives

- Build a role-based portal for authors and admins using Bubble.io.
- Store and display accurate royalty data for 10 authors across 18 books.
- Implement an asynchronous notification system using n8n webhooks.
- Log every notification attempt with its status (Processing → Success / Failed).
- Prevent duplicate notifications using processing-state guards on both Bubble and n8n sides.
- Provide admins with a bulk notification option that targets only authors with pending royalties.
- Ensure all royalty calculations match the source data exactly.

---

## 4. Technology Stack

| Technology | Role | Details |
|---|---|---|
| **Bubble.io** | Application platform | All UI pages, page workflows, database, privacy rules. Free plan, test version. |
| **n8n** | Workflow automation | Receives webhooks from Bubble, runs the summary pipeline, callbacks to Bubble. |
| **Bubble Database** | Data storage | Stores users (authors/admins), books, payout history, notification logs. |
| **Webhooks / REST API** | System integration | Bubble → n8n via authenticated POST; n8n → Bubble via backend API endpoint callback. |

No additional technologies are used.

---

## 5. Application Access

**Live Application (Test Version):**

```
https://apandey8957.bubbleapps.io/version-test/
```

This is Bubble.io's test (development) version. All features, workflows, and data are on this version.

> **Note:** The application is on Bubble's Free plan. Backend API workflow run limits apply. If a notification fails due to plan limits, it is logged as Failed and can be retried individually.

---

## 6. Test Account Information

> ⚠️ **Credentials are not published in this README.** Test account email addresses and passwords will be shared privately with the evaluator on request.

| Role | Email | Password | Access Level |
|---|---|---|---|
| **Author** | *[shared privately]* | *[shared privately]* | Own books, royalties, and notifications only |
| **Admin** | *[shared privately]* | *[shared privately]* | All authors, books, notification logs, bulk send |

After login, the application automatically redirects each user to their role-appropriate dashboard. An author attempting to access admin pages is redirected to the login page by a page-load guard workflow.

---

## 7. Application Features

### Author Portal

| Page | Features | Status |
|---|---|---|
| Login | Email/password authentication, role-based redirect | ✅ Complete |
| Dashboard | Summary cards: total earned, total paid, total pending, books count | ✅ Complete |
| My Books | Books table with status, royalty per copy, sort and filter | ✅ Complete |
| Book Detail | Per-book royalty breakdown, sales data, platform availability | ✅ Complete |
| Royalty Summary | Royalty breakdown across all books with ₹ formatting | ✅ Complete |
| Profile | Author profile: name, city, phone, joined date, author code | ✅ Complete |

### Admin Portal

| Page | Features | Status |
|---|---|---|
| Admin Dashboard | Platform summary: total authors, books, revenue, pending royalties | ✅ Complete |
| Authors List | All 10 authors with search, city filter, status filter, sort | ✅ Complete |
| Author Detail | Full profile, royalty stats, books table, notification history, Send button | ✅ Complete |
| Payouts | Payout history log across all authors | ✅ Complete |
| Notification Logs | All notification attempts with status, filters, and timestamps | ✅ Complete |
| Bulk Send | Send royalty summaries to all authors with pending royalty > ₹0 | ✅ Complete |

### Notification System

| Feature | Status |
|---|---|
| Async notification queue (Bubble → n8n → callback) | ✅ Complete |
| Notification log created per attempt (Processing → Success / Failed) | ✅ Complete |
| Duplicate prevention: blocks second send while one is Processing | ✅ Complete |
| Automatic expiry: Processing logs older than 10 min → Failed | ✅ Complete |
| Late callback guard: expired/failed logs cannot be overwritten | ✅ Complete |
| Secret header authentication on n8n webhook (X-BookLeaf-Secret) | ✅ Complete |
| Full author and book data sent in POST body (no Data API dependency) | ✅ Complete |
| Real email delivery | 🔲 Not required for this assessment |

---

## 8. Database Structure

### Data Types

| Data Type | Key Fields | Notes |
|---|---|---|
| **User** (built-in) | email, role (Author / Admin), name, author_code, city, phone, joined_date, total_earned, total_paid, pending_royalty, last_payout_date, royalty_status | Bubble's built-in User type extended with author fields. Role determines portal access. |
| **book** | title, isbn, genre, publication_date, status, mrp, royalty_per_copy, copies_sold, total_earned, royalty_paid, royalty_pending, author (User ref) | One author can have multiple books. Status: Published & Live / In Production. |
| **payout_history** | author (User ref), amount, payment_date, payment_method, reference_number, status | Individual payment records per author. Table structure is in place; no transaction-level source data was provided. |
| **notification_log** | author (User ref), status, request_id, triggered_by, created_date, responded_at, summary_text, error_message | One record per notification attempt. Lifecycle: Processing → Success \| Failed. |

### Royalty Status Logic

| Status | Condition | Indicator |
|---|---|---|
| Fully Paid | pending_royalty = 0 | 🟢 Fully Paid |
| Pending | pending_royalty > 0, last payout within 90 days | 🟡 Pending |
| Overdue | pending_royalty > 0 AND (last payout > 90 days ago, OR never paid for a book published > 90 days ago) | 🔴 Overdue |

### Sample Data

The application is pre-loaded with realistic sample data:
- **10 authors** across 8 cities in India
- **18 books** (15 Published & Live, 3 In Production)
- **10 notification log records** from testing runs
- All stored totals verified to match the sum of their individual book records exactly

---

## 9. n8n Workflow Documentation

### Workflow: BookLeaf – Author Royalty Summary Automation

**File in repository:** `BookLeaf – Author Royalty Summary Automation.json`

**Status:** Published and active (`"active": true` in the JSON)

**Purpose:** Receives a royalty notification request from Bubble, validates it, calculates royalty totals and status, generates a personalised royalty summary, builds an HTML email, and returns the result to Bubble via callback.

---

#### Trigger

| Property | Value |
|---|---|
| Node name | `Webhook – Receive Request` |
| HTTP method | POST |
| Path | `/bookleaf/royalty-summary` |
| Authentication | Header Auth (`X-BookLeaf-Secret`) — wrong or missing secret is rejected with 403 before the workflow runs |
| Response mode | `responseNode` (workflow controls the HTTP response itself) |

---

#### Execution Steps (all 18 nodes, in order)

| # | Node Name | Type | What It Does |
|---|---|---|---|
| 1 | **Webhook – Receive Request** | Webhook | Receives the POST from Bubble. Validates the secret header. Rejects unauthorised calls with 403. |
| 2 | **Workflow Config** | Set | Holds configuration flags: `email_enabled` (false by default), `email_test_recipient`, `email_from`. Single place to toggle email sending. |
| 3 | **Validate Request** | Code (JS) | Checks that `Content-Type` is `application/json`. Validates required fields: `author_id`, `request_id`, `triggered_by`, `author_name`, `author_email` (format-checked), `books_summary`. Parses the `books_summary` pipe-delimited string into a structured books array. |
| 4 | **Request Valid?** | IF | Routes to `Check Duplicate Request` if valid; to `Respond – Invalid Request` if not. |
| 5 | **Respond – Invalid Request** | Respond to Webhook | Returns `{ status: "Rejected" }` with validation error details. |
| 6 | **Check Duplicate Request** | Code (JS) | Checks n8n's persistent static data (survives across executions) for the `request_id` and `author_id`. Flags as duplicate if: (a) already completed successfully, or (b) currently processing (under 10 min). Failed requests can be retried. |
| 7 | **Is Duplicate?** | IF | Routes to `Respond – Duplicate Request` if duplicate; to `Prepare Royalty Data` if not. |
| 8 | **Respond – Duplicate Request** | Respond to Webhook | Returns `{ status: "Duplicate" }`. Does not call back to Bubble, so the original notification log is never overwritten. |
| 9 | **Prepare Royalty Data** | Code (JS) | Structures the author and book data into one clean object. Splits books into Published and In Production. Throws on missing data so the failure branch catches it. |
| 10 | **Calculate Royalty Totals** | Code (JS) | Sums total earned, paid, and pending from all books. Counts copies sold (published books only). Finds the most recent payout date. |
| 11 | **Check Royalty Status** | Code (JS) | Determines FULLY PAID / PENDING / OVERDUE using the same 90-day rule as Bubble. Calculates the next expected payout date (quarterly 45-day policy). Flags if balance is below ₹1,000 minimum threshold. Prepares the structured data used to build the summary. |
| 12 | **Build Email HTML** | Code (JS) | Builds the summary text and a full professional HTML email from the calculated data. Includes: BookLeaf header, royalty status badge (green/amber/red), three summary tiles (Earned/Paid/Pending), per-book table, last payout date, and next payout explanation. |
| 13 | **Real Email Enabled?** | IF | Checks `email_enabled` flag from Workflow Config. Currently `false` — no real email is sent. Routes to `Send Email` if true; to `Build Success Response` if false. |
| 14 | **Send Email** | Email Send | Only runs when `email_enabled` is `true`. Requires a configured SMTP credential. Supports `email_test_recipient` to redirect all emails to a test address. |
| 15 | **Build Success Response** | Code (JS) | Marks the request as done in n8n's static data (blocks future duplicate sends). Checks if n8n's calculated totals match Bubble's submitted totals. Builds the JSON payload for Bubble's callback. |
| 16 | **Respond – Success** | Respond to Webhook | Returns `{ status: "Success", summary: "...", totals_match: true/false, ... }` to Bubble. |
| 17 | **Build Failure Response** | Code (JS) | Single failure handler for all error paths (bad data, processing errors). Releases the duplicate lock so the request can be retried. Returns error details to Bubble. |
| 18 | **Respond – Failure** | Respond to Webhook | Returns `{ status: "Failed", error_message: "..." }` to Bubble. |

---

#### Node Groups (logical sections in the n8n canvas)

| Group | Nodes Included | Purpose |
|---|---|---|
| Validate request | Workflow Config, Validate Request, Request Valid? | Validates the incoming payload before anything else runs |
| Prevent duplicates | Check Duplicate Request, Is Duplicate? | Ensures the same request never produces two summaries |
| Work out royalty status | Calculate Royalty Totals, Check Royalty Status | Totals and classifies the author's royalty position |

---

#### Expected Result

| Outcome | Bubble notification_log status | n8n Response |
|---|---|---|
| Success | Processing → **Success**, `summary_text` populated | `{ status: "Success", totals_match: true }` |
| Duplicate request | Unchanged (no callback sent) | `{ status: "Duplicate" }` |
| Validation failed | Processing → **Failed** via callback | `{ status: "Failed", error_message: "..." }` |

---

#### Implementation Status

| Component | Status |
|---|---|
| Webhook and authentication | ✅ Implemented and published |
| Request validation (all 5 required fields) | ✅ Implemented |
| Duplicate prevention (n8n static data) | ✅ Implemented |
| Royalty data preparation and totals calculation | ✅ Implemented |
| Royalty status classification (FULLY PAID / PENDING / OVERDUE) | ✅ Implemented |
| HTML email builder | ✅ Implemented |
| Email sending (SMTP) | 🔲 Nodes built; `email_enabled` is `false` — not required for this assessment |
| End-to-end Bubble → n8n → callback flow | 🔄 Integration testing in progress |

---

## 10. Bubble and n8n Integration

### Complete Async Flow

```
Admin clicks "Send Royalty Summary" (single or bulk)
        │
        ▼
Bubble: queue_royalty_notification (backend workflow)
  - Creates notification_log { status: Processing, request_id: UUID, author, triggered_by }
  - Schedules expire_notification for now + 11 minutes
  - POST to n8n webhook with full author + book data in body
        │
        ▼
n8n: Webhook – Receive Request
  - Validates X-BookLeaf-Secret header
  - Validates request fields
  - Checks for duplicate (request_id / author_id in static data)
  - Calculates royalty totals and status → generates royalty summary
  - Builds HTML email (not sent in this assessment)
  - POST to Bubble royalty_callback with { request_id, summary_text, status }
        │
        ▼
Bubble: royalty_callback (exposed backend API endpoint)
  - Looks up notification_log by request_id WHERE status = Processing
  - Guard: if log is already Failed or Success → callback is ignored
  - Updates log: status = Success, summary_text stored, responded_at = now
        │
        ▼ (if n8n never responds within 10 min)
Bubble: expire_notification (backend workflow)
  - Sets all Processing logs older than 10 min → Failed
  - Author becomes re-triggerable
```

### Why No Data API Calls from n8n

Bubble's Data API is not available on the Free plan. All required data (author name, email, totals, books) is included in the POST body from Bubble to n8n. n8n uses only what Bubble sends — no secondary API calls are made.

### Duplicate Prevention (Two Layers)

1. **Bubble side:** `queue_royalty_notification` checks for an existing `Processing` log before creating a new one.
2. **n8n side:** `Check Duplicate Request` node reads n8n's persistent static data for the `request_id` and `author_id`. Prevents double-processing even if two Bubble calls arrive simultaneously.

### Bulk Processing

The bulk send button uses Bubble's `ScheduleAPIEventOnList`, which schedules one `queue_royalty_notification` call per author asynchronously. The list is filtered to `pending_royalty > 0`, so fully paid authors are automatically skipped. Each author's notification is independent — a failure for one does not affect the others.

---

## 11. Setup and Configuration

### Bubble Configuration

- Application at `apandey8957.bubbleapps.io`, Free plan, test version.
- API Connector configured with the n8n webhook URL and `X-BookLeaf-Secret` header (marked **Private** so it never appears in client-side logs).
- Backend workflow `royalty_callback` exposed as an API endpoint with admin-only authentication.
- Backend workflow `queue_royalty_notification` creates notification logs and calls n8n.
- Backend workflow `expire_notification` sweeps stale Processing logs to Failed (manual trigger on Free plan).
- Privacy rules enforce role-based data isolation: authors see only their own records; admins see all records.

### n8n Configuration

- Workflow published and active (`"active": true` in the exported JSON).
- **Webhook node** configured with Header Auth credential matching Bubble's `X-BookLeaf-Secret`.
- **Callback URL** in `Build Success Response` and `Build Failure Response` nodes points to Bubble's `royalty_callback` endpoint.
- **Email:** `email_enabled` is `false` in `Workflow Config`. To enable real email sending, set it to `true` and configure an SMTP credential and a verified `email_from` address.

> 🔒 **Security note:** The shared secret (`X-BookLeaf-Secret`), n8n webhook URL, and Bubble API credentials are stored as private credentials in their respective platforms. None of these are included in this repository.

---

## 12. Testing and Validation

### Data Accuracy

All 10 authors' stored royalty totals (earned, paid, pending) have been independently verified against the sum of their individual book records. All values match exactly.

### Successfully Tested Features

| Test | Evidence |
|---|---|
| Author login → Author Dashboard | Manual browser test, role redirect confirmed |
| Admin login → Admin Dashboard | Manual browser test, role redirect confirmed |
| Author cannot access admin pages | Privacy rule check + page-load guard redirect |
| Author click passes correct ID to Author Detail page | URL parameter `?author=<id>` confirmed in browser address bar |
| Author Detail page loads correct data for 3 different authors | Rendered for Priya Sharma, Rohit Kapoor, Ananya Reddy — each showed correct data |
| Duplicate notification blocked while one is Processing | Workflow logic confirmed — existing Processing log blocks new trigger |
| Failed n8n call → notification log recorded as Failed | 4 existing Failed log records with error messages in the database |
| Bulk send: Farhan Sheikh (₹0 pending) skipped | Filter on `pending_royalty > 0` confirmed |
| All 10 authors' royalty totals match book-level records | Arithmetic verified record by record |

### Partially Tested / In Progress

| Test | Current Status |
|---|---|
| End-to-end flow: Bubble → n8n → Success callback → notification log updated | In progress — `X-BookLeaf-Secret` header configuration completed; end-to-end run pending |
| Expiry sweep: 6 stuck Processing logs → Failed | In progress — `expire_notification` workflow ready; manual trigger pending |

### Pending Tests

| Test | Reason Pending |
|---|---|
| Generated summary content quality review | Dependent on successful end-to-end run |
| Bulk send with all authors having pending > ₹0 | Dependent on workflow limit reset on Free plan |

---

## 13. Demo Video

A screen recording of the application is available here:

**[▶ Watch Demo on Loom](https://www.loom.com/share/1fc4cd71086a44918358500171b60db2)**

The recording covers the author portal, admin portal, notification flow, and log management.

---

## 14. Assumptions and Limitations

### Assumptions

- **No payout transaction data in source:** The sample data provided per-book aggregate totals (total earned, total paid, last payout date) but not individual payment transaction records. The `payout_history` table structure is in place and ready, but no historical transactions exist in the dataset.
- **Email delivery is out of scope:** The assessment requires the notification log and generated summary to work correctly. Email sending is built into the n8n workflow but intentionally disabled (`email_enabled: false`). It can be enabled by configuring an SMTP credential.
- **Author accounts are pre-seeded:** Accounts were created to match the sample data authors. A production system would include an author onboarding and registration flow.

### Bubble Free Plan Limitations

| Limitation | Impact | How the app handles it |
|---|---|---|
| Data API disabled | n8n cannot fetch Bubble data | All required data is sent in the POST body from Bubble to n8n |
| Backend workflow run limits per time window | A bulk send of 10 authors may hit the cap; affected runs are killed before the failure handler runs, leaving logs as Processing | `expire_notification` sweeps stuck Processing logs to Failed; authors can be re-triggered individually |
| No scheduled cron jobs | `expire_notification` cannot run on a timer | Manual trigger required after each bulk operation |

---

## 15. Project Completion Summary

| Requirement | Status | Notes |
|---|---|---|
| Role-based authentication (Author / Admin) | ✅ Complete | Privacy rules + page-load guards |
| Author portal (6 pages) | ✅ Complete | All pages wired to live data |
| Admin portal (6 pages) | ✅ Complete | All pages wired to live data |
| Accurate royalty data for all 10 authors | ✅ Complete | Verified: all totals match book-level records |
| Asynchronous n8n webhook integration | ✅ Complete | Async queue → n8n → callback |
| Personalised royalty summary generation | ✅ Complete | 18-node workflow with validation |
| Notification log lifecycle (Processing → Success / Failed) | ✅ Complete | All transitions wired |
| Duplicate prevention (Bubble + n8n layers) | ✅ Complete | Two-layer guard |
| Automatic expiry of stuck Processing logs | ✅ Complete | `expire_notification` workflow; manual trigger on Free plan |
| Late callback guard | ✅ Complete | Callback ignored if log is already Failed/Success |
| Bulk notification (pending authors only) | ✅ Complete | Filtered to `pending_royalty > 0` |
| Secure secret header (X-BookLeaf-Secret) | ✅ Complete | Private credential in Bubble API Connector |
| Payout history section on Author Detail page | ✅ Complete | Built with empty state |
| Real email delivery | 🔲 Not required | Nodes built; `email_enabled: false` |
| End-to-end tested (Bubble → n8n → callback → Success log) | 🔄 In progress | Integration test pending |

---

> **Files inspected for this README:**
> - `Readme.md` — previous documentation
> - `BookLeaf – Author Royalty Summary Automation.json` — full n8n workflow export (nodes, connections, node groups, settings)
>
> **No application code, workflow JSON files, or database records were modified.**

---

*BookLeaf Publishing – Author Royalty Dashboard · Built with Bubble.io & n8n · Assessment 2*
