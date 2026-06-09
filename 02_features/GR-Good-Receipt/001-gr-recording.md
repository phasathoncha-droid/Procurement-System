# Feature: Good Receipt Recording

## Module
GR — Good Receipt

## Status
Built — enhancements planned (see Enhancements section)

## Implementation Status

| # | Feature | Status |
|---|---------|--------|
| — | GR recording inside PO page: qty received, GR date, remark, attachment | Built |
| — | Partial receipt: multiple GRs per PO supported | Built |
| E1 | Standalone GR List page with Document View, Line Item View, and delivery note field | Pending |
| E2 | GR Reversal flow | Pending |
| E3 | Bookkeeping integration: S3 file posting on GR confirmation and reversal | Pending |

## Overview
After a PO is Created, the requester or Procurement team records that goods or services have been received against that PO. Receipts can be partial or full. Each confirmed GR generates an accounting file uploaded to S3 for Bookkeeping to record. GR records are immutable — corrections are handled through a GR Reversal document.

## Solution Description

**GR Entry Point**
GR is recorded from the **GR List page** via the **"+ Record GR"** button in the toolbar. Clicking it opens the Record GR modal. Users do not record GR from the PO page directly.

**Record GR Modal — 3-Step Flow**

*Step 1 — Search*
A single search field lets the user find a PR or PO to receive against. As the user types, a grouped live dropdown appears with two sections:
- **Purchase Requests** — matches by PR number, topic, or requester name. Each result shows the PR number, topic, requester, and remaining units across all its POs.
- **Purchase Orders** — matches by PO number or vendor name. Each result shows the PO number, vendor, linked PR, and remaining units.

The dropdown only opens when the user has typed at least one character. It closes when the input is cleared.

| User clicks | Next step |
|---|---|
| A PR result | PR card appears + list of POs under that PR. User selects a PO. |
| A PO result | Skips the PO list step — goes directly to the items table. |

*Step 2 — PO Selection (only if PR was chosen)*
After selecting a PR, the system shows:
- A gray PR card at the top (PR number + topic + requester) with a ✕ button to go back to search
- A list of POs under that PR, each showing PO number, vendor, date, item count, PO total, and remaining units
- User clicks a PO row or its **Select** button to advance

*Step 3 — Receive Items*
After a PO is selected, the modal shows:
1. A blue PO summary card (PO number, vendor, PO date, PO total) with a ✕ to go back to PO list
2. **Items to Receive** table
3. **Receipt Details** form

**Items to Receive Table**
GR is recorded by entering a **quantity to receive** per line item. The system calculates cumulative GR progress as a percentage of the total ordered quantity and displays it as a live progress bar.

| Column | Notes |
|---|---|
| # | Row number |
| Item Name | From PO line item |
| Type | Asset or Expense badge |
| GL / Asset No. | Asset number (free text) or GL code from Finance Coding stage |
| Line Total | Total value of this line item (ordered qty × unit price) — for reference |
| Ordered Qty | Total quantity on the PO line — for reference |
| Remaining Qty | Ordered qty minus all previously confirmed GR quantities. Bold when > 0, muted when 0. |
| Qty to Receive | Numeric input. Cannot exceed Remaining Qty. Error state shown inline. Disabled (shows —) if Remaining Qty = 0. |
| GR Progress | Live progress bar showing cumulative receipt status. Grey segment = already received in prior GRs. Navy segment = this GR (grows as user types). Label shows total % after this GR. Turns red if quantity exceeds remaining. Fully received items show a solid green 100% bar. |

Fully received items are visually dimmed and the input is disabled.

**Live footer totals** (appear once any quantity > 0 is entered):
- Est. Amount (sum of `qty to receive × unit price` per item)

**Receipt Details Form**

| Field | Required | Notes |
|---|---|---|
| Receipt Date | Mandatory | Defaults to today, editable |
| Delivery Note No. | Optional | Reference from vendor's delivery slip |
| Delivery Document | Optional | File attachment — PDF, JPG, PNG, max 10 MB |
| Remarks | Optional | Free text |

