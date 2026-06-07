# Procurement System — Workspace

> **Start here every session.** Read this file first, then go to the linked files for detail.

---

## What Is This Project?
Ngern Turbo is replacing SAP's PO module and manual procurement processes with an internal procurement platform. The system covers the full lifecycle: Purchase Request → Purchase Order → Good Receipt → Billing → Bookkeeping.

Full context: [`.context/system.md`](.context/system.md) · [`.context/company.md`](.context/company.md)

---

## Where We Left Off
*(Last updated: 2026-06-04)*

- PR module is already built — currently in **enhancement phase**
- PO, GR, and Billing feature specs have been **drafted** — pending review and correction
- Vendor Management is already built — not in current scope
- Contract / Memo / Online Payment downstream flows — **not yet designed**
- Bookkeeping transaction payload — **TBD with Bookkeeping team**

---

## Module Status

| Module | Status | Feature File |
|---|---|---|
| Purchase Request (PR) | Done + Enhancement planned | [001-pr-creation-and-approval.md](02_features/PR-Purchase-Request/001-pr-creation-and-approval.md) |
| Purchase Order (PO) | Draft — pending review | [001-po-creation-and-approval.md](02_features/PO-Purchase-Order/001-po-creation-and-approval.md) |
| Good Receipt (GR) | Draft — pending review | [001-gr-recording.md](02_features/GR-Good-Receipt/001-gr-recording.md) |
| Billing | Draft — pending review | [001-billing-creation.md](02_features/Billing/001-billing-creation.md) |
| Vendor Management | Done | — |
| Approval Workflow Engine | In design | — |

---

## What to Work on Next

- [ ] Review and approve the 3 drafted feature files (PO, GR, Billing)
- [ ] Design the Contract / Memo / Online Payment flow
- [ ] Define Bookkeeping transaction payload with the Bookkeeping team
- [ ] Define cost center integration — procurement system reads cost center master data from Bookkeeping (API or sync approach TBD)
- [ ] Design GL code master list config screen — Accounting team needs a UI to manage GL codes
- [ ] Write PR enhancement requirements
- [ ] Design Approval Workflow Engine (configurable rules, delegation, escalation)

---

## How We Work Together
When you bring a new requirement, I will:
1. Read `.context/` to ground myself in Ngern Turbo's context
2. Analyze the requirement and ask at most 2 questions
3. Research market best practice → saved to `03_analysis/market-research/`
4. Draw the workflow diagram → saved to `04_diagrams/process-flows/`
5. Write the feature spec → saved to `02_features/<module>/`

Full instructions: [`CLAUDE.md`](CLAUDE.md)

---

## Folder Guide

| Folder | Purpose |
|---|---|
| `.context/` | Company and system context — always read before analyzing |
| `00_overview/` | Big picture scope and end-to-end process flow |
| `01_requirements/_templates/` | Feature writing template |
| `02_features/` | Feature specs per module — what goes to developers |
| `03_analysis/` | Market research and gap analysis per topic |
| `04_diagrams/` | Mermaid process flow and data model diagrams |
| `05_backlog/` | Epics and user stories for task tracker |
| `06_decisions/` | Key decisions and the reasoning behind them |
