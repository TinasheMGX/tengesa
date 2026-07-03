# Design Document & Frontend Guidelines — Tengesa POS

## 1. Design principles (in priority order)

1. **Speed over beauty.** Every design decision is judged by "does this slow down a queue?" Yoco's sell screen wins because a cashier never thinks — they tap tiles and charge.
2. **Glanceable in sunlight.** Market stalls and shopfronts are bright. High-contrast light theme is the default; dark theme optional.
3. **One-hand, thumb-first on phone.** Charge button, tender buttons, and PIN pad live in the bottom 40% of the screen.
4. **Trust through clarity.** Money amounts are always the largest text on screen. Currency is never ambiguous — every amount is prefixed (US$ / ZWG).
5. **Forgiving.** Destructive actions (refund, delete product) need confirmation + role; everything else is instantly undoable or editable.

## 2. Visual language

### Design tokens

```
Color
  --primary:        #0B7A3B   (market-stall green — distinct from Yoco blue; NBS-adjacent but own identity)
  --primary-dark:   #085C2C
  --accent:         #F5A623   (amber — alerts, low stock)
  --danger:         #D93025   (refunds, destructive)
  --success:        #1E8E3E
  --ink:            #111827   (primary text)
  --ink-muted:      #6B7280
  --surface:        #FFFFFF
  --surface-alt:    #F3F4F6
  --border:         #E5E7EB
  --usd:            #0B7A3B   (USD amounts)
  --zwg:            #2563EB   (ZWG amounts — colour-coded to prevent till errors)

Typography (Google Fonts — free)
  Display/amounts:  "Inter" 700, tabular numerals (font-variant-numeric: tabular-nums)
  Body:             "Inter" 400/500
  Scale:            32/24/20/16/14/12 — amounts on tender screen: 40px+

Spacing: 4pt grid (4/8/12/16/24/32)
Radius:  12px cards, 16px sheets, 999px pills
Touch:   min 48×48dp; primary buttons 56dp tall
Icons:   Lucide (free, consistent with Shop-Flow)
Elevation: borders + subtle shadow only at level 1; no heavy shadows
```

### Component inventory (shadcn/ui base, customised)
Button (primary/secondary/danger/ghost) · ProductTile (image, name, price, stock badge) · CartLine · AmountKeypad · PINPad · TenderButton (large, icon + label) · CurrencyAmount (paired US$/ZWG display) · StatCard (reports) · SyncBadge (green dot / amber pending count / red attention) · StaffAvatar · Sheet (variants, filters) · EmptyState (friendly, action-led) · ReceiptCard.

## 3. Key screen specs (Yoco-informed)

**Sell (home).** Tablet: left rail nav, centre = tile grid with category chips across the top, right = persistent cart panel with green Charge button pinned bottom-right (Yoco Counter layout). Phone: full-screen tile grid, floating cart pill bottom-right showing item count + total; tapping opens cart sheet. A keypad toggle (⌗) switches to quick-amount mode — Yoco's dual input pattern.

**Tender.** Total dominates the top in both currencies (USD large, ZWG beneath in blue). Tender types as a 2-column grid of large buttons. Amount-received keypad with smart suggestions (exact, next note: $5, $10, $20). Change line auto-computes, including "change in ZWG for USD payment" cross-currency case.

**History.** Search bar + filter chips (today, tender, staff, refunded). List rows: sale number, time, staff initial, tender icon, amount. Detail screen: line items, timeline (created → synced → refunded), actions row (Refund · Receipt · Note).

**Reports.** Period selector (Today / Week / Month / Custom) → stat cards (Gross sales, Transactions, Avg sale, Margin) → simple bar chart (Recharts) → breakdown lists. No dashboard clutter — one question per screen.

**Stock.** List with on-hand count and coloured badge (green ok / amber low / red out). Adjustment screen is a form, not a spreadsheet.

## 4. Interaction & motion

- Sale commit: instant optimistic UI + short green check animation (≤ 300 ms); never block on sync.
- Tile tap: 80 ms scale-down feedback; quantity badge increments in place (no navigation).
- Sheets over pages wherever a task takes < 15 seconds (variants, filters, PIN).
- Skeletons only on first cloud pull; local reads should never show a spinner.
- Haptics on charge success and PIN error (Capacitor Haptics).

## 5. Reference & inspiration sources

| Source | What to study |
|---|---|
| **Yoco POS App** (Play Store: `za.co.yocopos`) | Sell/History/Reports/More structure, tile view, tender screen, refund flow. Screenshot the listing images into `/docs/design/refs/` |
| **Yoco Counter marketing pages** (yoco.com/za/point-of-sale) | Tablet layout, cart panel, staff PIN UX |
| **Mobbin** (mobbin.com — free tier) | Search: "Square POS", "SumUp", "Loyverse", "Shopify POS" — checkout flows, keypad patterns, refund UX |
| **Loyverse POS** (free app) | Closest free competitor — install it, note what feels slow |
| **Behance** | Search: "POS app UI", "point of sale dashboard", "fintech Africa app" |
| **Figma Community** (free) | Search: "POS UI kit", "shadcn design system", "Untitled UI free" — use as wireframe starting points |
| **Dribbble** | "POS tablet", "cashier app" — visual polish references only, not flows |
| **M-Pesa / EcoCash apps** | What local users already understand for money UX (confirmation patterns, amount display) |

Workflow: collect 15–20 reference screenshots → annotate in FigJam (free) → wireframe the 6 core screens in Figma free tier → only then generate UI with Claude Code (feed it the annotated refs).

## 6. Frontend engineering guidelines

- **Stack:** React 18 + TypeScript strict + Vite + Capacitor 6; Tailwind + shadcn/ui; Zustand for UI state; TanStack Query only for cloud reads (local reads go straight to SQLite hooks).
- **Structure:** feature folders (`/features/sell`, `/features/stock`…) each owning components, hooks, and local queries; shared `/components/ui` and `/lib`.
- **Rules:**
  - No component may call Supabase directly — everything goes through the repository layer (`/lib/db`) so offline behaviour is uniform.
  - All money as **integer cents** in USD; ZWG derived at render/tender time. Never floats.
  - Every list virtualised past 50 items (`@tanstack/react-virtual`).
  - Buttons never say "OK" — verbs only ("Charge US$12.50", "Refund", "Save bill").
  - Design tokens live in `tailwind.config.ts` + CSS vars; no hard-coded hex in components.
  - Storybook skipped (budget/time); instead a `/dev/kitchen-sink` route rendering every component state.
- **Testing:** Vitest for money/rate/change-calculation logic (unit tests are mandatory here), Playwright for the sale happy path.
- **Accessibility:** labels on all inputs, `aria-live` on cart total, contrast ≥ 4.5:1, respects system font scaling up to 130%.
