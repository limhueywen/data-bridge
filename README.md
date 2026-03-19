# The Data Bridge
### A Centralized Data Management & Reconciliation System
**Grab × Loyverse POS × Infotech Accounting**

> *Orders flow. Customers are known. Accounts close themselves.*

---

## The Problem

A business running on multiple platforms — Grab Food, Loyverse POS, and Infotech — faces three critical bottlenecks:

| Problem | Impact |
|---|---|
| Loyverse gross sales never match Grab net payouts | Finance team cannot determine true profit |
| Grab masks customer phone numbers | Impossible to identify loyal or repeat buyers |
| No automated data transfer between systems | Staff spend hours on manual reconciliation daily |

---

## Our Solution

The **Data Bridge** is a custom event-driven middleware that sits between all three platforms and acts as a **Single Source of Truth (SSOT)**.

It ingests Grab orders in real time, translates them into Loyverse format, tracks customers using anonymous UUIDs, reconciles financials automatically, and exports a verified daily ledger to Infotech — with **zero staff intervention**.

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        DATA SOURCES                         │
│                                                             │
│   Grab Food App          Loyverse POS         Grab UUID     │
│   (delivery orders)      (walk-in orders)     (customer ID) │
└────────┬─────────────────────┬──────────────────┬───────────┘
         │  Webhook (JSON)     │  API Pull         │  Metadata
         ▼                     ▼                   ▼
┌─────────────────────────────────────────────────────────────┐
│                     INGESTION LAYER                         │
│                                                             │
│   Webhook Listener       Loyverse API        UUID Extractor │
│   Signature verify       Validate + format   Attach hidden  │
│   Python · GCF           Python · GCF        metadata       │
└─────────────────────────────┬───────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│              CORE ENGINE  (Python · Google Cloud Functions) │
│                                                             │
│   ┌──────────────────┐  ┌──────────────┐  ┌─────────────┐  │
│   │ Data             │  │ SKU          │  │ Financial   │  │
│   │ Transformation   │  │ Resolution   │  │ Reconcile   │  │
│   │ Grab → Loyverse  │  │ Item mapping │  │ Commission  │  │
│   └──────────────────┘  └──────────────┘  └─────────────┘  │
└──────────┬──────────────────┬────────────────────┬──────────┘
           │                  │                    │
           ▼                  ▼                    ▼
┌─────────────────────────────────────────────────────────────┐
│                       STORAGE LAYER                         │
│                                                             │
│   Order Queue            UUID Database       Sync Log       │
│   PostgreSQL             Firebase            All events     │
│   Offline fail-safe      Customer map        Errors/retries │
└──────────┬──────────────────┬────────────────────┬──────────┘
           │                  │                    │
           ▼                  ▼                    ▼
┌─────────────────────────────────────────────────────────────┐
│                       OUTPUT LAYER                          │
│                                                             │
│   Loyverse POS           Kitchen Display     Infotech       │
│   Receipt auto-created   Grab + walk-in      CSV / JSON     │
│   Kitchen ticket sent    unified view        Auto scheduled │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    FLUTTER DASHBOARD                        │
│           Web · iOS · Android — owner & manager view        │
│      Live sync logs · Daily summary · Customer insights     │
└─────────────────────────────────────────────────────────────┘
```

---

## Workflow — What Happens to Every Order

```
Step 1   Customer places order on Grab
         └─► Grab sends a secure webhook (JSON) to our backend instantly

Step 2   Webhook listener receives the order
         ├─► Verify cryptographic signature (security)
         ├─► Validate all required fields (completeness)
         └─► Extract hidden Grab Customer UUID (identity)

Step 3   Data transformation
         └─► Convert Grab format → Loyverse format
             Match item names to Loyverse SKUs

Step 4   Internet check
         ├─► Online  → push to Loyverse API immediately
         └─► Offline → store in PostgreSQL queue
                       auto-retry when connection restores
                       zero orders lost

Step 5   Push to Loyverse POS
         ├─► Receipt created automatically
         ├─► Kitchen ticket generated
         └─► Both Grab + walk-in orders visible in unified kitchen view

Step 6   End-of-day reconciliation
         ├─► Compare Grab settlement vs Loyverse receipts
         ├─► Separate: Gross Sales / Commission (30%) / Net Payout
         └─► Flag any unmatched anomalies for review

Step 7   Auto export to Infotech
         └─► Perfectly balanced daily ledger
             CSV / JSON format · scheduled · no manual action
```

---

## Customer Intelligence — Solving the UUID Problem

Grab hides all customer phone numbers for privacy compliance. Our system works around this using **Grab Customer UUIDs** — anonymous alphanumeric identifiers included in every order payload.

```
Order arrives
     │
     ▼