**Confirm Receipt** button is disabled until at least one item has a valid qty entered. Clicking it creates the GR record, posts to Bookkeeping, and shows a toast notification.

**Partial Receipt**
The system supports partial receipt — the user can record a quantity less than the remaining quantity for any item. The PO remains open for additional GR entries. PO GR Status reflects cumulative received quantity:
- **Not Started** — no GR recorded yet
- **Partially Received** — some items received, not all
- **Fully Received** — all line items fully received

**GR is Final**
Once confirmed, a GR record cannot be edited. To correct a wrong entry, the user creates a GR Reversal.

**GR Detail Modal**
Clicking the **pencil icon** on any GR row opens the GR Detail modal — a read-only view of the full GR record. It shows:
- **Header:** GR number, GR date, status badge, PO number, vendor, total amount, delivery note number
- **Items Received table:** all line items with qty received, unit price, and amount. Reversal rows show negative quantities and amounts in red.
- **Receipt Details:** delivery document attachment (downloadable) and remarks

The modal is read-only. No fields can be edited.

| Status | Footer buttons |
|---|---|
| Confirmed | Close + **Cancel GR** (red) |
| Reversed | Close only |
| Reversal | Close only |

**GR Reversal**
If a GR was recorded incorrectly, the user opens the GR Detail modal via the pencil icon and clicks **Cancel GR**. The system:
1. Closes the detail modal and opens the Reversal confirmation modal showing the original GR details (GR number, PO, vendor, amount, date) and the reversal document number (e.g. GR-0001-R)
2. Requires a mandatory cancellation reason (minimum 5 characters) before the Confirm button enables
3. Creates a GR Reversal document with negative quantities and amounts, **dated the same as the original GR date** (system-set, not editable)
4. Posts an offsetting accounting transaction to Bookkeeping on the reversal date (= original GR date)

The GR Reversal is an independent document with its own GR number and date. It appears in the GR list in date order, not nested under the original.

The original GR is tagged **Reversed**. The reversal document is tagged **Reversal**. **Cancel GR** is only available on Confirmed GRs — Reversed and Reversal documents cannot be cancelled again.

**Who Can Record / Reverse GR**

| Action | Requester | Procurement | Accounting |
|---|---|---|---|
| Record GR | ✅ Own PRs only | ✅ Any PO | — |
| Reverse GR | ✅ Own PRs only | ✅ Any GR | — |
| View GR List | ✅ Own PRs only | ✅ All | ✅ All |

**Bookkeeping Integration**
Every confirmed GR and every GR Reversal triggers an accounting transaction to Bookkeeping (transaction payload TBD with Bookkeeping team). If the send fails, the GR is still saved and flagged internally for retry.

**GR List Page — Access by Role**

| Role | GR List Page | Transactions Visible | Record GR | Reverse GR |
|---|---|---|---|---|
| Procurement | ✅ | All GRs | ✅ | ✅ Any GR |
| Accounting | ✅ | All GRs | — | — |
| Requester | ✅ | Own PRs only | ✅ Own PRs only | ✅ Own PRs only |
| CFO / Procurement Manager | — | Via PO page only | — | — |

*Toolbar — Row 1 (Actions)*

| Element | Notes |
|---|---|
| Search input | Filter by GR number, PO number, or vendor |
| Export button | Exports the current view to Excel |
| View toggle | Document View / Line Item View |
| + Record GR | Opens the Record GR modal |

*Toolbar — Row 2 (Filters)*

| Filter | Values | Visibility |
|---|---|---|
| Date range | From / To date pickers | Both views |
| Status | All / Confirmed / Reversed / Reversal | Both views |
| Type | All / Asset / Expense | Line Item View only |
| Reset | Clears all filters | Both views |

*Document View (default) — one row per GR, expandable*

Collapsed row columns:

