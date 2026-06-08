# Feature: Billing Creation and Confirmation

## Module
Billing

## Status
Built (inside PO page) — enhancements planned (see Enhancements section)

## Implementation Status

| # | Feature | Status |
|---|---------|--------|
| — | Billing recording inside PO page: invoice date, amount, attachment, remark | Built |
| — | Partial billing: multiple billing records per PO supported | Built |
| E1 | Create Billing form and Queue page | Pending |
| E2 | Billing List page: Document View and Line Item View | Pending |
| E3 | Approval chain: confirmation, return, and reject flow | Pending |
| E4 | Billing reversal flow | Pending |
| E5 | Bookkeeping integration: S3 file posting on confirmation and reversal | Pending |

## Overview
After a PO is created, Procurement records billing against it by entering the vendor's invoice details. Billing is independent of GR — both can happen in any order. Partial billing is supported (multiple invoices per PO). Once Accounting confirms, the record moves to the Billing List and an accounting file is uploaded to S3 for Bookkeeping to record. Confirmed invoices can be cancelled by Accounting via a Reversal.

## Solution Description

**Two Billing Paths**

- **Path A — Internal (Procurement-initiated):** Procurement creates a billing record directly in the system against a PO. It goes straight to Accounting for confirmation — no Procurement review step.
- **Path B — Vendor Portal (Phase 2):** The vendor submits a billing record through the vendor self-service portal. Procurement reviews it first, then forwards to Accounting for confirmation.

**Billing Statuses**

There are exactly 6 statuses for a billing record:

| Status | Where shown | Description |
|---|---|---|
| Waiting Procurement Review | Queue | External (Path B) only — vendor submitted, waiting for Procurement |
| Waiting Accounting Confirmation | Queue | Internal lands here directly; External arrives after Procurement forwards. Also used when Accounting returns an Internal billing |
| Rejected | Queue (hidden by default) | Permanently cancelled by Procurement or Accounting — dead end |
| Confirmed | Billing List | Accounting confirmed — AP transaction posted to Bookkeeping |
| Reversed | Billing List | Original invoice that was cancelled — offset by a Reversal record |
| Reversal | Billing List | Counter-entry that zeroes out the Reversed invoice — shown with negative amounts in red |

Return is an action, not a status. When Accounting returns an Internal billing, the status stays Waiting Accounting Confirmation. The detail modal detects a return by the presence of Accounting's return comment and shows Procurement an "Edit & Resubmit" button instead of read-only view.

Queue default view shows Waiting Procurement Review + Waiting Accounting Confirmation only. Rejected records are hidden by default — user must filter by Status to see them.

**Billing Form Fields**

The form is a 3-step flow: Step 1 — Select PO → Step 2 — Invoice Details → Step 3 — Documents

| Field | Who fills | Required | Notes |
|---|---|---|---|
| PO selection | User | Yes | Search by PR number, PO number, or vendor name |
| Invoice Number | User | Yes | Must be unique per vendor across all POs (Thai Revenue Dept. requirement) |
| Invoice Date | User | Yes | Date printed on vendor's tax invoice |
| Line items | User | Yes | Checkbox-select which PO line items are included in this invoice |
| Net Amount (per line) | User | Yes | Amount before VAT — entered by user from vendor's invoice |
| VAT Amount | System | — | Auto-calculated: Net × VAT Rate. Shown to 2 decimal places. Not editable |
| Invoice Total | System | — | Net Amount + VAT Amount. Shown to 2 decimal places |
| Invoice Document | User | Yes | PDF of vendor's tax invoice |
| Copy of PO | User | Yes | PDF of the Purchase Order |
| Delivery Goods Document | User | No | Delivery note or receipt — optional |
| Remarks | User | No | Free text |

VAT flag is inherited from PR price comparison stage — 7% (VAT-registered vendor) or 0% (non-VAT vendor). Not editable at billing. VAT Amount = `Math.round(Net × Rate) / 100`. 0% VAT vendors show "—" in the VAT column, not 0.00.

A progress bar per line item shows cumulative billed amount vs PO line amount. Turns red if current billing would exceed the PO line amount.

**Partial Billing**

Multiple billing records can be created against a single PO. PO Billing Status reflects cumulative billed amount:

| PO Billing Status | Meaning |
|---|---|
| Not Started | No billing recorded against this PO |
| Partial | Some amount billed, not fully billed |
| Complete | Total billed equals PO amount |

**Actions by Role and Path**

| Situation | Who | Available Actions |
|---|---|---|
| Waiting Procurement Review (External) | Procurement | Forward to Accounting · Reject (mandatory comment) |
| Waiting Accounting Confirmation | Accounting | Confirm · Return to Procurement (Internal only, mandatory comment) · Reject (mandatory comment) |
| Waiting Accounting Confirmation + return comment (Internal) | Procurement | Edit & Resubmit → back to Waiting Accounting Confirmation |
| Confirmed | Accounting | Cancel Invoice → triggers Reversal flow |
| Any status | Procurement / Accounting | View details (read-only) |

