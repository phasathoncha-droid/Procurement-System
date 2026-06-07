# Vault Procurement — UI Design Template

> Single source of truth for color, typography, component theming, and layout.
> Product type: **Enterprise B2B procurement / approval workflow** (data-dense admin app, Thai-first, desktop web).
> Stack: React 19 + Vite + TailwindCSS + lucide-react. Tokens live in `src/styles/tokens.css` (theme `data-theme="turbo"`).

---

## 1. Design Direction

| Aspect | Decision | Reason |
|--------|----------|--------|
| Style | **Professional flat + soft elevation** | Procurement is trust/accuracy-driven; avoid playful or skeuomorphic. Subtle shadows convey hierarchy without noise. |
| Density | **Data-dense** | Tables, Kanban, multi-step forms. Compact spacing, tabular figures for money. |
| Tone | Calm, corporate, confident | Deep navy primary + restrained pink accent = institutional but not generic blue-SaaS. |
| Mode | **Light only** (current) | Internal tool used in office lighting. Dark mode = future (tokens already token-driven, so it is feasible). |
| Icons | **lucide-react**, stroke 1.5–2px, size 20 (nav) / 16 (inline) | One family, no emoji. Already centralized in `src/components/icons/index.ts`. |

---

## 2. Color Palette

### 2.1 Brand (Primary & Secondary)

| Token | Hex | Role | Where |
|-------|-----|------|-------|
| `--primary-color` | `#1B3068` | **PRIMARY** — deep navy | Sidebar logo bar, active nav, primary buttons, links, key headings |
| `--primary-dark` | `#142451` | Primary pressed/hover-dark | Button `:active`, gradients |
| `--primary-light` | `#2A4A99` | Primary tint | Hover on primary surfaces, secondary emphasis |
| `--accent-color` | `#E8518A` | **SECONDARY / ACCENT** — Turbo pink | Header underline bar, NTB tag, accent CTA (`.btn-accent`), highlight strip |
| `--accent-dark` | `#C93D72` | Accent pressed/hover | `.btn-accent:hover` |
| `--accent-cyan` | `#4FC3F7` | Tertiary accent | Sparingly: info chips, chart series, decorative |

**Primary = navy `#1B3068`** → structure, navigation, trust.
**Secondary = pink `#E8518A`** → brand identity, draws the eye to one focal action/marker per screen. Never use pink for body text or large fills.

### 2.2 Semantic

| Token | Hex | Meaning |
|-------|-----|---------|
| `--green-accent` | `#10B981` | Success / approved / active |
| `--yellow-accent` | `#F59E0B` | Warning / pending / needs attention |
| `--red-accent` / `--danger-color` | `#EF4444` | Error / rejected / destructive |
| `--danger-dark` | `#DC2626` | Destructive button hover |

Functional color is **never the only signal** — always pair with icon + text label (badges already do this).

### 2.3 Neutrals (text, surface, border)

| Token | Hex | Use |
|-------|-----|-----|
| `--text-primary` | `#0F1F42` | Headings, body, table cells |
| `--text-secondary` | `#4B5E85` | Labels, helper text, meta |
| `--text-tertiary` | `#8799B8` | Placeholder, disabled, timestamps |
| `--text-on-primary` | `#FFFFFF` | Text on navy/pink fills |
| `--background-light` | `#F0F4F9` | App canvas (body) |
| `--background-color-primary` | `#FFFFFF` | Cards, sidebar, header, modals |
| `--background-color-secondary` | `#F8FAFC` | Table header, subtle panels, Kanban column |
| `--border-color` / `--gray-border` | `#CBD5E1` | Dividers, input borders |
| `--border-light` | `#EEF2F7` | Inner card dividers, table rows |
| `--gray-medium` | `#E2E8F0` | Disabled bg, track |
| `--hover-background` | `#EEF4FF` | Nav/row hover |

### 2.4 Badge / Status palette (soft bg + dark text — 4.5:1)

| State | BG | Text | Used by |
|-------|----|----|---------|
| Active / Approved / GOOD | `green-100` | `green-700` | StatusBadge, RatingBadge, KanbanCard |
| Pending / step / NEEDS_IMPROVEMENT | `amber-100` | `amber-700` | StatusBadge (`PENDING_STEP_*`), RatingBadge |
| Rejected / FAIL | `red-100` | `red-700` | StatusBadge, RatingBadge, KanbanCard alerts |
| Approved (info) | `blue-100` | `blue-700` | "approved" chip on KanbanCard |
| Vendor-code pending | `purple-100` | `purple-700` | StatusBadge |
| Returned for edit | `orange-100` | `orange-700` | StatusBadge |
| Draft / neutral | `gray-100` | `gray-700` | StatusBadge default |

