# PR Workflow Diagram

## Status: Enhancement Phase (Core built, refining)

```mermaid
flowchart TD
    A([Requester]) --> B[Open PR Ticket\nItem · Qty · Est. Price · Reason]
    B --> C{Manager\nApproval}
    C -->|Reject| B
    C -->|Approve| D[Procurement Team Reviews PR]
    D --> E[Collect Vendor Quotations\n& Compare Pricing]

    E --> EV{Winning vendor\nin master?}
    EV -->|Yes — Search & Select| EV1[Vendor code + Tax ID locked]
    EV -->|No — Manual Entry| EV2[Name + Tax ID recorded\nas unconfirmed]
    EV1 --> F[Assign Document Type per Line Item]
    EV2 --> F

    F --> G1[Type: Contract]
    F --> G2[Type: Memo]
    F --> G3[Type: Online Payment]
    F --> G4[Type: PO]

    G1 --> I{Procurement Manager\nApproval}
    G2 --> I
    G3 --> I
    G4 --> I

    I -->|Reject| E
    I -->|Approve| J{CFO Approval}
    J -->|Reject| E
    J -->|Approve| J1[[Vendor identity\nfrozen per line item]]

    J1 --> K[Accountant: Assign Cost Center\n+ GL or Asset Number]
    K --> O[Status: Waiting Create PO]

    O --> VQ{Vendor confirmed\nfrom master?}
    VQ -->|Yes — selected at Price Comparison| DOC[Create Document]
    VQ -->|No — manually entered| VS[Search vendor master\nby Tax ID]
    VS --> VM{Match found?}
    VM -->|Yes| DOC
    VM -->|No| VR[Register vendor in\nVendor Management]
    VR --> VS

    DOC --> D1([PO Created ✓])
    DOC --> D2([Contract Created ✓])
    DOC --> D3([Memo Created ✓])
    DOC --> D4([Online Payment Created ✓])

    style A fill:#4A90D9,color:#fff
    style J1 fill:#E8518A,color:#fff
    style D1 fill:#27AE60,color:#fff
    style D2 fill:#27AE60,color:#fff
    style D3 fill:#27AE60,color:#fff
    style D4 fill:#27AE60,color:#fff
    style VR fill:#F39C12,color:#fff
```

## PR States
| State | Description |
|---|---|
| Draft | Requester is filling in the PR |
| Pending Manager Approval | Submitted, awaiting manager |
| Pending Procurement Review | Manager approved, procurement reviewing |
| Pending Procurement Manager Approval | Quotes compared, awaiting Procurement Manager |
| Pending CFO Approval | Awaiting CFO sign-off |
| Pending Finance Coding | Awaiting Accountant cost center / GL / asset assignment |
| Waiting Create PO | Finance coding complete — PO lines ready to process |
| Complete | All line items have a purchasing document |
| Rejected | Rejected at any approval step |
| Cancelled | Cancelled by requester before completion |
