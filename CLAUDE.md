# Tengesa POS — Project Memory (v2)

Offline-first POS & stock management for Zimbabwean SMEs (Yoco Counter model, Zimbabwe realities: dual USD/ZWG, cash-dominant, intermittent connectivity). Owner: MGX. Near-zero budget. Developed with Claude Code.

**Authoritative docs:** `docs/00_MASTER_PLAN.md` (programme, 8 phases, gates) · `docs/01_PRD.md` (requirements, MoSCoW, acceptance criteria). Read the relevant phase doc before any work. `docs/archive/v1/` is superseded source material — never treat it as current spec without checking.

## Programme state

- Process: **strictly sequential phases with review gates.** Never begin the next phase without MGX's explicit gate approval.
- Current gate: **GATE 2 — UX & App Flow v1.0 approved (2026-07-04).** Phase 3 may begin on MGX approval.
- Platform plan: release-train Option B (adopted Gate 1) — R1 Android phone+tablet → R1.5 web dashboard/analytics → R2 desktop → R3 customer portal → R4 iOS.

## Gate 1 decisions (resolved 2026-07-04)

1. Release train Option B adopted? — **Yes.** R1 Android → R1.5 web dashboard → R2 desktop → R3 customer portal → R4 iOS.
2. Credit book (FR-E2) promoted to R1 Must? — **No.** Deferred to a future release. Stays Should.
3. Customer portal job-to-be-done (for R3 mini-PRD)? — **Receipts lookup + credit statements.** Token-authenticated, no customer login. Mini-PRD at R3.
4. This PRD formally supersedes Shop-Flow/Boss Kuku scope? — **Yes.** Tengesa is the product identity. v1 archive is source material only.
5. VAT/tax line needed for pilot merchants? — **No, deferred post-R1.** Rate corrected to 15.5%. Most pilot merchants unregistered.
6. Multi-store: schema now, feature R2 — confirmed? — **Yes.** `store_id` on key tables from Phase 4; UI is single-store in R1.

## Golden rules (binding on all code, all phases)

1. **Local SQLite is the source of truth.** UI never talks to the cloud SDK directly; everything goes through the repository layer. A feature that only works online is wrong.
2. **Money is integer USD cents.** ZWG is always derived (`usd_cents × rate`, rounded to nearest 0.05 ZWG), never stored as a price. Every sale stores `fx_rate_used`.
3. **Stock is an append-only movement log.** On-hand = SUM(qty_delta) — never stored as truth, never overwritten, never synced as a value.
4. **Every mutation writes its outbox row in the same local transaction.** No exceptions.
5. **Sales are append-only.** Corrections happen via linked refunds, never edits.
6. **Role gates live in the data layer** (cashier < supervisor < admin), mirrored in UI.
7. Sync must be **idempotent** — replaying any batch is a no-op.
8. `npm install` always with `--legacy-peer-deps`.
9. SQL migrations are written to `/supabase/migrations`; **MGX applies them manually** in the Supabase SQL Editor — never attempt to apply, never touch live data.
10. **No paid anything** without MGX's written approval: free/open-source tools, Supabase-class free tier (final DB decision with justification = Phase 4), Google Fonts + Lucide only.
11. MGX drives, verifies and commits; Claude architects, generates, reviews, debugs, instructs. **Always plan before implementing and show the plan.**
12. Docs update in the same commit as any behaviour change.

## Working preferences (MGX)

Actionable over abstract; no clutter or over-planning; plain concise outputs; calm sustainable cadence — phase gates are finish lines, respect them. When drafting anything merchant- or bank-adjacent, keep it professional and direct.

## Stack (pending Phase 4 ratification)

React 18 + TypeScript strict + Vite + Capacitor 6 (Android first) · Tailwind + shadcn/ui · Zustand · encrypted SQLite (`@capacitor-community/sqlite`) · Supabase (Postgres/Auth/Storage/Edge Functions) free tier · Vitest + Playwright · Sentry free tier · GitHub Actions. Custom outbox/pull-cursor sync engine (Phase 5 formalises).

## Key identifiers

- Package: `com.mgx.tengesa` (immutable once published)
- Sale numbers: server `TNG-YYYY-NNNNNN`; provisional offline `TNG-L-xxxx`
- Tenders (record-keeping only, never processed): cash_usd, cash_zwg, ecocash, zipit, card, other
- Bluetooth printing (R2): service `000018f0`, characteristic `00002af1`
- Design anchors: primary `#0B7A3B`, USD green / ZWG blue `#2563EB`, Inter tabular numerals, 48dp targets, verbs on buttons

## Definition of done (any feature, build phases)

Works in airplane mode → unit tests on money/stock/rate math → outbox rows verified → role gates tested → qa-reviewer subagent pass → lint clean → docs updated.

## Subagents

`.claude/agents/`: **engineer** (implementation, transactions), **ux-architect** (flows, screens, taps-to-complete), **qa-reviewer** (correctness, money math, MUST run before any phase gate), **sync-specialist** (sync protocol, convergence). Invoke explicitly by name.

## Decisions log

- 2026-07: Programme restructured to 8-phase sequential brief (master plan v2). v1 doc pack archived as source material. Competitive wedge confirmed: free inventory + staff management (Loyverse charges) + dual-currency (nobody has) + true offline-first.
- 2026-07-04: Gate 1 approved — PRD v2.1. Option B release train adopted. Credit book deferred (future release). Customer portal = receipts + credit statements. Tengesa identity confirmed. VAT deferred (15.5% when built). Multi-store schema day 1, feature R2.
- 2026-07-04: Gate 2 approved — UX & App Flow v1.0. 41 screens, 15 sections. Horizontal scroll category chips. Receipt provisional number is permanent. Z-report image-only R1. Cash-up admin-only. Void = full refund (same flow).
