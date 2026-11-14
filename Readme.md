Technical Assessment — Assessment 2

# BookLeaf Publishing Author Royalty Dashboard

An Author Royalty Management Portal built with Bubble.io and n8n workflow automation — managing authors, books, royalties, payout history, and automated notifications.

🟣 Bubble.io ⚡ n8n Automation 🔗 Webhook Integration No-Code · Full-Stack

Contents

[01Project Overview](#s1) [02Problem Statement](#s2) [03Objectives](#s3) [04Technology Stack](#s4) [05Application Access](#s5) [06Test Accounts](#s6) [07Application Features](#s7) [08Database Overview](#s8) [09n8n Workflow Documentation](#s9) [10Bubble & n8n Integration](#s10) [11Setup & Configuration](#s11) [12Testing & Validation](#s12) [13Screenshots / Demo](#s13) [14Assumptions & Limitations](#s14) [15Project Completion Summary](#s15)

## 01 Project Overview

BookLeaf Publishing – Author Royalty Dashboard is a web-based management portal that helps BookLeaf Publishing track authors, their published books, royalty earnings, payout history, and automated royalty summary notifications.

The portal has two separate access roles: **Authors** can log in to view their own royalty data, books, and notifications. **Admins** can view and manage all authors, trigger royalty summary notifications, and monitor the notification log.

The notification feature uses an asynchronous architecture: Bubble sends a POST request to an n8n webhook, n8n processes the summary and calls back to Bubble's API endpoint with the result. This ensures Bubble remains responsive while n8n handles the generation work independently.

## 02 Problem Statement

Publishing companies that work with many authors face a recurring challenge: keeping each author informed about their royalty status. Manually calculating, compiling, and sending individual royalty summaries is time-consuming and error-prone.

BookLeaf Publishing needed a centralised system where:

- Authors can self-serve and check their own royalty information at any time.
- Admins can view all author and book data from a single dashboard.
- Royalty summary notifications can be sent to individual authors or in bulk with minimal manual effort.
- Every notification attempt is logged for audit and troubleshooting.

## 03 Objectives

- Build a role-based portal for authors and admins using Bubble.io.
- Store and display author, book, and royalty data accurately.
- Implement an asynchronous notification system using n8n webhooks.
- Log every notification attempt with its status (Processing / Success / Failed).
- Prevent duplicate notifications using a processing-state guard.
- Provide admins with a bulk notification option for authors with pending royalties.
- Ensure all royalty calculations match the source data exactly.

## 04 Technology Stack

| Technology | Role | Details |
| --- | --- | --- |
| **Bubble.io** | Application platform | All pages, workflows, database, and privacy rules. Free plan, test version. |
| **n8n** | Workflow automation | Receives webhook from Bubble, generates royalty summary, callbacks to Bubble. |
| **Bubble Database** | Data storage | Stores authors, books, payout history, and notification logs. |
| **Webhooks / REST API** | System integration | Bubble → n8n via POST; n8n → Bubble via exposed backend API endpoint. |

## 05 Application Access

Live Application (Test Version)

The application is deployed on Bubble.io and accessible at:

`https://apandey8957.bubbleapps.io/version-test/`

This is the test (development) version. All features, workflows, and data are on this version.

ℹ️

The application is built on Bubble.io's Free plan. Backend API workflow limits apply. If a notification fails due to plan limits, it is logged as Failed and can be retried individually.

## 06 Test Account Information

⚠️

**Credentials are not published publicly.** Test account email addresses and passwords will be shared privately with the evaluator on request.

| Role | Email | Password | Access Level |
| --- | --- | --- | --- |
| **Author** | *\[shared privately\]* | *\[shared privately\]* | Own books, royalties, and notifications only |
| **Admin** | *\[shared privately\]* | *\[shared privately\]* | All authors, books, notification logs, bulk send |

After logging in, the application automatically redirects each user to their role-appropriate dashboard. An author attempting to access admin pages is redirected to the login page.

## 07 Application Features

### Author Portal

| Page | Feature | Status |
| --- | --- | --- |
| Login | Email/password authentication, role-based redirect | Complete |
| Dashboard | Summary cards: total earned, paid, pending, books count | Complete |
| My Books | Books table with status, royalty per copy, sort and filter | Complete |
| Book Detail | Per-book royalty breakdown, sales data, availability | Complete |
| Royalty Summary | Royalty breakdown across all books with ₹ formatting | Complete |
| Profile | Author profile information | Complete |

### Admin Portal

| Page | Feature | Status |
| --- | --- | --- |
| Admin Dashboard | Platform-wide summary: total authors, books, revenue, pending royalties | Complete |
| Authors List | All 10 authors with search, city filter, status filter, sort | Complete |
| Author Detail | Full profile, royalty stats, books table, notification history, Send button | Complete |
| Payouts | Payout history log across all authors | Complete |
| Notification Logs | All notification attempts with status, filters, and timestamps | Complete |
| Bulk Send | Send royalty summaries to all authors with pending royalty > ₹0 | Complete |

### Notification System

| Feature | Status |
| --- | --- |
| Async notification queue (Bubble → n8n → callback) | Complete |
| Notification log created per attempt (Processing → Success/Failed) | Complete |
| Duplicate prevention: blocks second send while one is Processing | Complete |
| Automatic expiry: Processing logs older than 10 min → Failed | Complete |
| Late callback guard: expired/failed logs cannot be overwritten | Complete |
| Secret header authentication on n8n webhook (X-BookLeaf-Secret) | Complete |
| Full author and book data sent in POST body (no Data API dependency) | Complete |
| Real email delivery | Not required |

## 08 Database Overview

The Bubble database contains the following data types:

| Data Type | Key Fields | Notes |
| --- | --- | --- |
| **User** | email, role (Author / Admin), name, author_code, city, phone, joined_date | Built-in Bubble type extended with author fields. Role determines portal access. |
| **book** | title, isbn, genre, publication_date, status, mrp, royalty_per_copy, copies_sold, total_earned, royalty_paid, royalty_pending, author (User ref) | One author can have multiple books. Status: Published & Live / In Production. |
| **payout_history** | author (User ref), amount, payment_date, payment_method, reference_number, status | Individual payment records per author. Currently no records; structure is ready. |
| **notification_log** | author (User ref), status, request_id, triggered_by, created_date, responded_at, summary_text, error_message | One record per notification attempt. Lifecycle: Processing → Success \| Failed. |

### Royalty Status Logic

| Status | Condition | Indicator |
| --- | --- | --- |
| Fully Paid | pending_royalty = 0 | 🟢 Fully Paid |
| Pending | pending_royalty > 0, last payout within 90 days | 🟡 Pending |
| Overdue | pending_royalty > 0, last payout > 90 days ago (or never paid for a book published > 90 days ago) | 🔴 Overdue |

### Sample Data

The application is pre-loaded with realistic sample data: 10 authors, 18 books (15 published, 3 in production), and 10 notification log records from testing. All stored totals have been verified to match the sum of their individual book records exactly.

## 09 n8n Workflow Documentation

Royalty Summary Notification Published

Purpose Receives a royalty summary request from Bubble, generates a formatted summary, and returns the result to Bubble via callback.

Trigger Webhook (HTTP POST) from Bubble's API Connector. Authenticated using the `X-BookLeaf-Secret` header (Header Auth credential in n8n, Private in Bubble).

Input Payload `request_id`, `author_name`, `author_email`, `author_code`, `total_earned`, `total_paid`, `pending_royalty`, `last_payout_date`, and a `books` array with per-book royalty details. All data is sent directly from Bubble — no Data API call required.

Processing Steps Webhook node receives the POST and validates the secret header. Summary generation node formats the author's royalty data into a structured text summary. HTTP Request node POSTs the result to Bubble's `royalty_callback` backend API endpoint, sending `request_id` and `summary_text`.

Bubble Integration Bubble calls this workflow via API Connector. n8n calls back to Bubble's exposed backend workflow endpoint (`/api/1.1/wf/royalty_callback`) with admin-only authentication.

Expected Output Bubble's `notification_log` record for the author is updated from `Processing` → `Success`, with the generated `summary_text` stored in the log.

Email Delivery Not required for this assessment. The workflow generates and stores the royalty summary. Email delivery can be added later by enabling an SMTP node with a verified sender.

Testing Status Integration testing in progress — workflow is published and responding. End-to-end callback tested after secret header configuration.

## 10 Bubble & n8n Integration

### Complete Async Flow

1

**Admin triggers notification (single or bulk)** Bubble's `queue_royalty_notification` backend workflow creates a `notification_log` record with status `Processing` and a unique `request_id`.

2

**Bubble POSTs to n8n webhook** Full author and book data is sent in the request body. The `X-BookLeaf-Secret` header authenticates the call. An expiry job is scheduled for 11 minutes later as a safety net.

3

**n8n generates the royalty summary** Using the data received in the POST body — no additional API calls to Bubble required. n8n formats the summary and prepares the callback payload.

4

**n8n calls back to Bubble** POST to Bubble's `royalty_callback` endpoint with `request_id` and `summary_text`. Admin-only authentication applied.

5

**Bubble updates the notification log** The log record matching the `request_id` is updated: status → `Success`, `summary_text` stored, `responded_at` recorded. Guard: if the log is already `Failed` or `Success`, the callback is ignored.

6

**Fallback: expiry sweep** If n8n never responds within 10 minutes, the `expire_notification` workflow sets any `Processing` log older than 10 minutes to `Failed`. The author can then be re-notified.

### Duplicate Prevention

Before creating a new notification log, the queue workflow checks whether a `Processing` log already exists for the author. If one exists, the new request is blocked. This prevents double-sending when an admin clicks the button more than once.

### Bulk Processing

The bulk send button uses Bubble's `ScheduleAPIEventOnList` to process one author at a time asynchronously. The list is filtered to authors with `pending_royalty > 0`, so authors who are fully paid are automatically skipped. Each author gets an independent notification log, and a failure for one author does not affect the others.

## 11 Setup & Configuration

### Bubble Configuration

- Application hosted at `apandey8957.bubbleapps.io` on the Free plan.
- API Connector configured with the n8n webhook URL and `X-BookLeaf-Secret` header (marked Private).
- Backend workflow `royalty_callback` exposed as an API endpoint with admin-only authentication.
- Privacy rules enforce role-based data access: authors see only their own records; admins see all records.

### n8n Configuration

- Workflow published and active.
- Webhook node configured with Header Auth credential (`X-BookLeaf-Secret`).
- Callback URL pointed to Bubble's backend API endpoint.
- No external email service configured (not required for this assessment).

🔒

**Security note:** The shared secret (`X-BookLeaf-Secret`), n8n webhook URL, and Bubble API credentials are stored as private credentials. They are not included in this README and will not be shared publicly.

## 12 Testing & Validation

### Data Accuracy Verification

✅

All 10 authors' stored royalty totals (earned, paid, pending) have been verified against the sum of their individual book records. All values match exactly.

### Feature Testing Summary

| Test Case | Method | Result |
| --- | --- | --- |
| Author login → redirected to Author Dashboard | Manual browser test | Pass |
| Admin login → redirected to Admin Dashboard | Manual browser test | Pass |
| Author cannot access admin pages | Privacy rule + page guard | Pass |
| Author click on Authors list passes correct ID | URL parameter inspection | Pass |
| Author Detail page loads correct author's data | Rendered for 3 authors | Pass |
| Duplicate notification blocked while Processing | Workflow logic check | Pass |
| Failed n8n call → log recorded as Failed | 4 existing log records | Pass |
| Bulk send: Farhan Sheikh (₹0 pending) skipped | Filter on pending > 0 | Pass |
| End-to-end: Bubble → n8n → callback → Success log | Live test after secret header fix | In Progress |
| Expiry sweep: stuck Processing logs → Failed | Manual trigger | In Progress |

## 13 Screenshots / Demo

The live application can be accessed and tested directly at:

`https://apandey8957.bubbleapps.io/version-test/`

Key pages to review during evaluation:

| Page | Path | Login Required |
| --- | --- | --- |
| Login | `/version-test/` | — |
| Author Dashboard | `/version-test/author_dashboard` | Author role |
| Admin Dashboard | `/version-test/admin_dashboard` | Admin role |
| Admin Authors List | `/version-test/admin_authors` | Admin role |
| Author Detail (example) | `/version-test/admin_author_detail?author=<id>` | Admin role |
| Notification Logs | `/version-test/admin_logs` | Admin role |

## 14 Assumptions & Limitations

### Assumptions

- Sample data was provided without individual payout transaction records; only aggregate totals (total earned, total paid, last payout date) were given per book. The `payout_history` table structure is in place, but no historical transaction data exists in the source dataset.
- Real email delivery is not part of the assessment scope. The notification system generates and stores royalty summaries; email sending can be enabled by adding an SMTP integration to n8n.
- Author accounts are pre-seeded to match the sample data. In a production system, an author onboarding flow would be added.

### Known Limitations (Bubble Free Plan)

- **Backend workflow run limits:** The Free plan restricts the number of server-side workflow runs per time period. A bulk notification of 10 authors may hit this limit. Affected authors receive a `Failed` log and can be re-notified individually once the limit resets.
- **Data API disabled:** Bubble's Data API is not available on the Free plan. The notification system is designed around this — all required data is sent in the POST body from Bubble to n8n, so no Data API access is needed.
- **No scheduled cron jobs:** The expiry sweep workflow (`expire_notification`) cannot run on a fixed schedule on the Free plan. It must be triggered manually or via a backend workflow call after a bulk operation.

## 15 Project Completion Summary

| Requirement | Status | Notes |
| --- | --- | --- |
| Role-based authentication (Author / Admin) | Complete | Privacy rules + page guards implemented |
| Author portal (dashboard, books, royalties, profile) | Complete | All 6 author pages built and wired to live data |
| Admin portal (dashboard, authors, payouts, logs) | Complete | All 6 admin pages built and wired to live data |
| Accurate royalty data for all 10 authors | Complete | Verified: all totals match book-level records |
| n8n webhook integration | Complete | Async flow implemented; end-to-end test in progress |
| Notification log (Processing / Success / Failed lifecycle) | Complete | All lifecycle transitions wired |
| Duplicate prevention | Complete | Processing-state guard blocks duplicate sends |
| Automatic expiry of stuck logs | Complete | Sweeper workflow ready; manual trigger required on Free plan |
| Late callback guard | Complete | Callback rejected if log is already Failed/Success |
| Bulk notification (pending authors only) | Complete | Filtered to pending_royalty > 0 |
| Secure secret header (X-BookLeaf-Secret) | Complete | Private credential in Bubble API Connector |
| Payout history section on Author Detail page | Complete | Section built with empty state; no source data to seed |
| Real email delivery | Not Required | Explicitly excluded from assessment scope |

🎯

**Assessment summary:** This project demonstrates practical no-code application development using Bubble.io, asynchronous API integration with n8n webhooks, structured database design with privacy rules, and reliable notification management with duplicate prevention and automatic failure recovery. The architecture is designed to work within Bubble's Free plan constraints without compromising on feature completeness.

BookLeaf Publishing – Author Royalty Dashboard · Built with Bubble.io & n8n · Assessment 2