### 2.5 Role badge palette (Sidebar)

| Role | Class |
|------|-------|
| super_admin | `bg-purple-100 text-purple-700` |
| approver | `bg-green-100 text-green-700` |
| verifier | `bg-amber-100 text-amber-700` |
| accounting | `bg-cyan-100 text-cyan-700` |
| procurement | `bg-orange-100 text-orange-700` |
| requester (default) | `bg-blue-100 text-blue-700` |

---

## 3. Component → Color Map

| # | Component (type) | Exact component / file | Background | Text / Icon | Border / Accent |
|---|------------------|------------------------|-----------|-------------|-----------------|
| 1 | **App shell** | `pages/Layout.tsx` | canvas `#F0F4F9` | `--text-primary` | — |
| 2 | **Sidebar** | `components/sidebar/Sidebar.tsx` | white | `--text-secondary` items | right border `--gray-border` |
| 3 | Sidebar logo bar | Sidebar header | **primary `#1B3068`** | white | NTB tag = **accent pink** |
| 4 | Sidebar active item | `.nav-active` | **primary `#1B3068`** | white | — |
| 5 | Sidebar hover item | `.nav-item:hover` | `--hover-background #EEF4FF` | primary | — |
| 6 | Sidebar role chip | `getRoleBadgeClass()` | per-role 100 | per-role 700 | — |
| 7 | **Header** | `components/header/Header.tsx` | white | `--text-primary` title | bottom border `--gray-border`; **accent pink** underline bar |
| 8 | **Primary button** | buttons (Tailwind `bg-primary`) | primary → `primary-dark` hover | white | radius `--button-border-radius 8px` |
| 9 | **Accent button** | `.btn-accent` | **accent pink** → `accent-dark` hover | white | pink glow shadow on hover |
| 10 | **Danger button** | destructive actions | `--danger-color` → `--danger-dark` | white | — |
| 11 | Secondary/ghost button | outline buttons | white / transparent | primary | border `--gray-border` |
| 12 | **Card / panel** | `.shadow-card`, list cards | white | `--text-primary` | radius `--card-border-radius 12px`, `--card-shadow` |
| 13 | **DataTable** | `components/ui/DataTable.tsx` | white body | cells `--text-primary` | header `#F8FAFC`, row divider `--border-light`, hover `--hover-background` |
| 14 | **StatusBadge** | `components/ui/StatusBadge.tsx` | §2.4 soft bg | §2.4 dark text | `rounded-full` |
| 15 | **RatingBadge** | `components/ui/RatingBadge.tsx` | §2.4 (green/amber/red) | §2.4 | `rounded-full` |
| 16 | **Kanban column** | `components/kanban/KanbanColumn.tsx` | `gray-50`; locked `gray-100/70`; drop-target `blue-50 + ring-blue-300` | header `gray-700` | `border-t-4` (status color), count chip = status bg/text |
| 17 | **Kanban card** | `components/kanban/KanbanCard.tsx` | white | title `gray-700` | border `gray-200`; type/VAT/alert chips per §2.4 |
| 18 | **Modal / dialog** | `components/ui/ConfirmDialog.tsx`, kanban modals | white | `--text-primary` | scrim 40–60% black; danger CTA = red |
| 19 | **Form input** | form pages | white | `--text-primary`; placeholder `--text-tertiary` | border `--gray-border`; focus ring **primary** |
| 20 | Input error | validation | white | `--red-accent` message below field | border `--red-accent` |
| 21 | **LoadingSpinner** | `components/ui/LoadingSpinner.tsx` | — | **primary** | — |
| 22 | **Pagination** | `components/ui/Pagination.tsx` | white | primary active page | border `--gray-border` |
| 23 | **Toast** | react-toastify | white + semantic left bar | semantic | aria-live polite |
| 24 | **Avatar** | `components/kanban/ResponsibleAvatar.tsx` | role/initial tint | white initials | round |
| 25 | Links / inline action | anywhere | — | **primary**, hover `primary-light` | underline on hover |

**One primary CTA per screen.** Use accent pink for the single focal action or brand marker only; everything else is navy/neutral.

---

## 4. Typography

| Token | Value |
|-------|-------|
| Family | `'Plus Jakarta Sans', 'Noto Sans Thai', sans-serif` (Sarabun TTF bundled for Thai PDF) |
| Scale (px) | 12 · 14 · 16(base) · 18 · 24 · 32 |
| Body | 16px / line-height 1.5 |
| Headings (h1–h3) | weight 600, `--text-primary` |
| Labels | weight 500 |
| Money / counts | **tabular figures**, right-aligned (prevents column jitter) |
| Line length | 60–75 chars desktop |

---

## 5. Spacing, Radius, Shadow, Layout

