# Procurement System — Project Scope Overview

## Objective
Build an internal procurement platform for Ngern Turbo that replaces SAP's PO module and eliminates manual paper/email-based processes.

## Modules & Delivery Status

| # | Module | Status | Notes |
|---|---|---|---|
| 1 | Purchase Request (PR) | Enhancement | Core flow built; enhancements in progress |
| 2 | Purchase Order (PO) | In Design | State: Draft → Pending Approval → Created |
| 3 | Good Receipt (GR) | In Design | Partial & full; integrates with Bookkeeping |
| 4 | Billing / Invoice | In Design | Partial & full; integrates with Bookkeeping |
| 5 | Vendor Management | Done | Vendor master, AVL, bank info — already built |
| 6 | Approval Workflow Engine | In Design | Configurable by amount, role, category |

## High-Level Process Flow

```
[Requester] → Open PR
      ↓
[Manager] → Approve PR
      ↓
[Procurement] → Review + Collect Vendor Quotes + Compare Pricing
      ↓  Assign document type per line item:
      │
      ├── Contract / Memo / Online Payment ───→ Document Created ✓ (flow ends here)
      │
      └── PO ──────────────────────────────────────────────────────────┐
                                                                        ↓
                                               [Procurement Manager] → Approve
                                                                        ↓
                                                           [CFO] → Approve
                                                                        ↓
                                             [Accountant] → Assign Cost Center
                                                            + GL (Expense) or Asset Number
                                                                        ↓
                                                       Status: Waiting Create PO
                                                                        ↓
                                               [Financial Controller] → Approve PO
                                                                        ↓
                                                [CFO] → Approve PO (if above threshold)
                                                                        ↓
                                                           PO Status: Created ✓
                                                                        ↓
                                              ┌─────────────────────────────────────┐
                                              │  Against the PO (partial or full):  │
                                              │                                     │
                                              │  [Warehouse] → Good Receipt (GR)    │
                                              │   → Send transaction to Bookkeeping │
                                              │                                     │
                                              │  [Procurement] → Create Billing     │
                                              │   → Send transaction to Bookkeeping │
                                              └─────────────────────────────────────┘
```

## Document Type Rules
| Type | Downstream Behavior |
|---|---|
| PO | Full PO approval flow → Created → enables GR and Billing |
| Contract | Document created directly — no PO state machine |
| Memo | Document created directly — no PO state machine |
| Online Payment | Document created directly — no PO state machine |

## GR & Billing Design Principles
- Both GR and Billing are **linked to a PO**
- **Partial** and **full** completion supported for each
- Multiple GR and Billing records can exist per PO
- Each confirmed GR and approved Billing sends an **accounting transaction to Bookkeeping**
- GR status and Billing status are **independent state machines** from each other and from PO

## Billing Approach
- Procurement team creates billing records internally
- Supplier self-service portal for vendor billing — Phase 2 (TBD)

## Integration Points
| System | Purpose |
|---|---|
| Bookkeeping (internal) | Receives accounting transactions from GR and Billing |
| HR / Active Directory | User roles and approval hierarchy |
| Email (SMTP) | Approval notifications |

## Out of Scope (Phase 1)
- Supplier self-service portal (Phase 2)
- Contract lifecycle management (CLM)
- Strategic sourcing / e-Auction

## Key Business Rules
- PR is **complete** only when all line items have a purchasing document
- PO CFO approval is **configurable** by amount threshold
- GR, Billing, and PO each have **independent** status flows
- Non-PO document types do not enter the PO workflow
