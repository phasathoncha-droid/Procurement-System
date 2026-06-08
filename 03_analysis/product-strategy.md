# Product Strategy — Ngern Turbo Procurement System

**Date:** 2026-06-07
**Goal:** Replace SAP PO module and manual procurement processes with a purpose-built internal platform.

---

## North Star

> Every procurement transaction — from request to payment recording — lives in one system. SAP is completely off the critical path.

---

## Out of Scope (This System)

- WHT (Withholding Tax) generation
- e-Tax invoice integration
- Payment execution
- Asset number generation (remains in SAP until further decision)

---

## Current State Inventory

| Module | What Exists Today | Gap |
|---|---|---|
| PR | Full flow built: creation, approval chain, price comparison (vendor free-text or select from master UI only), finance coding (cost center + GL as free-text copy-paste, asset number as free text) | Entity field missing; finance coding not master-driven (smart search TBD); vendor linkage enforcement (E2/E3/E4) not built |
| PO | Creates PO (SAP number entered manually → instant Created), Contract, Memo, Online Payment — items and amounts pre-filled from PR | No in-system approval; PO number comes from SAP; no cancellation flow |
| GR | Inside PO page: quantity received, GR date, remark, attachment, partial receipt | No delivery note field; no reversal/reversal types; no Bookkeeping posting; not a standalone module |
| Billing | Inside PO page: invoice date, amount, attachment, remark, partial billing | No invoice number field; no Accounting confirmation flow (Accounting is outside system); no reversal; no Bookkeeping posting; not a standalone module |
| Vendor Management | Full scope, built | — |
| Bookkeeping Integration | Not started | S3 upload pattern agreed; file format TBD with Bookkeeping team |

---

## Integration Pattern — Bookkeeping

The procurement system generates a structured file per accounting event and uploads it to S3. The Bookkeeping system reads from S3 and records the transaction.

**Triggers:**
- GR confirmed → generate GR accounting file → upload to S3
- GR reversed → generate reversal file → upload to S3
- Billing confirmed → generate AP file → upload to S3
- Billing reversed → generate reversal file → upload to S3

**File format:** TBD with Bookkeeping team.

The **legal entity** (NTB, NTBX, NTBI, NTBPL) selected on the PR is the entity under which Bookkeeping records the transaction.

---

## What SAP Is Still Doing Today

| SAP Role | Replacement Plan |
|---|---|
| PO approval | Build approval process in this system (Phase 3) |
| PO number generation | System generates own PO number — format: `PO-{BE year}/{sequence}` e.g. `PO-2569/00001` (Phase 3) |
| Accounting recording (GR + Billing) | S3 file upload → Bookkeeping reads and records (Phase 2) |
| Asset number generation | **Remains in SAP** — accountant enters asset number from SAP as free text |

**SAP parallel run:** The system continues to accept a manually entered SAP PO number through Phase 1 and Phase 2. The switch to internal PO number generation is a deliberate cutover decision in Phase 3 — not forced. Both systems run in parallel until confidence is established.

---

## Roadmap

### Phase 1 — Make the System Reliable *(Highest Priority)*

**Goal:** Ensure PR data is accurate, trustworthy, and structurally ready to connect to Bookkeeping. No SAP changes yet — this is purely about fixing what exists.

#### 1A — PR Corrections (Foundation)

These are corrections to the existing PR, not new features. Required before any accounting integration can be built.

- [ ] **Entity field on PR creation** — Requester selects legal entity (NTB / NTBX / NTBI / NTBPL). This entity is passed to Bookkeeping in all accounting files generated from this PR.
- [ ] **Finance Coding — master-driven** — Cost center and Expense GL sourced from Bookkeeping master data via smart search (API or sync TBD). Asset number remains free text (from SAP). Current free-text finance coding must be replaced.

#### 1B — PR Vendor Linkage (Enhancement)

Enforces that the vendor approved at Price Comparison is the vendor on the final document. No substitution possible after CFO approval.

- [ ] **E1 — Two vendor input modes at Price Comparison** — Search & Select (vendor in master → locks vendor code + tax ID immediately) or Manual Entry (vendor not yet in master → records name + tax ID as unconfirmed)
- [ ] **E2 — Vendor identity frozen after CFO approval** — winning vendor per line item cannot change after CFO approves
- [ ] **E3 — Vendor match check at Waiting Create PO (manual entry path only)** — Procurement searches vendor master by tax ID; match found → proceed; no match → register vendor first
- [ ] **E4 — Vendor field on document is read-only** — pre-populated from confirmed vendor code, not editable at document creation

**Exit criterion for Phase 1:** All PRs carry a valid entity tag and master-driven finance codes. Vendor on every document is traceable back to the approved vendor at Price Comparison.

---

### Phase 2 — Complete the Accounting Pipeline

**Goal:** GR and Billing are standalone modules that post reliably to Bookkeeping via S3. This proves the accounting backbone works before SAP is removed from the PO step. SAP still generates the PO number during this phase.

#### 2A — GR as Standalone Module

- [ ] Extract GR from PO page into its own module
- [ ] Add transaction types: **Confirmed**, **Reversed**, **Reversal** — to support double-entry accounting recording
- [ ] Add fields: Delivery Note number, Delivery Document attachment
- [ ] GR confirmation triggers S3 file upload to Bookkeeping
- [ ] GR reversal triggers offsetting S3 file upload to Bookkeeping
- [ ] Partial receipt tracking per line item
- [ ] GR list page with Document View and Line Item View

#### 2B — Billing as Standalone Module