**Billing Reversal**

After a billing is Confirmed, Accounting can cancel it:
- Accounting clicks **Cancel Invoice** on the confirmed record's detail modal
- A cancellation modal shows invoice details + warning that this cannot be undone
- Mandatory reason required (minimum 5 characters)
- On confirm: original record status → Reversed; a new Reversal counter-record is created with the same invoice number + "-R" suffix and negative amounts
- Reversal is posted to Bookkeeping to offset the original AP entry

Authority: Accounting only. No second approval required. This is not a credit note — Reversal = full cancellation of the AP liability.

**Return (Accounting → Procurement)**

Return is an action, not a status. Accounting returns an Internal billing to Procurement with a mandatory comment. Procurement edits and resubmits → status resets to Waiting Accounting Confirmation. Does not restart the Procurement review step. Return to vendor portal is not supported in Phase 1 — incorrect external submissions must be rejected.

**Queue Tab**

Shows all active billing records. Default filter: Waiting Procurement Review + Waiting Accounting Confirmation. Rejected records hidden by default.

| Column | Notes |
|---|---|
| Invoice No. | |
| PO Number | |
| Vendor | |
| Source | Internal / External badge |
| Requester | Who created / submitted the billing |
| Net Amount | Right-aligned, 2dp |
| Invoice Total | Right-aligned, 2dp |
| Status | Badge |
| Submitted | Date |
| Action | View button |

Queue filters: Status (multi-select), Source (Internal / External)

**Billing List Tab**

Shows Confirmed, Reversed, and Reversal records only. Supports Document View and Line Item View.

Document View — one expandable row per invoice:

| Invoice Date | Invoice No. | PO Number | Vendor | VAT Rate | Net Amount | VAT Amt | Invoice Total | Status | Action |
|---|---|---|---|---|---|---|---|---|---|
| 2026-06-01 | INV-0001 | PO-0042 | ABC Co. | 7% | 100,000 | 7,000 | 107,000 | Confirmed | View |
| 2026-06-03 | INV-0002 | PO-0038 | XYZ Co. | 0% | 50,000 | — | 50,000 | Reversed | View |
| 2026-06-03 | INV-0002-R | PO-0038 | XYZ Co. | 0% | -50,000 | — | -50,000 | Reversal | View |

> Reversal invoice date is system-set to the same date as the original invoice — not editable.

Line Item View — one flat row per line item. Reversal rows show negative amounts in red.

Billing List filters: Status (Confirmed / Reversed / Reversal), VAT Rate (7% / 0%)

**Bookkeeping Integration**

After Accounting confirms, the system posts an AP liability transaction to Bookkeeping. After a Reversal, the system posts an offsetting transaction to zero out the original AP entry. If the send fails, the billing record is saved and flagged for retry. Transaction payload TBD with Bookkeeping team.

## Acceptance Criteria
- **Eligibility:** Billing can only be created against a PO with status = Created. No dependency on GR status.
- **Path A flow:** Procurement submits → Waiting Accounting Confirmation → Accounting confirms → Confirmed → Bookkeeping transaction posted.
- **Path B flow (Phase 2):** Vendor submits → Waiting Procurement Review → Procurement forwards → Waiting Accounting Confirmation → Accounting confirms → Confirmed → Bookkeeping posted.
- **Reject:** Both Procurement (Path B) and Accounting can reject with mandatory comment. Status = Rejected. Dead end.
- **Rejected visibility:** Hidden by default in Queue. User must filter by Status = Rejected to see them.
- **Return (Internal only):** Accounting returns Internal billing with mandatory comment. Procurement edits and resubmits → Waiting Accounting Confirmation. No return to vendor portal in Phase 1.
- **Mandatory fields:** Invoice Number, Invoice Date, Net Amount per line, Invoice Document (PDF), Copy of PO (PDF).
- **Invoice Number uniqueness:** Unique per vendor across all POs.
- **VAT auto-calculation:** VAT Amount = Net × VAT Rate, 2 decimal place precision. Not editable at billing. 0% VAT shows "—" not "0.00".
- **Partial billing:** Multiple billing records per PO. Progress bar per line shows cumulative billed vs PO line amount. Turns red if billing would exceed PO line amount.
- **Billing Reversal:** Accounting only. Mandatory reason (≥5 characters). Creates Reversal counter-record with negative amounts dated same as original. Posts offsetting transaction to Bookkeeping. Cannot be undone.
- **Billing List statuses:** Confirmed, Reversed, Reversal. Reversal rows show negative amounts in red with red-tinted background.
- **Bookkeeping integration:** Confirmed billing triggers AP transaction. Send failure saves record and flags for retry.

