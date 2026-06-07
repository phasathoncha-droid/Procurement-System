# Feature: Vendor Portal — Billing Submission

## Module
Billing

## Status
Phase 2

## Overview
The vendor portal is a self-service web portal where approved vendors can view all their POs, track transaction history, submit invoices against a PO, contract, or memo, and monitor billing status in real time. This replaces email and phone-based invoice submission and reduces back-and-forth between vendors and the Procurement / Accounting team.

## Solution Description

### Vendor Access and Authentication

Vendor access is invite-only — tied directly to the Vendor master in Vendor Management.

**Onboarding flow:**
1. Procurement adds a portal user (name + email) under the vendor record in Vendor Management
2. System sends an invite email with a registration link — **no expiry**
3. Vendor clicks the link, sets a password, and gains access to the portal
4. The portal account is permanently linked to the vendor's master record — VAT status and WHT rate load automatically from it

Only emails registered under the vendor's account can access the portal. Vendors cannot self-register without an invite.

**Multi-user per vendor:**
One vendor can have multiple portal users (e.g. finance contact + sales contact). Procurement manages vendor users from Vendor Management:
- Add a new portal user (name + email) under any vendor record
- Remove a user
- Reset a user's password
- Each user receives their own invite link

All users under the same vendor are **Submitters** — they can submit invoices and view all their vendor's data. There is no admin tier on the vendor side. Only Procurement manages user access.

---

### Portal Pages

**1. PO List**
All Purchase Orders issued to this vendor. Read-only — vendor cannot edit PO content.

| Reference | Type | Date | Items | Total Amount | Remaining | Billing Status | Action |
|---|---|---|---|---|---|---|---|
| PO-0042 | PO | 2026-05-10 | 3 | 145,000 | 27,300 | Partial | Submit Invoice |
| PO-0038 | PO | 2026-05-20 | 1 | 12,500 | 0 | Complete | — |
| CONTRACT-2026-003 | Contract | 2026-04-01 | 3 | 480,000 | 180,000 | Partial | Submit Invoice |
| MEMO-2026-011 | Memo | 2026-05-15 | 2 | 75,000 | 75,000 | Not Started | Submit Invoice |

Columns:
- **Reference** — PO number, Contract reference, or Memo reference. Clickable — opens detail panel.
- **Type** — PO / Contract / Memo badge
- **Date** — date document was created
- **Items** — number of line items
- **Total Amount** — total PO/Contract/Memo value
- **Remaining** — amount still billable (Total − confirmed billings − pending billings). Shows ฿ 0 when Billing Status is Complete.
- **Billing Status** — Not Started / Partial / Complete
- **Action** — Submit Invoice button; hidden when Billing Status is Complete

PO detail page shows:
- PO header (vendor, date, total)
- Line items (item name, qty, unit price, amount)
- GR history (date, qty received per item) — read-only
- Billing history (invoices submitted and their status)

---

**2. Submit Invoice**
Vendor submits an invoice against a PO, contract, or memo. The form adapts based on reference type.

**Reference type selection (step 1):**

| Type | When to use |
|---|---|
| **PO** | Invoice against a specific Purchase Order |
| **Contract** | Invoice against an ongoing contract — vendor enters contract number |
| **Memo** | Invoice against an internal memo — vendor enters memo reference |

When **PO** is selected, the form pre-fills from the PO and enforces the PO amount ceiling. When **Contract** or **Memo** is selected, the vendor enters the reference number manually and a contract/memo value must be entered by Procurement when the reference is registered — the same ceiling rule applies (cumulative invoices cannot exceed the contract/memo value).

**Form fields:**