- [ ] Extract Billing from PO page into its own module
- [ ] Add field: Invoice Number (required, unique per vendor per Thai Revenue Dept.)
- [ ] Full flow: Procurement creates → Accounting confirms → Bookkeeping file posted
- [ ] Return flow: Accounting returns to Procurement with comment → Procurement edits and resubmits
- [ ] Reject flow: permanent dead end, mandatory comment
- [ ] Reversal support: Confirmed billing can be cancelled → creates Reversal counter-record with negative amounts → offsetting file posted to Bookkeeping
- [ ] Partial billing: multiple invoices per PO, progress bar per line

#### 2C — Bookkeeping Integration (S3) *(starts after 2A and 2B are complete)*

- [ ] Define file format and schema with Bookkeeping team *(can be done in parallel with 2A/2B as a design activity — implementation starts after modules exist)*
- [ ] Implement S3 upload on GR confirmation, GR reversal, Billing confirmation, Billing reversal
- [ ] Failed upload handling — record saved, flagged for retry
- [ ] Retry mechanism (manual button or background auto-retry — TBD)

**Exit criterion for Phase 2:** A full cycle — PR → PO (SAP number, as today) → GR → Billing → file posted to Bookkeeping — runs end-to-end with no manual handoff. Accounting records are complete and auditable without SAP.

---

### Phase 3 — Advanced Reporting and Credit Note *(TBC)*

**Goal:** Give finance and management full visibility into procurement spend, and handle vendor credit notes as a formal document type within the billing flow.

#### 3A — Advanced Report

- [ ] **Procurement spend report** — aggregated view by vendor, department, entity, and period
- [ ] **PO status report** — open POs, partial receipts, pending billings
- [ ] **GR aging report** — time between PO creation and GR confirmation
- [ ] **Billing aging report** — time between GR and invoice confirmation
- [ ] **Export** — export to Excel/CSV; filters by date range, entity, department, vendor, status
- [ ] **Role-based access** — Procurement Manager, CFO, and Accounting can access; Procurement and Requester have limited scope

#### 3B — Credit Note

- [ ] **Credit Note as document type** — vendor issues a credit note against a confirmed billing; Accounting creates a Credit Note record linked to the original invoice
- [ ] **Negative amount posting** — Credit Note generates a negative AP adjustment entry uploaded to Bookkeeping via S3
- [ ] **Partial credit support** — credit note may cover one or more line items, not necessarily the full invoice amount
- [ ] **Status flow** — Credit Note follows its own status: Waiting Accounting Confirmation → Confirmed / Rejected
- [ ] **Linked on Billing List** — Credit Note record visible on the Billing List page, linked to the original invoice

**Exit criterion for Phase 3:** Finance team can pull a full spend report for any period without exporting from multiple screens. Credit notes are recorded and posted to Bookkeeping without manual workarounds.

---

### Phase 4 — Remove SAP from the PO Step

**Goal:** Generate and approve POs entirely inside this system. This is the SAP retirement step — executed only after Phase 2 proves the accounting backbone is reliable.

#### 4A — PO Approval Process

- [ ] **PO number auto-generation** — format `PO-{BE year}/{5-digit sequence}`, assigned at Draft, never changes. Replaces manual SAP PO number entry.
- [ ] **Approval chain** — Financial Controller approves; CFO approves only when PO total exceeds configurable threshold
- [ ] **Rejection handling** — PO returns to Draft with reviewer comments; Procurement edits and resubmits
- [ ] **CFO threshold config** — CFO sets the amount threshold; applies to new POs only
- [ ] **PO cancellation flow** — Procurement initiates with mandatory reason; requires approval chain; blocked if GR or confirmed Billing exists

**SAP cutover:** Once 4A is live and validated, the PO number field switches from manual SAP entry to auto-generated. This is a deliberate, coordinated cutover — not automatic.

**Exit criterion for Phase 4:** First PO is created, approved, and issued inside this system with no SAP step. SAP is no longer on the critical path for procurement.

---

### Phase 5 — Vendor Self-Service *(Future)*

**Goal:** Reduce Procurement workload, give vendors visibility into their own transactions.

- [ ] **Vendor Portal — Billing (Path B):** Vendors submit invoices directly through the portal. Procurement reviews before forwarding to Accounting.
- [ ] **Vendor self-registration:** Vendors register themselves via portal. Procurement/Admin approves. Replaces internal-only vendor registration.

---

## Key Dependencies and Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Bookkeeping file format TBD | Blocks Phase 2 entirely | Start Bookkeeping team conversation immediately — this is the longest lead-time item |
| Finance Coding master data source | Phase 1B blocked until Bookkeeping API/sync is defined | Agree on integration approach (real-time API call vs. periodic sync) early |
| Asset number still in SAP | Minor — accountant manually enters free text | Accept as known limitation; revisit when SAP retirement is broader |
| Entity field missing from current PR | All Bookkeeping records will lack entity tag | Fix in Phase 1A before any accounting recording is built |
| Accounting pipeline not proven before SAP cutover | Risk of financial recording gaps if Phase 4 ships before Phase 2 is stable | Phase sequencing enforces this: Phase 2 exit criterion must be met before Phase 4 begins |

---

## Open Questions

- [ ] How does the system pull cost center and GL master data from Bookkeeping — real-time API call or periodic sync?
- [ ] S3 file format and schema — to be defined with Bookkeeping team
- [ ] Bookkeeping retry mechanism — manual button or automatic background retry?
- [ ] Contract / Memo / Online Payment downstream flows — any accounting recording required, or purely document tracking?
- [ ] When Phase 3 PO approval goes live, how are in-flight POs handled — do existing SAP-numbered POs migrate, or does the cutover apply to new POs only?
