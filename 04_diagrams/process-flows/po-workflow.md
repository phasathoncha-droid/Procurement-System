# PO Workflow Diagram

## PO is created from a PR line item where Document Type = PO

```mermaid
flowchart TD
    A([PR: Waiting Create PO]) --> B[System auto-creates PO\nStatus: Draft]
    B --> C[Financial Controller\nReviews PO]
    C --> D{FC Decision}
    D -->|Reject| E[PO returned with comments]
    E --> B
    D -->|Approve| F{CFO Approval Required?\nBased on Amount Config}
    F -->|No - Below Threshold| G
    F -->|Yes - Above Threshold| H{CFO Decision}
    H -->|Reject| E
    H -->|Approve| G[PO Status: Created ✓]

    G --> I[PO is now active\nGR and Billing can be recorded]

    I --> J[[Good Receipt GR\nsee gr-workflow.md]]
    I --> K[[Billing\nsee billing-workflow.md]]

    style A fill:#F39C12,color:#fff
    style G fill:#27AE60,color:#fff
    style J fill:#8E44AD,color:#fff
    style K fill:#8E44AD,color:#fff
```

## PO States
| State | Description |
|---|---|
| Draft | PO auto-generated from approved PR line; awaiting submission for approval |
| Pending Approval | Submitted to Financial Controller (and CFO if above threshold) |
| Created | Fully approved — GR and Billing can now be recorded against this PO |

## Business Rules
- PO is auto-created from PR lines where document type = PO after Finance coding is complete
- CFO approval step is **configurable** — triggered only when PO amount exceeds the configured threshold
- Rejection at any step returns the PO to Draft with comments
- Once **Created**, GR and Billing are independent — they do not change the PO status
- PO status does **not** depend on GR or Billing completion