| Field | Source | PO | Contract / Memo | Notes |
|---|---|---|---|---|
| Reference Type | Vendor selects | PO / Contract / Memo | — | |
| PO Number | Pre-filled | ✅ mandatory | — | Hidden for Contract/Memo |
| Contract / Memo Ref. | Vendor enters | — | ✅ mandatory | Hidden for PO |
| Vendor Name | Pre-filled | ✅ | ✅ | From Vendor master |
| Invoice Number | Vendor enters | ✅ | ✅ | Must be unique per vendor |
| Invoice Date | Vendor enters | ✅ | ✅ | Date on vendor's invoice |
| Line items to bill | Vendor selects | ✅ | ✅ | PO: select from PO lines. Contract/Memo: free-text description + amount |
| Net Amount | Vendor enters / calculated | ✅ | ✅ | Amount before VAT |
| VAT | System calculates | ✅ | ✅ | 7% or 0% from Vendor master |
| Invoice Total | System calculates | ✅ | ✅ | Net + VAT — read-only |
| Invoice Document | Vendor uploads | ✅ mandatory | ✅ mandatory | PDF of vendor's tax invoice |
| Copy of PO | Vendor uploads | ✅ mandatory | — | Not required for Contract/Memo |
| Supporting Document | Vendor uploads | optional | optional | Delivery note, contract copy, etc. |
| Remarks | Vendor enters | optional | optional | Free text |

**VAT use cases — vendor enters net amount 100:**

| Case | Net Amount | VAT | Invoice Total |
|---|---|---|---|
| VAT vendor (7%) | 100.00 | +7.00 | 107.00 |
| Non-VAT vendor (0%) | 100.00 | +0.00 | 100.00 |

VAT is read-only on the form — set by Procurement and not editable by the vendor. Payment and WHT are out of scope for this system — handled by Bookkeeping when processing payment.

**PO amount ceiling rule:**
- The sum of all invoices (confirmed + pending) against a PO cannot exceed the PO value.
- Exception: a vendor may submit an invoice that brings the cumulative total up to **PO value − 1 THB**, and the system will treat the PO as fully billed. This allows minor rounding differences without requiring a PO amendment.
- If a submission would cause the cumulative total to exceed the PO value, the system blocks submission and shows an error with the remaining billable amount.

**Financial summary panel (on PO detail):**
Shows the billing position against this PO at a glance:

| Field | Value |
|---|---|
| PO Value | ฿ 145,000 |
| GR Value (delivered) | ฿ 90,250 |
| Total Billed (confirmed) | ฿ 85,600 |
| Pending (under review) | ฿ 32,100 |
| Remaining Billable | ฿ 27,300 |

Once submitted, the invoice enters the Procurement review queue.

---

**3. My Invoices**
Full history of all invoices submitted by this vendor. Vendor tracks status here without contacting Procurement.

| Invoice No. | Invoice Date | Reference | Type | Net Amount | Invoice Total | Status | Action |
|---|---|---|---|---|---|---|---|
| INV-2024-001 | 2026-06-01 | PO-0042 | PO | 80,000 | 85,600 | Confirmed | — |
| INV-2024-002 | 2026-06-03 | PO-0038 | PO | 50,000 | 50,000 | Waiting Review | — |
| INV-2024-003 | 2026-06-04 | PO-0042 | PO | 30,000 | 32,100 | Waiting Review | — |

**Invoice status values (vendor-facing):**

| Status | Meaning |
|---|---|
| Waiting Review | Submitted — being processed internally (Procurement or Accounting) |
| Confirmed | Accounting confirmed — billing is posted to Bookkeeping |
| Rejected | Permanently rejected — vendor must submit a new invoice |

**Why 3 statuses:** Vendors do not need visibility into internal routing between Procurement and Accounting. From the vendor's perspective, the invoice is either being processed, confirmed, or rejected. Exposing "Waiting Confirmation" would leak internal workflow details that are irrelevant to the vendor. Status labels are intentionally simple for non-technical vendor users.

**Note:** Return to vendor is **not supported in Phase 1**. If Procurement finds an error in an external submission, they reject it with a mandatory comment. The vendor submits a corrected invoice. The "Edit and Resubmit" flow is internal (Procurement only) and not exposed to the vendor portal.

---

### Approval Flow

1. Vendor submits invoice → enters **Procurement Review queue** (status: Waiting Procurement Review)
2. Procurement reviews:
   - **Forward to Accounting** → status: Waiting Accounting Confirmation
   - **Reject** → status: Rejected (permanent). Vendor must submit a new invoice. No return to vendor in Phase 1.
3. Accounting reviews:
   - **Confirm** → status: Confirmed. Bookkeeping AP transaction posted. Vendor sees Confirmed.
   - **Reject** → status: Rejected (permanent). Mandatory comment required.

---

### Notifications

