# Tengesa POS — Project Memory

Offline-first POS & stock management app for Zimbabwean SMEs (Yoco Counter model, Zimbabwe realities). Owner: MGX. Full specs live in `docs/` — read the relevant doc before implementing any feature.

## Golden rules

1. **SQLite is the source of truth.** The UI never talks to Supabase directly. Everything goes through the repository layer in `src/lib/db`. If a feature works only online, it is wrong.
2. **Money is integer USD cents.** ZWG is always derived (`usd_cents × rate`, rounded to nearest 0.05 ZWG) and never stored as a price. Every sale stores the `fx_rate_used`.
3. **Stock is a movement log.** On-hand is `SUM(qty_delta)` — never stored, never overwritten, never synced as a value.
4. **Every mutation writes its outbox row in the same SQLite transaction.** No exceptions, even before sync existed.
5. **Sales are append-only.** Corrections happen via refunds, never edits.
6. **Role gates live in the repository layer**, mirrored in UI. cashier < supervisor < admin.
7. `npm install` always with `--legacy-peer-deps`.
8. SQL migrations are written to `/supabase/migrations` but MGX applies them manually in the Supabase SQL Editor — never attempt to apply them.
9. MGX writes/commits code himself where he chooses; Claude's role is architecture, generation, review, debugging, instruction. Always plan before implementing; show the plan.
10. No paid dependencies, services, fonts or APIs. Supabase free tier + Google Fonts + Lucide only.

## Stack

React 18 + TS strict + Vite + Capacitor 6 (Android) · Tailwind + shadcn/ui · Zustand · @capacitor-community/sqlite (encrypted) · Supabase (Postgres/Auth/Storage/Edge Functions) · Vitest + Playwright · Sentry free tier · GitHub Actions.

## Design tokens (authoritative copy in docs/03_DESIGN.md)

Primary `#0B7A3B`, danger `#D93025`, accent `#F5A623`; USD amounts green, ZWG amounts blue `#2563EB`; Inter with tabular numerals; 4pt grid; 48dp min touch; buttons are verbs ("Charge US$12.50").

## Key identifiers

- Package: `com.mgx.tengesa`
- Sale numbers: server `TNG-YYYY-NNNNNN`, provisional offline `TNG-L-xxxx`
- Bluetooth printing (v1.1): service `000018f0`, characteristic `00002af1`
- Tenders: cash_usd, cash_zwg, ecocash, zipit, card, other — record-keeping only, no processing

## Definition of done (per feature)

Works in airplane mode → unit tests on any money/stock math → outbox rows verified → role gates tested → qa-reviewer subagent pass → no new lint errors → docs updated if behaviour changed.

## Decisions log

- 2026-07: Adopted this doc pack as Tengesa's formal spec. Custom sync engine over PowerSync/ElectricSQL (control + zero cost). ZWG derived, not stored. Tips & split bills deferred to v1.1.
<!-- append new decisions below with date -->