## Process Flow

```mermaid
flowchart TD
    A([PO Status: Created])
    A --> B1[Path A: Procurement creates billing]
    A --> B2[Path B: Vendor submits via portal\nPhase 2]

    B1 --> C[Waiting Accounting Confirmation]

    B2 --> D{Procurement Review}
    D -->|Reject| RJ([Rejected — dead end])
    D -->|Forward| C

    C --> E{Accounting Decision}
    E -->|Confirm| F[Confirmed → Billing List\nPost AP to Bookkeeping]
    E -->|Return| G[Back to Procurement\nwith comment]
    E -->|Reject| RJ
    G -->|Edit & Resubmit| C

    F --> H{Accounting cancels?}
    H -->|Cancel Invoice + reason| I[Reversed + Reversal created\nOffset posted to Bookkeeping]

    style A fill:#27AE60,color:#fff
    style F fill:#27AE60,color:#fff
    style I fill:#94A3B8,color:#fff
    style RJ fill:#EF4444,color:#fff
    style B2 fill:#8E44AD,color:#fff
```

## Enhancements

### Context
The current flow records billing inside the PO page with no invoice number, no approval chain, and no Bookkeeping integration. E1–E5 build billing as a standalone module with a proper Create Billing form, Queue, Billing List, approval chain, reversal flow, and Bookkeeping integration.

---

### E1 — Create Billing Form and Queue Page

**Status:** Pending

Replaces billing recording inside the PO page with a dedicated Billing page accessible from main navigation. Includes a Create Billing 3-step modal and a Queue page showing active billing records awaiting action.

**Acceptance Criteria:**

*Create Billing — 3-step modal*
- "+ Create Billing" button opens the Create Billing modal.
- Step 1 — Select PO: user searches by PR number, PO number, or vendor name. Live dropdown appears when at least one character is typed.
- Step 2 — Invoice Details: user fills in invoice information. Mandatory fields: Invoice Number, Invoice Date, at least one line item selected, Net Amount per selected line item.
- Step 3 — Documents: user uploads supporting files. Mandatory: Invoice Document (PDF), Copy of PO (PDF). Optional: Delivery Goods Document, Remarks.
- Submit button is disabled until all mandatory fields are filled and mandatory files are uploaded.

*Create Billing — form fields*

| Field | Required | Notes |
|---|---|---|
| PO selection | Yes | Search by PR number, PO number, or vendor name |
| Invoice Number | Yes | Must be unique per vendor across all POs (Thai Revenue Dept. requirement). Duplicate blocked with inline error. |
| Invoice Date | Yes | Date printed on vendor's tax invoice |
| Line items | Yes | Checkbox-select which PO line items are included. At least one required. |
| Net Amount (per line) | Yes | Amount before VAT — entered by user from vendor's invoice |
| VAT Amount | Auto | Calculated: Net × VAT Rate. 2 decimal places. Not editable. |
| Invoice Total | Auto | Net Amount + VAT Amount. 2 decimal places. |
| Invoice Document | Yes | PDF of vendor's tax invoice |
| Copy of PO | Yes | PDF of the Purchase Order |
| Delivery Goods Document | No | Delivery note or receipt |
| Remarks | No | Free text |

- VAT rate is inherited from PR price comparison — 7% or 0%. Not editable at billing.
- A progress bar per line item shows cumulative billed amount vs PO line amount. Turns red if billing would exceed the PO line amount.

*Queue page*
- Default view shows Waiting Procurement Review and Waiting Accounting Confirmation records only.
- Rejected records are hidden by default — user must filter by Status = Rejected to see them.
- Columns: Invoice No., PO Number, Vendor, Source (Internal / External), Requester, Net Amount, Invoice Total, Status, Submitted date, View action.
- Filters: Status (multi-select), Source (Internal / External).

---

### E2 — Billing List Page

**Status:** Pending

Dedicated Billing List page showing all Confirmed, Reversed, and Reversal records with Document View and Line Item View.

**Acceptance Criteria:**

- Shows Confirmed, Reversed, and Reversal records only.
- Filters: Status (Confirmed / Reversed / Reversal), VAT Rate (7% / 0%).

Document View — one expandable row per invoice:

| Invoice Date | Invoice No. | PO Number | Vendor | VAT Rate | Net Amount | VAT Amt | Invoice Total | Status | Action |
|---|---|---|---|---|---|---|---|---|---|
| 2026-06-01 | INV-0001 | PO-0042 | ABC Co. | 7% | 100,000 | 7,000 | 107,000 | Confirmed | View |
| 2026-06-03 | INV-0002 | PO-0038 | XYZ Co. | 0% | 50,000 | — | 50,000 | Reversed | View |
| 2026-06-03 | INV-0002-R | PO-0038 | XYZ Co. | 0% | -50,000 | — | -50,000 | Reversal | View |

