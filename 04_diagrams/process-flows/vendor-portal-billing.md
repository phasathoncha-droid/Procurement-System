# Diagram: Vendor Portal Billing Flow (Phase 2)

```mermaid
flowchart TD
    A([Procurement assigns vendor email\nin Vendor Management]) --> B[System sends invite email\nto vendor]
    B --> C[Vendor registers via invite link\nsets password]
    C --> D[Vendor logs in to Portal]

    D --> E[View PO List]
    E --> F[Select PO — click Submit Invoice]
    F --> G[Fill billing form\nInvoice No · Date · Items · Net Amount\nVAT auto-calculated · Attach invoice PDF]
    G --> H[Vendor submits\nStatus: Waiting Procurement Review]

    H --> I{Procurement Review}
    I -->|Reject — mandatory comment| RJ([Rejected — dead end\nVendor submits new invoice])
    I -->|Forward to Accounting| J[Status: Waiting Accounting Confirmation]

    J --> K{Accounting Decision}
    K -->|Reject — mandatory comment| RJ
    K -->|Confirm| M[Status: Confirmed\nPost AP to Bookkeeping]
    M --> N([Vendor sees Confirmed status])

    style A fill:#4A90D9,color:#fff
    style N fill:#27AE60,color:#fff
    style RJ fill:#EF4444,color:#fff
```

## Notes
- **Return to vendor is not supported in Phase 1.** Incorrect submissions are rejected — vendor submits a new invoice.
- Vendor-facing statuses: Waiting Review · Waiting Confirmation · Confirmed · Rejected
- Internal statuses mirror Phase 1 billing: Waiting Procurement Review → Waiting Accounting Confirmation → Confirmed / Rejected
