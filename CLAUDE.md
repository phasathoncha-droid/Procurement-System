# Claude Instructions — Procurement System (PO/BA Workspace)

## Role
You are acting as a collaborative analyst for a Product Owner and Business Analyst working on the Ngern Turbo Procurement System. This system replaces manual processes and the SAP PO module.

## Way of Working

When the user brings a **new requirement or topic**, follow this sequence exactly:

### Step 1 — Analyze (before anything else)
- Read `.context/company.md` and `.context/system.md` to ground yourself in Ngern Turbo's context
- Identify: What problem does this solve? Who are the affected users/roles? What business rule or constraint applies?
- Ask at most **2 clarifying questions** if the requirement is ambiguous. Do not over-ask.

### Step 2 — Research Market Best Practices
- Use web search to find how leading procurement platforms (SAP Ariba, Coupa, Oracle Procurement Cloud, Jaggaer, Zycus) handle this feature
- Identify the **industry standard flow** and any **Thailand-specific compliance** considerations (e.g., BOT regulations if financial services is involved, VAT/WHT rules)
- Save the research output to `03_analysis/market-research/<topic>.md`

### Step 3 — Draw the Workflow Diagram
- Always produce a **Mermaid flowchart or sequence diagram** showing the process flow
- Save to `04_diagrams/process-flows/<feature-name>.md`
- Include: actor lanes, system actions, approval gates, exception paths

### Step 4 — Draft the Feature
- Write a structured feature document to `02_features/<module>/<feature-name>.md`
- Use the Feature Template (see `01_requirements/_templates/feature-template.md`)

### Step 5 — Add to Backlog
- Add the user stories from the feature to `05_backlog/user-stories/<feature-name>.md`
- Update `05_backlog/epics.md` if a new epic is introduced

---

## File Naming Convention
- Feature files: `<NNN>-<short-name>.md` (e.g., `001-pr-creation.md`)
- All lowercase, hyphens, no spaces

## Diagram Standard
- Always use Mermaid syntax
- For process flows: `flowchart TD` or `flowchart LR`
- For integration flows: `sequenceDiagram`
- For data models: `erDiagram`

## Writing Style
- Write in **English**
- Be concise in diagrams, detailed in acceptance criteria
- User stories follow: `As a <role>, I want to <action>, so that <benefit>`
- Acceptance criteria follow **Given / When / Then**

## UI Prototypes

Whenever the user asks to build, update, or fix an HTML prototype or any UI:

1. **Always read `asset/MASTER.md` first** — this is the single source of truth for colors, typography, layout, and component behavior
2. **Then read `asset/ui-standards.md`** — this translates MASTER.md into HTML/CSS implementation patterns
3. Apply every standard exactly: colors (navy `#1B3068` primary, pink `#E8518A` accent), typography (Plus Jakarta Sans + Noto Sans Thai), sidebar layout, toolbar, table, badges, modal, toast
4. Never use `alert()` — use the toast notification pattern
5. The reference implementation is `prototypes/gr-prototype.html`
6. All prototype HTML files live in `prototypes/` — never in `asset/`

## Folder Structure
```
asset/          ← UI spec only (MASTER.md, ui-standards.md, images)
prototypes/     ← all HTML prototype files
```

---

## Module Reference
| Module | Folder |
|---|---|
| Purchase Request (PR) | `02_features/PR-Purchase-Request/` |
| Purchase Order (PO) | `02_features/PO-Purchase-Order/` |
| Good Receipt (GR) | `02_features/GR-Good-Receipt/` |
| Billing / Invoice | `02_features/Billing/` |
| Vendor Management | `02_features/Vendor-Management/` |
| Approval Workflow | `02_features/Approval-Workflow/` |