| Token | Value |
|-------|-------|
| Spacing scale | 4 / 8 / 16 / 24 / 32 (`--spacing-xs…xl`), 8pt rhythm |
| Card radius | 12px · Button radius 8px |
| Card shadow | `0 1px 3px rgba(27,48,104,.08), 0 4px 12px rgba(27,48,104,.06)` |
| Card hover | `0 4px 12px rgba(27,48,104,.12), 0 8px 24px rgba(27,48,104,.08)` |
| Sidebar | 240px (expanded) / 64px (collapsed) |
| Header | 56px fixed |
| Content max | `max-w-7xl`, page padding 24px |
| z-index | sidebar 50 · header 40 · modal/scrim 1000 |
| Motion | 150–300ms, ease-out enter / ease-in exit; `transform`+`opacity` only |

---

## 6. Wireframes (ASCII)

### 6.1 App Shell (all authenticated pages)

```
┌────────────┬────────────────────────────────────────────────────────┐
│ ▮ Procurement│  Page Title                              ▂▂▂▂ (pink)   │ ← Header 56px (white)
│   [NTB]     ├────────────────────────────────────────────────────────┤
│  (navy bar) │                                                          │
├────────────┤                                                          │
│ ◧ ใบขอซื้อ   │   ┌──────────────────────────────────────────────────┐ │
│ ▦ Board     │   │                                                  │ │
│ ▤ คำขออนุมัติ│   │           PAGE CONTENT (cards / table)           │ │
│ ◫ คู่ค้า     │   │                                                  │ │
│ ☑ ประเมิน    │   │                                                  │ │
│ ✉ อีเมล      │   └──────────────────────────────────────────────────┘ │
│ ⚙ admin…    │                                                          │
│            │                                                          │
│ ─────────  │      canvas #F0F4F9, content padding 24px               │
│ 🅰 user [RQ]│                                                          │
│ ⎋ Logout   │                                                          │
└────────────┴────────────────────────────────────────────────────────┘
 Sidebar 240px (white, navy logo bar, active item = navy fill)
```

### 6.2 List Page (e.g. Purchase Requests / Vendors)

```
┌──────────────────────────────────────────────────────────────┐
│ ทะเบียนคู่ค้า                          [ + สร้างใหม่ ] (navy)  │ ← H1 + primary CTA
├──────────────────────────────────────────────────────────────┤
│ 🔎 ค้นหา…        [ สถานะ ▾ ] [ ประเภท ▾ ]      [ ⬇ Export ]   │ ← filter bar (white card)
├──────────────────────────────────────────────────────────────┤
│ ┌──────────────────────────────────────────────────────────┐ │
│ │ Code ▲ │ Name        │ Status      │ Rating   │ Actions   │ │ ← table header #F8FAFC, sortable
│ ├────────┼─────────────┼─────────────┼──────────┼───────────┤ │
│ │ V-001  │ ACME Co.    │ ◖อนุมัติ◗grn │ ◖ดี◗ green│ ✎  🗑      │ │ ← StatusBadge / RatingBadge
│ │ V-002  │ Beta Ltd.   │ ◖รอ◗ amber  │ ◖รอ◗ gray │ ✎  🗑      │ │   row hover = #EEF4FF
│ └──────────────────────────────────────────────────────────┘ │
│                                  ‹  1  2  3  ›  (active=navy)  │ ← Pagination
└──────────────────────────────────────────────────────────────┘
```

### 6.3 Procurement Board (Kanban)

```
┌──────────────────────────────────────────────────────────────────┐
│ Procurement Board        🔎 search   [ owner ▾ ] [ type ▾ ]        │
├──────────────────────────────────────────────────────────────────┤
│ ┃gray ┃navy ┃amber ┃green ┃red                                    │ ← border-t-4 = status color
│ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐                                │
│ │Draft│ │รอตรวจ│ │รออนุมัติ│ │อนุมัติ│ │ปฏิเสธ│   column bg = gray-50  │
│ │  3 │ │  5 │ │  2 │ │  8 │ │  1 │   count chip = status bg/text   │
│ ├────┤ ├────┤ ├────┤ ├────┤ ├────┤                                │
│ │┌──┐│ │┌──┐│ │    │ │┌──┐│ │    │   ← KanbanCard (white, gray-200)│
│ ││PR││ ││PR││ │    │ ││PR││ │    │     ◖type◗ chip, ◖VAT◗ grn/amber│
│ ││🔒││ ││  ││ │    │ ││  ││ │    │     🅰 owner avatar · due date  │
│ │└──┘│ │└──┘│ │    │ │└──┘│ │    │   drop target → blue-50 + ring  │
│ └────┘ └────┘ └────┘ └────┘ └────┘                                │
└──────────────────────────────────────────────────────────────────┘
```