| GR Date | GR Number | PO Number | Vendor | Items | Total Amount | Delivery Note | Status | Action |
|---|---|---|---|---|---|---|---|---|
| 2026-06-01 | GR-0001 | PO-0042 | ABC Co. | 3 items | 145,000 | DN-2026-0042 | Reversed | ✏ |
| 2026-06-03 | GR-0002 | PO-0038 | XYZ Co. | 1 item | 12,500 | — | Confirmed | ✏ |
| 2026-06-01 | GR-0001-R | PO-0042 | ABC Co. | 3 items | -145,000 | DN-2026-0042 | Reversal | ✏ |

Expanded sub-row columns:

| Item Name | Type | Asset No. / GL Code | Qty Received | Unit Price | Amount |
|---|---|---|---|---|---|
| Laptop Dell | Asset | A-20240012 | 2 | 55,000 | 110,000 |
| HDMI Cable | Expense | 5210600010 ค่าซ่อมแซม | 5 | 500 | 2,500 |

The **pencil (✏) icon** appears on every row. Clicking it opens the GR Detail modal.

*Line Item View — one row per item, flat*

| GR Date | GR Number | PO Number | Vendor | Item Name | Type | Asset No. / GL Code | Qty Received | Unit Price | Amount | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| 2026-06-01 | GR-0001 | PO-0042 | ABC Co. | Laptop Dell | Asset | A-20240012 | 2 | 55,000 | 110,000 | Reversed |
| 2026-06-03 | GR-0002 | PO-0038 | XYZ Co. | Mouse | Expense | 5210500040 | 10 | 3,250 | 32,500 | Confirmed |
| 2026-06-01 | GR-0001-R | PO-0042 | ABC Co. | Laptop Dell | Asset | A-20240012 | -2 | 55,000 | -110,000 | Reversal |

Reversal rows appear in their own date order — not nested under the original.

## Acceptance Criteria
- **Eligibility:** GR can only be recorded against a PO with status = Created.
- **Who can record:** Requester of the PR or any Procurement team member.
- **GR entry point:** Via "+ Record GR" button on the GR List page — opens the Record GR modal.
- **Search:** The modal search field searches by PR number, topic, requester, PO number, or vendor name. Dropdown only appears when the user has typed at least one character.
- **PR → PO drill-down:** Selecting a PR shows all POs under it. User must then select a PO before entering quantities.
- **Direct PO selection:** Searching by PO number or vendor skips the PR/PO list step and goes directly to the items table.
- **Items table:** Shows all PO line items with line total, ordered qty, remaining qty, a qty to receive input, and a live GR progress bar. Fully received items (remaining qty = 0) are dimmed and the input is disabled.
- **Quantity input:** User enters a quantity to receive. Cannot exceed remaining qty. Error state shown inline; Confirm button stays disabled until resolved. GR progress bar updates live showing cumulative receipt % against total ordered qty.
- **Partial receipt:** User can record a qty less than remaining. System tracks cumulative received qty. PO GR Status updates (Not Started / Partially Received / Fully Received).
- **Multiple GRs per PO:** Each GR is an independent record. No limit per PO.
- **Receipt Details:** Receipt Date (mandatory, defaults to today), Delivery Note No. (optional), Delivery Document file (optional, PDF/JPG/PNG max 10 MB), Remarks (optional).
- **Confirm Receipt button:** Disabled until at least one item has a valid qty > 0 and no qty validation errors exist.
- **GR is final:** A confirmed GR cannot be edited.
- **GR Detail modal:** Opened via the pencil icon on any row. Read-only view of GR header, items received, delivery document, and remarks. Footer shows "Cancel GR" button only on Confirmed GRs.
- **GR Reversal:** Triggered by clicking "Cancel GR" inside the GR Detail modal. System opens reversal confirmation modal, requires mandatory reason (≥5 characters), creates GR Reversal with negative quantities and amounts dated the same as the original GR date (system-set, not editable), posts offsetting Bookkeeping transaction on that same date.
- **Reversal as independent document:** GR Reversal has its own GR number and date. Appears in GR list in date order — not nested under the original.
- **Cancel GR availability:** Available only on Confirmed GRs via the GR Detail modal. Reversed and Reversal documents cannot be cancelled again.
- **Bookkeeping integration:** Every confirmed GR and GR Reversal triggers an accounting transaction to Bookkeeping. If send fails, GR is saved and flagged for retry.
- **GR List visibility:** Visible to Accounting team and Procurement team only.
- **Type filter:** Visible in Line Item View only. Hidden and reset in Document View.
- **Export:** Exports the current filtered list to Excel.

