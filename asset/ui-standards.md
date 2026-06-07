# Procurement System — UI Standards

**Primary source of truth:** `asset/MASTER.md`  
**Reference implementation:** `asset/gr-prototype.html`

All HTML prototypes MUST follow these standards. Every value here is derived directly from `MASTER.md`. Do not invent one-off values. When in doubt, read `MASTER.md` first.

---

## Colors

Derived from `MASTER.md §2`.

```css
/* Brand */
--primary:          #1B3068;   /* deep navy — buttons, links, active nav, headings */
--primary-dark:     #142451;   /* pressed / hover-dark */
--primary-light:    #2A4A99;   /* hover on primary surfaces */
--primary-bg:       #EEF4FF;   /* row hover, selected bg */
--accent:           #E8518A;   /* Turbo pink — one focal CTA per screen, header bar, NTB tag */
--accent-dark:      #C93D72;   /* accent hover */

/* Semantic */
--success:          #10B981;   /* approved, confirmed, active */
--success-light:    #D1FAE5;   /* success badge bg */
--warning:          #F59E0B;   /* pending, needs attention */
--warning-light:    #FEF3C7;   /* warning badge bg */
--danger:           #EF4444;   /* error, rejected, destructive */
--danger-light:     #FEE2E2;   /* danger badge bg */

/* Surfaces */
--bg-canvas:        #F0F4F9;   /* app body / page background */
--bg-primary:       #FFFFFF;   /* cards, sidebar, header, modals */
--bg-secondary:     #F8FAFC;   /* table header, subtle panels, toolbar */

/* Borders & text */
--border:           #CBD5E1;   /* dividers, input borders */
--border-light:     #EEF2F7;   /* inner card dividers, table rows */
--text-primary:     #0F1F42;   /* headings, body, table cells */
--text-secondary:   #4B5E85;   /* labels, helper text, meta */
--text-muted:       #8799B8;   /* placeholder, disabled, timestamps */
```

---

## Typography

Derived from `MASTER.md §4`.

```css
font-family: 'Plus Jakarta Sans', 'Noto Sans Thai', sans-serif;
/* Import: @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700;800&display=swap'); */
/* Noto Sans Thai covers all Thai characters at every weight */
```

| Usage | Size | Weight |
|---|---|---|
| Page title (H1) | 24px | 600 |
| Section / card label | 11px, uppercase, letter-spacing 0.6px | 700 |
| Form labels | 13px | 600 |
| Body / table data | 14px | 400 |
| Small hints / muted | 11–12px | 400 |
| Money / counts | tabular figures, right-aligned | 600 |
| Codes (GR, PO, GL numbers) | `'Plus Jakarta Sans', sans-serif`, 12px | 600 |

---

## App Shell

Derived from `MASTER.md §5, §6.1`.

- **App canvas:** `background: #F0F4F9`
- **Content max-width:** `max-w-7xl` (≈1280px), page padding 24px

### Header (top bar)
- Height: 56px, fixed, `z-index: 40`
- Background: `#FFFFFF`
- Bottom border: `1px solid #CBD5E1`
- Bottom accent bar: `3px solid #E8518A` (Turbo pink — brand identity)
- Left: Logo mark + system name + module name
- Right: User name + role chip + Log out button

```html
<header class="app-header">
  <div class="brand">
    <div class="brand-mark">P</div>
    <span class="brand-text">Procurement</span>
    <span class="brand-sep">/</span>
    <span class="brand-module">Module Name</span>
  </div>
  <div class="header-right">
    <span class="user-info">Name — Role</span>
    <button class="btn-logout">Log out</button>
  </div>
  <div class="header-accent-bar"></div><!-- pink bottom bar -->
</header>
```

```css
.app-header {
  background: #FFFFFF;
  border-bottom: 1px solid #CBD5E1;
  position: relative;
}
.header-accent-bar {
  position: absolute; bottom: 0; left: 0; right: 0;
  height: 3px; background: #E8518A;
}
```