### 6.4 Detail Page (PR / Approval Request)

```
┌──────────────────────────────────────────────────────────────┐
│ ‹ กลับ   PR-2026-0042   ◖รออนุมัติขั้นที่ 2◗ amber             │ ← back + title + StatusBadge
├───────────────────────────────────┬──────────────────────────┤
│  ┌─ รายละเอียด (card) ───────────┐ │ ┌─ Approval timeline ──┐ │
│  │ Requester · Dept · Date       │ │ │ ✓ ขั้นที่1  green     │ │
│  │ ─────────────────────────     │ │ │ ◷ ขั้นที่2  amber(now)│ │
│  │ Items table (qty · price · ∑) │ │ │ ○ ขั้นที่3  gray      │ │
│  │   tabular figures, right-align│ │ └──────────────────────┘ │
│  └───────────────────────────────┘ │ ┌─ Actions ───────────┐ │
│  ┌─ Comments (CommentSection) ───┐ │ │ [ อนุมัติ ] green     │ │ ← primary action group
│  │ 🅰 user · text · time          │ │ │ [ ส่งกลับ ] orange    │ │
│  │ [ พิมพ์… Cmd/Ctrl+Enter ส่ง ]   │ │ │ [ ปฏิเสธ ]  red       │ │ ← destructive separated
│  └───────────────────────────────┘ │ └──────────────────────┘ │
└───────────────────────────────────┴──────────────────────────┘
```

### 6.5 Form Page (Vendor / PR create-edit)

```
┌──────────────────────────────────────────────────────────────┐
│ ‹ กลับ   สร้างคู่ค้าใหม่                                        │
├──────────────────────────────────────────────────────────────┤
│ ┌─ ข้อมูลทั่วไป (fieldset) ─────────────────────────────────┐ │
│ │  ชื่อคู่ค้า *           [____________________]             │ │ ← label above, required *
│ │  เลขผู้เสียภาษี *        [____________________]             │ │
│ │                         helper text (text-secondary)      │ │
│ │  ประเภท                 [ เลือก ▾ ]                        │ │
│ │  ┗ error → border red + message below field               │ │ ← inline validation on blur
│ ├─ ผู้ติดต่อ (VendorContactsEditor) ────────────────────────┤ │
│ │  [ + เพิ่มผู้ติดต่อ ]                                      │ │
│ └───────────────────────────────────────────────────────────┘ │
│                              [ ยกเลิก ]   [ บันทึก ] (navy/CTA)│ ← cancel ghost + primary
└──────────────────────────────────────────────────────────────┘
```

### 6.6 Login Page

```
┌──────────────────────────────────────────────┐
│                                                │
│              ▮ Procurement [NTB]               │ ← logo: navy + pink tag
│           ระบบจัดซื้อจัดจ้าง                     │
│                                                │
│        ┌──────────────────────────┐            │
│        │  [ 🅖 Sign in with Google ]│           │ ← @react-oauth/google
│        └──────────────────────────┘            │
│                                                │
│        canvas #F0F4F9, centered white card     │
└──────────────────────────────────────────────┘
```

---

## 7. UX Guardrails (enterprise data-app priorities)

- **Accessibility (critical):** body 16px, contrast ≥4.5:1 (badges already pass), visible focus ring (navy), icon-only buttons need `aria-label`, status conveyed by icon+text not color alone.
- **Tables/data:** sortable headers with `aria-sort`, tabular figures for money, empty state ("ยังไม่มีข้อมูล" + action), skeleton/shimmer >300ms, virtualize lists ≥50 rows.
- **Forms:** label above field (not placeholder-only), required `*`, validate on blur, error below field + summary on submit, focus first invalid field, confirm before destructive/dismiss-with-unsaved.
- **Navigation:** active item highlighted (navy fill), role-gated items hidden cleanly, predictable back, preserve scroll/filter on return, destructive (logout) separated at sidebar bottom.
- **Feedback:** disable button + spinner during async, toast auto-dismiss 3–5s & `aria-live`, undo for bulk/destructive where possible.
- **Motion:** 150–300ms, `transform`/`opacity` only, respect `prefers-reduced-motion`, no layout shift.

---

## 8. Anti-Patterns to Avoid

- ✕ Pink (`accent`) as body text, large fills, or more than one focal point per screen.
- ✕ Emoji as functional icons — use lucide only.
- ✕ Raw hex in components — reference tokens / Tailwind token classes.
- ✕ Color-only status (always badge text + icon).
- ✕ Placeholder-as-label; errors only at top of form.
- ✕ Random shadow/radius values — use the scale in §5.
- ✕ Animating `width`/`height`/`top`/`left`.
