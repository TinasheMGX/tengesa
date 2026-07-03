# Claude Code Prompt Pack — Tengesa POS

How to use: one phase per Claude Code session. Paste the phase prompt, let it plan first, review the plan, then approve implementation. Commit at phase end. Claude Code auto-loads `CLAUDE.md` (project memory) and `.claude/agents/` (subagents). MGX rule of engagement: **Claude architects, generates and reviews; MGX drives, verifies and commits.**

---

## Phase 0 — Scaffold & foundations

```
Read docs/01_PRD.md, docs/03_DESIGN.md and docs/04_BACKEND.md fully before writing anything.

Scaffold the Tengesa POS app:
- Vite + React 18 + TypeScript strict + Capacitor 6 (Android platform added)
- Tailwind + shadcn/ui with the design tokens from docs/03_DESIGN.md §2 wired into tailwind.config.ts and CSS vars
- @capacitor-community/sqlite set up with an encrypted database, a migration runner, and migration 001 implementing the FULL local schema from docs/04_BACKEND.md §3 (including outbox and sync_state)
- Repository layer skeleton in src/lib/db with the interfaces from docs/04_BACKEND.md §5 — implemented against SQLite, no Supabase anywhere yet
- Feature-folder structure (features/sell, features/history, features/stock, features/reports, features/settings)
- Bottom-tab navigation shell (phone) with the 5 tabs from docs/02_APP_FLOW.md §1, plus a /dev/kitchen-sink route
- Vitest configured; write the money utilities (integer cents, ZWG derivation, rounding to nearest 0.05, cross-currency change) WITH exhaustive unit tests first
- npm installs must use --legacy-peer-deps

Use the ux-architect subagent to sanity-check the navigation shell, then the qa-reviewer subagent on the money utilities before finishing. Plan first; show me the plan before implementing.
```

## Phase 1 — Catalogue & stock (offline only)

```
Implement the catalogue and stock features per docs/01_PRD.md §5.1 and flows in docs/02_APP_FLOW.md §5:
- Product CRUD (name, image [local file for now], category, brand, cost, price, barcode, SKU, variants), soft delete
- Category management with sort order
- Stock as a movement log ONLY (docs/04_BACKEND.md schema) — on-hand always derived; opening stock, received, damaged, count-correction flows with reasons and staff stamping
- Low-stock threshold + badge on tiles and a Stock tab alert list
- CSV import/export mapping to my existing tuckshop catalogue format (ask me for a sample file before implementing the mapper)
All offline against SQLite. Every mutation must also write an outbox row in the same transaction (payloads per docs/04_BACKEND.md §4) even though sync doesn't exist yet.
Use qa-reviewer on the movement-log math (count corrections especially).
```

## Phase 2 — Sell flow (the money path)

```
Implement the complete sale flow per docs/02_APP_FLOW.md §3 and design specs in docs/03_DESIGN.md §3:
- Sell screen: tile grid + category chips + quick-amount keypad toggle; phone cart pill + cart sheet
- Variant/modifier bottom sheet
- Cart: qty steppers, line discount, bill discount, sales note
- Tender screen: dual-currency total display, tender-type grid (cash_usd, cash_zwg, ecocash, zipit, card, other — driven by settings toggles), amount-received keypad with smart suggestions, cross-currency change calculation
- Atomic sale commit: sale + lines + payments + stock movements + outbox in ONE SQLite transaction; optimistic UI; provisional sale numbers TNG-L-xxxx
- Saved/open bills (park + resume)
- Receipt: rendered card matching docs/03_DESIGN.md, exported as image to the Android share sheet
- Staff PIN lock screen + staff switching + role gates (discount > X% needs supervisor)
Write a Playwright test for the happy path and a kill-app-mid-commit test proving atomicity. This is the heart of the app — use the engineer subagent for the transaction logic and qa-reviewer before finishing.
```

## Phase 3 — Sync engine & Supabase

```
Stand up the cloud side and sync per docs/04_BACKEND.md §2–§4:
- Write the Postgres schema + RLS policies as SQL migration files in /supabase/migrations (I will run them in the Supabase SQL Editor myself — do not attempt to apply them)
- Edge functions sync/push and sync/pull exactly per the protocol: batched ops, idempotency keys table, per-op acks, server sale numbers, cursor-based pull with tombstones
- Client sync engine: outbox drain, pull-and-apply in FK order, conflict rules (LWW for products, additive for stock, append-only for sales), triggers (Network plugin, foreground, post-sale debounced, manual), exponential backoff
- Sync status UI in More tab: pending count, last synced, NeedsAttention list
- Owner auth screens (Supabase email/password), device registration, restore-to-new-device flow
Test plan: two emulators, same account, offline edits on both, verify convergence. Use the sync-specialist subagent to review the protocol implementation and qa-reviewer for the idempotency replay test.
```

## Phase 4 — History, refunds & reports

```
Implement per docs/02_APP_FLOW.md §4 and §8, PRD FR-04, FR-09:
- History tab: search, filter chips, sale detail with timeline, edit note, reshare receipt
- Refund flow: supervisor gate, full/partial, required reason, optional restock, linked negative record, updated receipt
- Reports: Today/Week/Month/Custom; stat cards (gross, count, avg, margin from cost prices); breakdowns by tender/product/category/staff; simple Recharts bars
- End-of-day cash-up per currency with variance capture and shareable Z-report image
All reports compute from local SQLite (must work offline). Use qa-reviewer on refund math and cash-up reconciliation.
```

## Phase 5 — Hardening & Play Store release

```
Work through docs/05_SECURITY_PLAYSTORE.md as a literal checklist. For each item: implement or verify, and tick it in the doc with a note.
Additionally:
- Onboarding flow per docs/02_APP_FLOW.md §2 including offline trial mode
- Account deletion (in-app trigger + edge function cascade)
- Privacy policy draft (I'll host on GitHub Pages)
- Sentry integration with PII scrubbing
- GitHub Actions workflow: lint, test, build signed AAB (keystore via secrets)
- ProGuard rules, allowBackup=false, cleartext off, target SDK per current Play requirement (check docs, ask me to verify in Play Console)
- Store listing copy + screenshot shot-list
Finish with the qa-reviewer subagent doing a full-app review against the PRD's functional requirements table, producing a gap report.
```

## Ongoing session hygiene

- Start sessions with: "Read CLAUDE.md and the phase context. Summarise current state from git log before proposing work."
- After decisions: `# remember: <decision>` to append to project memory.
- Weekly: "Use qa-reviewer to audit the last week's commits against docs/01_PRD.md and update the open-questions list."

## Claude Skills to create in-repo (`.claude/skills/`)

1. **supabase-migration** — house rules for writing migration SQL destined for the SQL Editor (idempotent, RLS included, rollback noted)
2. **money-math** — the cents/ZWG/rounding rules with worked examples, so every generated component gets it right
3. **receipt-render** — receipt layout spec + share-sheet export recipe
4. **sync-op** — template for adding a new synced entity (schema both sides, outbox payload, pull apply, conflict rule)