### Sidebar
- Width: 240px (expanded), fixed left, `z-index: 50`
- Background: `#FFFFFF`, right border `1px solid #CBD5E1`
- **Logo bar** (top of sidebar): `background: #1B3068`, white text, NTB tag uses accent pink
- Nav items: 13px, weight 500, `color: #4B5E85`
- **Active item:** `background: #1B3068; color: #FFFFFF; font-weight: 600`
- Hover: `background: #EEF4FF; color: #1B3068`
- Role chip at bottom: per-role soft bg/text (see `MASTER.md §2.5`)

---

## Page Layout

```css
.page-container { max-width: 1280px; margin: 0 auto; padding: 1.5rem; }
```

---

## Buttons

Derived from `MASTER.md §3`.

| Class | Background | Text | Usage |
|---|---|---|---|
| `.btn-primary` | `#1B3068` → `#142451` hover | white | Primary action |
| `.btn-accent` | `#E8518A` → `#C93D72` hover | white | Single focal CTA per screen (use sparingly) |
| `.btn-secondary` | `#FFFFFF`, border `#CBD5E1` | `#1B3068` | Cancel / secondary |
| `.btn-danger` | `#EF4444` → `#DC2626` hover | white | Destructive confirm |
| `.btn-logout` | transparent, border white 30% | white | Header / sidebar only |

- Border-radius: 8px
- Font: 13px, weight 600, inherit
- Padding large: `0.625rem 1.25rem`
- Padding small (`.btn-sm`): `0.375rem 0.75rem`, 12px, weight 500
- **One primary CTA per screen.** Use accent pink only for the single focal action or brand marker.

---

## Cards

```css
.card {
  background: #FFFFFF;
  border: 1px solid #CBD5E1;
  border-radius: 12px;
  box-shadow: 0 1px 3px rgba(27,48,104,.08), 0 4px 12px rgba(27,48,104,.06);
}
.card:hover {
  box-shadow: 0 4px 12px rgba(27,48,104,.12), 0 8px 24px rgba(27,48,104,.08);
}
```

---

## Toolbar Pattern (inside card)

Two-row pattern for data-dense pages:

```
Row 1 — Actions:   [ 🔍 Search... ]                    [ Export ]  [ View Toggle ]  [ + Primary Action ]
Row 2 — Filters:   FILTER  [ Date From ] [ – ] [ Date To ]  [ Status ▾ ]  [ Type ▾ ]  [ Reset ]
```

- Row 1 background: `#FFFFFF`, `border-bottom: 1px solid #CBD5E1`
- Row 2 background: `#F8FAFC`, `border-bottom: 1px solid #CBD5E1`
- Padding: `12px 16px`

### Search input
```html
<div class="search-wrap">
  <span class="search-icon"><!-- SVG, stroke #1B3068 --></span>
  <input class="search-input" type="text" placeholder="Search…" />
  <button class="search-clear">✕</button>
</div>
```

### View toggle (Document / Line Item)
```html
<div class="view-toggle">
  <button class="vbtn active">Document View</button>
  <button class="vbtn">Line Item View</button>
</div>
```
Active state: `background: #1B3068; color: #FFFFFF`

---

## Form Elements

- Border-radius: 8px
- Border: `1px solid #CBD5E1`
- Focus: `border-color: #1B3068` + `box-shadow: 0 0 0 3px rgba(27,48,104,0.15)`
- Font: inherit (Plus Jakarta Sans / Noto Sans Thai)
- Height for inline filter inputs: 36px
- Labels: 13px, weight 600, `color: #0F1F42`
- Required asterisk: `<span class="req">*</span>` in `#EF4444`
- Selects: custom chevron, `appearance: none`
- Error state: border `#EF4444`, error message below field in `#EF4444`

---

## Table

Derived from `MASTER.md §3 #13`.

```css
thead tr  { background: #F8FAFC; border-bottom: 2px solid #CBD5E1; }
thead th  { font-size: 11px; font-weight: 700; text-transform: uppercase; letter-spacing: 0.5px; color: #4B5E85; padding: 12px 16px; }
tbody tr  { border-bottom: 1px solid #EEF2F7; transition: background 0.12s; }
tbody tr:hover { background: #EEF4FF; }
tbody td  { padding: 13px 16px; font-size: 14px; color: #0F1F42; }
```