Vendor receives an email notification when:
- Invoice is submitted successfully
- Invoice is Rejected (by Procurement or Accounting) — with mandatory rejection comment
- Invoice is Confirmed by Accounting

---

## Acceptance Criteria
- **Access:** Vendor can only access the portal via an invite. No self-registration without invite.
- **Multi-user:** One vendor can have multiple portal users. Procurement adds/removes/resets users from Vendor Management. All users under the same vendor share the same data view.
- **Invite:** Invite link has no expiry. Remains valid until the vendor registers. Procurement manages access by adding or removing users.
- **VAT:** Set by Procurement — 7% or 0%. Read-only for vendor. Not editable on billing form.
- **PO List:** Vendor sees all POs, Contracts, and Memos issued to them. Content is read-only.
- **Remaining on list:** Calculated as Total − confirmed billings − pending billings. Forced to ฿ 0 when Billing Status is Complete.
- **Billing Status:** Not Started / Partial / Complete — derived from confirmed + pending billings against the total value.
- **Reference types:** Vendor can submit invoice against a PO, a Contract (reference number), or a Memo (reference number).
- **PO ceiling rule:** Sum of all invoices (confirmed + pending) against a PO cannot exceed PO value. Exception: cumulative total reaching PO value − 1 THB is treated as fully billed. Submissions that would exceed PO value are blocked with an error showing remaining billable amount.
- **Financial summary:** PO detail shows PO Value, GR Value, Total Billed (confirmed), Pending (under review), and Remaining Billable.
- **Invoice submission — PO type:** Invoice Number, Invoice Date, at least one line item, Invoice Document (PDF), and Copy of PO (PDF) are mandatory.
- **Invoice submission — Contract/Memo type:** Invoice Number, Invoice Date, Contract/Memo reference, at least one line item description and amount, and Invoice Document (PDF) are mandatory. Copy of PO is not required.
- **Partial billing:** Vendor can submit invoice for a subset of PO line items or partial quantity.
- **No return to vendor in Phase 1:** Procurement rejects with mandatory comment. Vendor submits a new invoice.
- **Accounting rejection:** Mandatory comment. Status = Rejected. Dead end.
- **Invoice statuses:** Waiting Review / Confirmed / Rejected. Vendor sees only 3 statuses — internal routing between Procurement and Accounting is not exposed.
- **My Invoices:** Vendor tracks all submitted invoices and their current status without contacting Procurement.
- **Notifications:** Vendor receives email on: submission successful, rejected (with comment), confirmed.
- **Audit trail:** Every action (submit, forward, confirm, reject) is timestamped and recorded per invoice.

## Process Flow
Reference: [vendor-portal-billing.md](../../04_diagrams/process-flows/vendor-portal-billing.md)

## Decisions Log
- **Multi-user per vendor** — ✅ Confirmed. One vendor can have multiple portal users. Procurement manages from Vendor Management.
- **Non-PO invoices** — ✅ Confirmed. Vendors can submit against PO, Contract, or Memo. Ceiling rule applies to all types.
- **Invite expiry** — ✅ No expiry. Link stays valid until vendor registers. Procurement manages access by adding/removing users.
- **Contract/Memo ceiling** — ✅ Capped. Same rule as PO: cumulative invoices cannot exceed the registered contract/memo value (± 1 THB tolerance).
- **Multi-user roles** — ✅ Submitter only. No admin tier on the vendor side. Procurement controls all user management.
- **PO over-billing** — ✅ Confirmed. Blocked. Exception: PO value − 1 THB tolerance counts as fully billed.
- **Invoice statuses** — ✅ Confirmed. Waiting Review / Confirmed / Rejected. Internal routing (Procurement → Accounting) is not exposed to the vendor — both internal queues map to "Waiting Review" from the vendor's perspective.
- **Deposit tracking** — ✅ Out of scope. Cash/deposit tracking not required.
- **Approval flow** — ✅ Follows current internal flow: Vendor → Procurement → Accounting → Bookkeeping.
- **WHT certificate** — Out of scope. Handled by Bookkeeping. Consider Phase 3.

## Open Questions
None outstanding.

## Related Features
- [Billing Creation and Confirmation](001-billing-creation.md)
- [PO Creation and Approval](../../02_features/PO-Purchase-Order/001-po-creation-and-approval.md)
