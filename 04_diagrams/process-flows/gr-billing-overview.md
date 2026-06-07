# GR & Billing Overview Diagram

## Triggered after PO Status = Created

### AS-IS (Current State)
- **GR:** Recorded inside PO page by Requester or Procurement. Fields: quantity, GR date, remark, attachment. No cancellation mechanism. No Bookkeeping posting.
- **Billing:** Recorded inside PO page by Procurement. Fields: invoice date, amount, attachment, remark. No invoice number. Accounting has no visibility in the system — all communication is manual via email. No Bookkeeping posting.

### To-Be (Target State — diagram below)

```mermaid
flowchart TD
    A([PO Status: Created])

    A --> C[GR Flow — Partial or Full]
    A --> D[Billing Flow — Partial or Full]

    C --> C1[Requester or Procurement\nrecords Goods / Services Received]
    C1 --> C2[Specify quantity received\nagainst PO line items\nGR date · Delivery Note · Attachment · Remark]
    C2 --> C3{Full Receipt?}
    C3 -->|Partial| C4[GR Status: Partial\nPO remains open for more GR]
    C3 -->|Full| C5[GR Status: Complete]
    C4 --> C6[Generate accounting file\nUpload to S3 → Bookkeeping reads]
    C5 --> C6

    C6 --> CX{Cancel GR?}
    CX -->|Cancel with reason| CY[GR Reversal created\nnegative qty and amount\nUpload offsetting file to S3]

    D --> D1[Procurement creates Billing record\nInvoice No · Date · Amount · Attachment]
    D1 --> D2[Waiting Accounting Confirmation]
    D2 --> D3{Accounting Decision}
    D3 -->|Return — Internal only| D4[Back to Procurement\nEdit & Resubmit]
    D4 --> D2
    D3 -->|Reject| D5([Rejected — dead end\nmandatory comment])
    D3 -->|Confirm| D6[Confirmed\nGenerate AP file · Upload to S3]
    D6 --> D7{Cancel needed?}
    D7 -->|Cancel Invoice + reason| D8[Reversed + Reversal created\nUpload offsetting file to S3]

    C6 --> E[(S3\nBookkeeping reads\nand records)]
    CY --> E
    D6 --> E
    D8 --> E

    style A fill:#27AE60,color:#fff
    style E fill:#2C3E50,color:#fff
    style D5 fill:#EF4444,color:#fff
    style D8 fill:#94A3B8,color:#fff
    style CY fill:#94A3B8,color:#fff
```

## Key Design Points
- GR and Billing are **both linked to the same PO**
- Each can be done multiple times (partial) until fully completed
- GR and Billing are **independent** — billing does not require GR to be complete
- **GR actor is Requester or Procurement** — not Warehouse
- **GR posts to Bookkeeping via S3 file** — no Accounting confirmation step required
- **Billing requires Accounting confirmation** before posting to Bookkeeping via S3 file
- Both GR and Billing reversals generate offsetting S3 files to zero out the original entry
- S3 file format and payload: **TBD with Bookkeeping team**

## Billing Statuses (Queue)
| Status | Description |
|---|---|
| Waiting Accounting Confirmation | Active — awaiting Accounting action |
| Rejected | Permanent dead end — no resubmission |

## Billing Statuses (Billing List)
| Status | Description |
|---|---|
| Confirmed | AP file posted to Bookkeeping via S3 |
| Reversed | Original invoice — cancelled, offset by Reversal record |
| Reversal | Counter-entry that zeroes out the Reversed AP entry |

## GR Statuses
| Entity | Possible States |
|---|---|
| GR Record | Confirmed / Reversed / Reversal |
| PO GR Status | Not Started → Partial → Complete |
| PO Billing Status | Not Started → Partial → Complete |