## Process Flow

```mermaid
flowchart TD
    A([PO Status: Created]) --> B[User clicks + Record GR\non GR List page]
    B --> C[Search by PR number\nPO number or vendor]
    C --> D{Search result type}
    D -->|PR selected| E[PR card shown\nSelect a PO from the list]
    D -->|PO selected directly| F[Go to items table]
    E --> F
    F --> G[Enter Qty to Receive\nper line item]
    G --> H[Fill Receipt Details\nDate - Delivery Note - Attachment - Remarks]
    H --> I[Confirm Receipt]
    I --> J[GR Created — Confirmed]
    J --> K[Post accounting transaction\nto Bookkeeping]
    J --> L{Cancel GR?}
    L -->|Yes — enter reason| M[GR Reversal Created\nNegative qty and amount]
    M --> N[Post offsetting transaction\nto Bookkeeping]
    M --> O[Original GR tagged Reversed]
    L -->|No| P([GR remains Confirmed])

    style A fill:#27AE60,color:#fff
    style J fill:#27AE60,color:#fff
    style M fill:#E74C3C,color:#fff
    style P fill:#27AE60,color:#fff
```

## Enhancements

### Context
The current flow records GR inside the PO page with minimal fields. E1–E4 build the standalone GR module, reversal flow, Bookkeeping integration, and delivery note field — completing GR as a first-class module.

---

### E1 — Standalone GR List Page

**Status:** Pending

Replaces GR recording inside the PO page with a dedicated GR List page accessible from the main navigation. Provides Document View and Line Item View with filters, export, and the "+ Record GR" entry point.

**Acceptance Criteria:**

*Page access*
- GR List page is accessible from the main navigation by Procurement, Accounting, and Requesters.
- Requesters see only GRs from their own PRs. Procurement and Accounting see all GRs.
- Accounting can view GRs but cannot record or reverse them.

*Toolbar*
- Row 1: Search input (by GR number, PO number, vendor), Export button, Document View / Line Item View toggle, and "+ Record GR" button.
- Row 2: Date range filter, Status filter (All / Confirmed / Reversed / Reversal), Type filter (All / Asset / Expense), Reset button.
- Type filter is visible in Line Item View only — hidden in Document View.
- Export downloads the current filtered view as Excel.

*Record GR — 3-step modal*
- "+ Record GR" button opens the Record GR modal.
- Step 1 Search: user types to search by PR number, topic, requester, PO number, or vendor name. Dropdown appears only when at least one character is typed.
- Selecting a PR shows a PR card and a list of POs under that PR. User must select a PO to proceed.
- Selecting a PO directly skips the PR/PO list and goes straight to the items table.
- Step 2 Items to Receive: shows all PO line items with ordered qty, remaining qty, a qty to receive input, and a live GR progress bar. Fully received items (remaining qty = 0) are dimmed and input is disabled.
- Qty to receive cannot exceed remaining qty. Error shown inline; Confirm button disabled until resolved.
- Step 3 Receipt Details: Receipt Date (mandatory, defaults to today), Delivery Note No. (optional), Delivery Document (optional, PDF/JPG/PNG max 10 MB), Remarks (optional).
- Confirm Receipt button is disabled until at least one item has a valid qty > 0 and no validation errors exist.

*Document View* — default. One collapsed row per GR document:

| GR Date | GR Number | PO Number | Vendor | Items | Total Amount | Delivery Note | Status | Action |
|---|---|---|---|---|---|---|---|---|
| 2026-06-01 | GR-0001 | PO-0042 | ABC Co. | 3 items | 145,000 | DN-2026-0042 | Reversed | ✏ |
| 2026-06-03 | GR-0002 | PO-0038 | XYZ Co. | 1 item | 12,500 | — | Confirmed | ✏ |
| 2026-06-01 | GR-0001-R | PO-0042 | ABC Co. | 3 items | -145,000 | DN-2026-0042 | Reversal | ✏ |

