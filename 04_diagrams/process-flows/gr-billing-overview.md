# GR & Billing Overview Diagram

## Triggered after PO Status = Created

```mermaid
flowchart TD
    A([PO: Created]) --> B[PO is active]

    B --> C[GR Flow\nPartial or Full]
    B --> D[Billing Flow\nPartial or Full]

    C --> C1[Warehouse records\nGoods/Services Received]
    C1 --> C2[Specify quantity received\nagainst PO line items]
    C2 --> C3{Full Receipt?}
    C3 -->|No - Partial| C4[GR recorded as Partial\nPO remains open for more GR]
    C3 -->|Yes - Full| C5[GR recorded as Complete]
    C4 --> C6[Send accounting entry\nto Bookkeeping]
    C5 --> C6

    D --> D1[Procurement creates\nBilling record]
    D1 --> D2[Link to PO\nEnter invoice details]
    D2 --> D3{Full or Partial?}
    D3 -->|Partial| D4[Billing recorded as Partial\nPO open for more Billing]
    D3 -->|Full| D5[Billing recorded as Complete]
    D4 --> D6[Send accounting entry\nto Bookkeeping]
    D5 --> D6

    C6 --> E[(Bookkeeping\nInternal Accounting)]
    D6 --> E

    style A fill:#27AE60,color:#fff
    style E fill:#2C3E50,color:#fff
    style C6 fill:#E74C3C,color:#fff
    style D6 fill:#E74C3C,color:#fff
```

## Key Design Points
- GR and Billing are **both linked to the same PO**
- Each can be done multiple times (partial) until fully completed
- GR and Billing are **independent** — you can bill without GR and vice versa (exact rules TBD)
- Every GR and Billing action triggers an **accounting transaction to Bookkeeping**
- Transaction payload structure: **TBD with Bookkeeping team**

## States (Draft — to be detailed in feature spec)
| Entity | Possible States |
|---|---|
| GR | Pending → Partial / Complete |
| Billing | Pending → Partial / Complete |
