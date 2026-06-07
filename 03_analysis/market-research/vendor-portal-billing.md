# Market Research — Vendor Portal Billing

## Platforms Researched
Coupa (CSP), SAP Ariba Network, Oracle iSupplier, Jaggaer

---

## Vendor Onboarding
- Leading platforms use email-based invitation; vendor registers using the invited email
- Role-based access: admin, submitter, viewer inside the portal

**Applied to Ngern Turbo:** Email is already stored in Vendor Management. Vendor registers on the portal using that email — no separate invitation flow needed.

---

## Document Visibility
- Vendors see only documents assigned to them
- Documents show key info: reference number, amount, status, remaining balance
- Vendors can track all submitted billing in a status table (pending / approved / rejected)

---

## Vendor Billing Flow (Industry Standard)
```
Draft → Submit → Buyer Review → Finance Approve → Payment
```
**Applied to Ngern Turbo:** Vendor submits → Procurement reviews → Accounting confirms → Bookkeeping transaction sent.

---

## One Bill per Document (Our Rule)
- Industry default allows multiple invoices per PO (partial/progressive billing)
- Buyer can enforce one-invoice-per-document as a business rule
- System tracks cumulative billed amount to prevent over-billing

**Applied to Ngern Turbo:** One bill per document. Partial amount is allowed but a second bill cannot be created for the same document. Remaining balance is visible to vendor.

---

## Thailand-Specific
- **e-Tax Invoice:** Required by Revenue Department — seller/buyer tax ID, VAT 7% itemized. Phase 2.
- **WHT:** Handled by Accounting during confirmation step.