Each row is expandable. Sub-row shows line items:

| Item Name | Type | Asset No. / GL Code | Qty Received | Unit Price | Amount |
|---|---|---|---|---|---|
| Laptop Dell | Asset | A-20240012 | 2 | 55,000 | 110,000 |
| HDMI Cable | Expense | 5210600010 ค่าซ่อมแซม | 5 | 500 | 2,500 |

Pencil icon (✏) on every collapsed row opens the GR Detail modal.

*Line Item View* — one flat row per line item across all GRs:

| GR Date | GR Number | PO Number | Vendor | Item Name | Type | Asset No. / GL Code | Qty Received | Unit Price | Amount | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| 2026-06-01 | GR-0001 | PO-0042 | ABC Co. | Laptop Dell | Asset | A-20240012 | 2 | 55,000 | 110,000 | Reversed |
| 2026-06-03 | GR-0002 | PO-0038 | XYZ Co. | Mouse | Expense | 5210500040 | 10 | 3,250 | 32,500 | Confirmed |
| 2026-06-01 | GR-0001-R | PO-0042 | ABC Co. | Laptop Dell | Asset | A-20240012 | -2 | 55,000 | -110,000 | Reversal |

Reversal rows appear in date order — not nested under the original. Reversal quantities and amounts shown in red.

---

### E2 — GR Reversal Flow

**Status:** Pending

Allows a Confirmed GR to be reversed when recorded incorrectly. Creates an independent Reversal document with negative quantities and amounts, and posts an offsetting accounting transaction.

**Acceptance Criteria:**
- "Cancel GR" button is shown in the GR Detail modal footer only for Confirmed GRs.
- "Cancel GR" is visible only to Requesters (own PRs) and Procurement team members. Accounting cannot reverse a GR.
- Clicking "Cancel GR" opens a Reversal confirmation modal showing the original GR details and the reversal document number.
- Cancellation reason is mandatory (minimum 5 characters). Confirm button is disabled until filled.
- GR Reversal is created with negative quantities and amounts. Reversal date is set to the original GR date — not editable.
- GR Reversal appears in the GR list as an independent document in date order, not nested under the original.
- Original GR is tagged Reversed. Reversal document is tagged Reversal.
- Reversed and Reversal documents cannot be cancelled again — "Cancel GR" is not shown on them.

---

### E3 — Bookkeeping Integration

**Status:** Pending

Every confirmed GR and GR Reversal posts an accounting transaction to Bookkeeping (S3 file upload). Failed posts are flagged for retry without blocking the GR record.

**Acceptance Criteria:**
- Every confirmed GR triggers an accounting transaction to Bookkeeping immediately after confirmation.
- Every GR Reversal triggers an offsetting transaction on the reversal date (= original GR date).
- If the Bookkeeping post fails, the GR record is still saved. The failed GR is flagged internally for retry.
- Transaction payload format to be defined with the Bookkeeping team (TBD).

---

## Decisions Log
- **GR is final** — ✅ Confirmed GRs cannot be edited. Corrections are handled by creating a GR Reversal document.
- **Reversal date = original GR date** — ✅ The Reversal is dated the same as the original GR so the offsetting accounting transaction posts to the correct period.
- **Reversal as independent document** — ✅ GR Reversal appears in the GR list in date order, not nested under the original, to keep the audit trail flat and clear.
- **Bookkeeping retry** — ✅ Failed posts are flagged for retry. Manual retry vs. automatic background retry: TBD with Bookkeeping team.

## Open Questions
- [ ] **Bookkeeping transaction payload:** Fields and format to be defined with the Bookkeeping team.
- [ ] **Bookkeeping retry:** Manual retry button or automatic background retry for failed sends?

## Related Features
- [PO Creation and Approval](../../02_features/PO-Purchase-Order/001-po-creation-and-approval.md)
- [Vendor Portal Billing](../../02_features/Billing/002-vendor-portal-billing.md)