Extract UUID from payload
     │
     ├── UUID exists in database?
     │        │
     │        ├── YES → Update profile
     │        │         (visit count +1, total spend updated)
     │        │
     │        └── NO  → Create new "Grab User" profile
     │
     ▼
Business insights unlocked:
  • New vs Repeat customers (across delivery + walk-in)
  • High-value customers (ranked by cumulative spend)
  • Inactive customers (30+ days — for re-engagement campaigns)
```

---

## Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| Backend / Webhook | Python + Google Cloud Functions | Serverless event processing |
| Order Queue | PostgreSQL | Offline fail-safe storage |
| Customer Database | Firebase | UUID mapping + profile store |
| POS Integration | Loyverse API | Receipt creation, item sync |
| Delivery Integration | Grab Merchant API | Webhook ingestion |
| Accounting Export | CSV / JSON | Infotech-compatible format |
| Dashboard | Flutter (Web / iOS / Android) | Owner-facing control panel |

---

## Key Design Decisions

### Offline-resilient queue
Internet connections fail — especially during peak lunch hours. Every order that cannot be pushed immediately is stored in a local PostgreSQL queue and automatically retried when connectivity restores. No orders are ever lost.

### UUID-based customer tracking
Grab's privacy policy prohibits sharing customer phone numbers. Rather than treating this as a limitation, the system uses the anonymous UUID Grab provides in every payload to build persistent customer profiles — fully compliant, fully functional.

### Zero staff training required
The Data Bridge operates entirely at the middleware layer. Cashiers continue using Loyverse exactly as before. Kitchen staff see one unified screen. Nothing in the front-of-house workflow changes. The system is invisible to the people it serves.

### Mathematically verified financials
The reconciliation layer does not estimate or approximate. It compares every Grab settlement line against every generated Loyverse receipt, applies the exact commission percentage, and flags any discrepancy for review before export. Accounting receives numbers it can trust.

---

## Live Prototype

A working demonstration of the core workflow — real-time order sync, unified kitchen display, and automated financial reconciliation — has been built using **Google Sheets + Apps Script** to simulate the middleware layer.

**[View Live Demo →](https://docs.google.com/spreadsheets/d/1i9kTp1SCWkNQZ-W0c8lRU1hvbQlYgSdq4f3X9R1L90g/edit?usp=sharing)**

The prototype demonstrates:
- `Incoming_Orders` — Grab orders auto-synced with status tracking
- `Loyverse_Orders` — Walk-in orders from POS
- `Kitchen_Display` — Unified view merging both order streams
- `Daily_Summary` — Auto-generated financial report with commission breakdown
- `Sync_Log` — Full event log including error and retry events
- `Financial_History` — Permanent daily record, never overwritten
- `Order_History` — Archived orders with date stamps

---

## Expected Impact

| Area | Before | After |
|---|---|---|
| Order entry | Manual re-typing per order | Fully automatic, under 3 seconds |
| Kitchen visibility | Grab tablet + POS separate | One unified screen |
| Daily reconciliation | 30–60 min manual work | Zero — auto-generated |
| Customer tracking | Not possible | UUID-based profiles built automatically |
| Accounting export | Manual data transfer | Scheduled, verified, automatic |
| Error risk | High — human copy-paste | Zero — system-verified math |

---

## Why This Works

Most integration solutions are built for ideal conditions. This one is built for reality.

Three hard truths about retail technology shaped every architectural decision:

**Internet connections fail.** The order queue ensures no transaction is lost — orders hold locally and sync the moment connectivity returns.

**Data privacy is non-negotiable.** UUID tracking builds full customer intelligence without ever accessing or storing restricted personal data.

**Staff resist change.** Nothing in the frontline workflow has changed. The Data Bridge does all the heavy lifting invisibly, in the background.

The result is a system that is resilient by design, compliant by default, and ready for production from day one — with zero retraining, zero disruption, and zero tolerance for data loss.

---

## Project Timeline

| Phase | Timeline | Deliverable |
|---|---|---|
| Phase 1 | Apr 2026 | Webhook listener + data transformation |
| Phase 2 | May 2026 | UUID tracking + customer database |
| Phase 3 | Jun 2026 | Financial reconciliation + Infotech export |
| Phase 4 | Jun 2026 | Flutter dashboard + full system testing |

*Estimated project start: Apr–Jun 2026 (subject to change)*

---

## Submitted For

**DXP Challenge 2026**
Deadline: 19 March 2026, 6PM
Submission: [https://forms.gle/ad4Yt8Y36rqBwFFL9](https://forms.gle/ad4Yt8Y36rqBwFFL9)