> Reversal invoice date is system-set to the same date as the original invoice — not editable.

Each row expands to show line items. Reversal rows show negative amounts in red with red-tinted background.

Line Item View — one flat row per line item across all invoices. Reversal rows appear in date order with negative amounts.

---

### E3 — Approval Chain

**Status:** Pending

Adds a full approval chain between Procurement submission and billing being Confirmed: Accounting confirmation, return to Procurement for correction, and permanent rejection by either role.

**Acceptance Criteria:**

*Confirmation*
- Internal billing submitted by Procurement lands at Waiting Accounting Confirmation immediately.
- External billing (Path B) lands at Waiting Accounting Confirmation after Procurement forwards it.
- Accounting can Confirm the billing — status moves to Confirmed and Bookkeeping transaction is posted.

*Return*
- Accounting can return an Internal billing with a mandatory comment. Status remains Waiting Accounting Confirmation.
- Procurement sees an "Edit & Resubmit" button in the detail modal when a return comment is present.
- After Procurement edits and resubmits, status resets to Waiting Accounting Confirmation.
- Return to vendor portal is not supported in Phase 1 — incorrect external submissions must be rejected.

*Reject*
- Procurement can reject an External billing at Waiting Procurement Review with a mandatory comment.
- Accounting can reject any billing at Waiting Accounting Confirmation with a mandatory comment.
- Rejected status is permanent — no further actions available. Dead end.
- Rejected records are hidden by default in the Queue. User must filter by Status = Rejected to see them.

---

### E4 — Billing Reversal Flow

**Status:** Pending

Allows Accounting to cancel a Confirmed billing by creating a Reversal counter-record with negative amounts. Procurement cannot trigger a reversal — this is Accounting authority only.

**Acceptance Criteria:**

- "Cancel Invoice" button is available only on Confirmed records in the Billing List, visible to Accounting only. Procurement cannot reverse a billing.
- Accounting must enter a mandatory cancellation reason (≥5 characters). Cannot be undone.
- On confirm: original record status → Reversed; a Reversal counter-record is created with the same invoice number + "-R" suffix and negative amounts.
- Reversal record invoice date is system-set to the same date as the original invoice — not editable.
- Reversal counter-record appears in the Billing List in date order — not nested under the original.

---

### E5 — Bookkeeping Integration

**Status:** Pending

Posts accounting transactions to Bookkeeping on both billing confirmation and reversal. Failed posts are flagged for retry without blocking the billing record.

**Acceptance Criteria:**

- Every Confirmed billing triggers an AP liability transaction to Bookkeeping immediately after confirmation.
- Every Billing Reversal triggers an offsetting transaction to Bookkeeping to zero out the original AP entry.
- If the Bookkeeping post fails, the billing record is still saved. The failed record is flagged internally for retry.
- Transaction payload format to be defined with the Bookkeeping team (TBD).

---

## Decisions Log
- **Invoice uniqueness** — ✅ Unique per vendor — aligns with Thai Revenue Dept. tax invoice requirements.
- **Return is an action not a status** — ✅ Status stays Waiting Accounting Confirmation. Return comment triggers Edit & Resubmit UI for Procurement.
- **No return to vendor portal in Phase 1** — ✅ Incorrect external submissions are rejected; vendor resubmits new invoice.
- **Reject is permanent** — ✅ Both Procurement and Accounting can reject with mandatory comment.
- **Rejected hidden by default** — ✅ Reduces noise in Queue. User must filter to see them.
- **Reversal ≠ Credit Note** — ✅ Credit note = partial discount. Reversal = full cancellation of AP liability.
- **Cancellation authority** — ✅ Accounting only. No second approval needed. Mandatory reason for audit trail.
- **Monetary precision** — ✅ 2 decimal places throughout. `Math.round(net × rate) / 100` to avoid floating point loss.
- **0% VAT display** — ✅ VAT column shows "—" not "0.00" for non-VAT vendors.
- **Due Date** — ✅ Out of scope — handled by Bookkeeping.

## Open Questions
- [ ] **Bookkeeping transaction payload:** Fields and format TBD with Bookkeeping team.
- [ ] **Bookkeeping retry:** Manual retry button or automatic background retry?
- [ ] **Rejected record retention:** How long do Rejected records stay visible in the Queue?

## Related Features
- [PO Creation and Approval](../../02_features/PO-Purchase-Order/001-po-creation-and-approval.md)
- [GR Recording](../../02_features/GR-Good-Receipt/001-gr-recording.md)
- [Vendor Portal Billing](002-vendor-portal-billing.md)
