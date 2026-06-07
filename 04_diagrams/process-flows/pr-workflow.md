# PR Workflow Diagram

## Status: Enhancement Phase (Core built, refining)

```mermaid
flowchart TD
    A([Requester]) --> B[Open PR Ticket\nItem · Qty · Est. Price · Reason]
    B --> C{Manager\nApproval}
    C -->|Reject| B
    C -->|Approve| D[Procurement Team Reviews PR]
    D --> E[Collect Vendor Quotations\n& Compare Pricing]
    E --> F[Assign Document Type per Line Item]

    F --> G1[Type: Contract]
    F --> G2[Type: Memo]
    F --> G3[Type: Online Payment]
    F --> G4[Type: PO]

    G1 --> H1([Contract Created ✓])
    G2 --> H2([Memo Created ✓])
    G3 --> H3([Online Payment Created ✓])

    G4 --> I{Procurement Manager\nApproval}
    I -->|Reject| E
    I -->|Approve| J{CFO Approval}
    J -->|Reject| E
    J -->|Approve| K[Accountant: Assign Cost Center]

    K --> L{Item Type?}
    L -->|Asset| M[Assign Asset Number]
    L -->|Expense| N[Assign GL Account]
    M --> O[Status: Waiting Create PO]
    N --> O

    O --> P[→ Proceed to PO Workflow]

    style A fill:#4A90D9,color:#fff
    style H1 fill:#27AE60,color:#fff
    style H2 fill:#27AE60,color:#fff
    style H3 fill:#27AE60,color:#fff
    style P fill:#F39C12,color:#fff
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
