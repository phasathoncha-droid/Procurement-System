# System Context — Ngern Turbo Procurement System

## Purpose
Replace manual procurement processes (email, spreadsheets) and the SAP PO module with a purpose-built internal procurement platform.

## Current Status
- **PR module:** Already built. Focus is on enhancements and integration.
- **PO module:** Designing the state machine and approval flow now.
- **GR module:** To be designed. Will integrate with Bookkeeping.
- **Billing / Invoice module:** To be designed. Will integrate with Bookkeeping.
- **Bookkeeping:** Internal accounting service at Ngern Turbo. Procurement will send accounting transactions to it upon GR confirmation and invoice approval. (Detail TBD.)

---

## PR Lifecycle (Implemented — Enhancement Phase)

### Phase 1 — Request & Approval
1. **Requester** opens a PR ticket
2. **Requester's Manager** approves
3. **Procurement Team** reviews
4. **Procurement** collects vendor quotations, compares pricing
   - Each line is assigned a **document type**:
     | Type | Document | PO Flow? |
     |---|---|---|
     | 1 | PO | Yes — follows PO state machine |
     | 2 | Contract | No |
     | 3 | Memo | No |
     | 4 | Online Payment | No |
5. **Procurement Manager** approves
6. **CFO** approves

### Phase 2 — Finance Coding
7. **Accountant** inserts cost center
8. **Accountant** classifies item:
   - Asset → assigns Asset Number
   - Expense → assigns GL Account
9. PR status → **Waiting Create PO**

### Phase 3 — PO Creation (AS-IS → TO-BE)
- **AS-IS:** Accountant enters PO number from SAP manually
- **TO-BE:** System creates PO directly — SAP removed from this step
- PR = **Complete** when all line items have a purchasing document

---

## PO State Machine

```
Draft → Pending Approval → Created
```

- **Draft:** PO is being prepared (auto-created from approved PR line items with type = PO)
- **Pending Approval:** PO submitted for approval chain:
  1. Financial Controller reviews
  2. CFO reviews *(configurable by amount threshold)*
- **Created:** PO approved, dispatched to vendor

> GR status and Billing status are **separate state machines** — independent of PO status.

---

## GR & Billing Integration with Bookkeeping

- Upon **GR confirmation** → procurement sends accounting transaction to Bookkeeping
- Upon **Invoice/Billing approval** → procurement sends accounting transaction to Bookkeeping
- **Bookkeeping** = internal Ngern Turbo accounting service
- Transaction payload details: **TBD**

---

## Key Actors

| Actor | Role |
|---|---|
| Requester | Opens PR |
| Requester's Manager | First-level PR approval |
| Procurement Team | Reviews PR, collects quotations |
| Procurement Manager | Approves vendor selection |
| CFO | Approves PR and optionally PO (config-based) |
| Accountant | Cost center, asset no., GL coding |
| Financial Controller | First PO approver |
| Warehouse / Receiver | Confirms GR |
| Finance / AP | Processes invoice, payment |
| Vendor | Receives PO, delivers, submits invoice |

## Compliance (Thailand)
- WHT (Withholding Tax) certificate generation required
- VAT 7% on vendor invoices
- e-Tax invoice integration — Phase 2

## TBDs
- CFO PO approval threshold: configurable by admin
- Bookkeeping transaction payload structure
- ERP/finance handoff for payment execution
- Contract / Memo / Online Payment downstream flows