- All data text: `#0F1F42` — no colored data text except status badges
- Links (GR/PO numbers): `color: #1B3068`, underline on hover
- Money columns: tabular figures, right-aligned, weight 600
- Expand button: 22×22px, border, `▶` / `▼` toggle

### Sub-table (expandable rows)
- Background: `#F8FAFC`
- Border-top: `1px solid #CBD5E1`
- Hover: `background: #EEF4FF`

---

## Status Badges

Derived from `MASTER.md §2.4`. Pill shape, `border-radius: 100px`, dot prefix via `::before`.

```css
/* Confirmed / Active / Approved */
.b-confirmed { background: #D1FAE5; color: #065F46; }
.b-confirmed::before { background: #10B981; }

/* Reversed / Draft / Neutral */
.b-reversed  { background: #F1F5F9; color: #475569; }
.b-reversed::before  { background: #94A3B8; }

/* Reversal / Rejected / Danger */
.b-reversal  { background: #FEE2E2; color: #991B1B; }
.b-reversal::before  { background: #EF4444; }

/* Pending / Warning */
.b-pending   { background: #FEF3C7; color: #92400E; }
.b-pending::before { background: #F59E0B; }
```

Dot: `::before { content:''; width:6px; height:6px; border-radius:50%; display:inline-block; margin-right:6px; }`

### Type badges (Asset / Expense)
```css
.b-asset   { background: #EFF6FF; color: #1D4ED8; padding: 2px 8px; border-radius: 6px; }
.b-expense { background: #FFFBEB; color: #92400E; padding: 2px 8px; border-radius: 6px; }
```

---

## Modal

Derived from `MASTER.md §3 #18`.

- Backdrop: `rgba(0,0,0,0.45)`, `z-index: 1000`
- Modal: `border-radius: 12px`, `background: #FFFFFF`, animation `scale(0.96) → scale(1)`
- **Header:** `background: #FFFFFF`, `border-bottom: 1px solid #CBD5E1`, title 18px/700 `#0F1F42`, subtitle 12px `#4B5E85`
- Close button: `background: #F1F5F9`, `color: #4B5E85`, 32×32px, border-radius 8px
- Body: `padding: 1.5rem`
- Footer: `background: #F8FAFC`, border-top `1px solid #CBD5E1`, Cancel + primary action right-aligned

---

## Toast Notifications

Derived from `MASTER.md §3 #23`. Use instead of `alert()`.

- White background + left semantic color bar
- Auto-dismiss: 3–5 seconds
- Position: `fixed; top: 68px; right: 24px; z-index: 9999`
- Success: left bar `#10B981`, icon green checkmark
- Error: left bar `#EF4444`

---

## Layer Cards (form sections inside modal)

```css
.layer-card { border: 1px solid #CBD5E1; border-radius: 10px; padding: 1.25rem; background: #FFFFFF; }
.layer-card-header { border-bottom: 2px solid #CBD5E1; padding-bottom: 8px; margin-bottom: 1rem; }
.layer-card-title { font-size: 11px; font-weight: 700; text-transform: uppercase; letter-spacing: 0.6px; color: #4B5E85; }
```

---

## Border Radius Scale

| Value | Usage |
|---|---|
| 6px | Small — type badges |
| 8px | Standard — buttons, inputs, close button |
| 10px | Layer card |
| 12px | Card, modal |
| 100px | Badge pill |

---

## Scrollbar

```css
::-webkit-scrollbar { width: 6px; height: 6px; }
::-webkit-scrollbar-track { background: transparent; }
::-webkit-scrollbar-thumb { background: #CBD5E1; border-radius: 3px; }
```

---

## What NOT to do

Derived from `MASTER.md §8`.

- Do NOT use a font other than Plus Jakarta Sans / Noto Sans Thai
- Do NOT use `alert()` — use toast notifications
- Do NOT hardcode colors outside this palette
- Do NOT use pink (`--accent`) as body text, large fills, or more than one focal point per screen
- Do NOT use emoji as functional icons
- Do NOT use `border-radius` values other than: 6px / 8px / 10px / 12px / 100px
- Do NOT animate `width` / `height` / `top` / `left` — use `transform` + `opacity` only
- Do NOT use color alone to convey status — always pair with text label
