# Product Requirements Document — Tengesa POS
**Version:** 1.0 · **Owner:** MGX · **Status:** Draft for review · **Date:** July 2026

---

## 1. Vision

A free-to-run, offline-first Android POS and stock management app for Zimbabwean small traders (tuckshops, boutiques, salons, takeaways, hardware kiosks), matching the simplicity of Yoco Counter but built for Zimbabwe's realities: dual-currency (USD/ZWG), intermittent connectivity, load shedding, cash-dominant tendering, and EcoCash/Zipit as everyday payment methods.

**One-line pitch:** "Yoco Counter for Zimbabwe — sell, track stock, and know your numbers, even with no network."

## 2. Problem statement

Zimbabwean SME traders run on exercise books and memory. Existing POS apps assume constant connectivity, single currency, and integrated card processing — none of which hold locally. Yoco proved the model in SA: free software, dead-simple selling screen, item-level stock, staff PINs, reports. No equivalent exists for the Zimbabwean informal/SME market.

## 3. Target users & personas

| Persona | Context | Key needs |
|---|---|---|
| **Mai Tino — tuckshop owner** | Sells ~150 SKUs, USD & ZWG, phone-based, no laptop | Fast checkout, dual-currency pricing, end-of-day cash total, works offline |
| **Kuda — boutique owner** | Product variants (size/colour), 1–2 staff | Variants, staff PINs, refund control, stock counts |
| **Tatenda — takeaway operator** | Fast queue, combos, saved orders | Tile catalogue, speed, saved bills, daily best-sellers |
| **Owner-manager (remote)** | Checks business from anywhere | Reports on a second device once synced |

Primary device: low/mid-range Android phone (2–4 GB RAM, Android 10+). Secondary: cheap Android tablet at the counter.

## 4. Goals & success metrics

| Goal | Metric | Target (6 months post-launch) |
|---|---|---|
| Usable offline | % of sales captured with no connectivity that sync successfully | ≥ 99.5% |
| Speed at the till | Time to complete a 3-item cash sale | ≤ 10 seconds |
| Adoption | Active merchants | 50 (pilot), 500 (year 1) |
| Reliability | Crash-free sessions | ≥ 99.5% |
| Cost | Monthly infra cost | $0 (Supabase free tier) |

## 5. Scope

### 5.1 MVP (Phase A — Play Store launch target)

**Selling**
- Sell screen: quick-amount entry AND tile-based product catalogue (Yoco pattern)
- Cart: quantities, line discounts, whole-bill discount, sales notes
- Tender types (record-keeping, per Yoco's model): Cash USD, Cash ZWG, EcoCash, Zipit/bank transfer, Card (bank POS), Other — configurable toggles
- Dual-currency: prices held in USD; ZWG computed at a merchant-set daily rate; tender in either; change calculation across currencies
- Saved/open bills (park a sale, resume later)
- Digital receipt: shareable image/PDF via WhatsApp/SMS share sheet (no SMS gateway cost)

**Catalogue & stock**
- Products: name, image, category, brand, cost price, selling price (USD), barcode, SKU
- Variants (size/colour) and simple modifiers
- Item-level stock tracking: opening stock, deduction on sale, manual adjustments (received, damaged, count correction) with reasons
- Low-stock threshold + in-app alert badge
- CSV import/export of catalogue (reuse MGX's existing tuckshop catalogue format)

**Staff & access**
- Owner account (email/password via Supabase Auth; Google OAuth optional)
- Staff profiles with 4-digit PIN, fast switch on the sell screen
- Roles: Cashier / Supervisor / Admin (three-tier, as in Shop-Flow)
- Permission gates: refunds, discounts over X%, stock adjustments, reports, settings
- Every sale/refund/discount/adjustment stamped with staff ID

**History & reports**
- History tab: searchable sales list; per-sale detail; refund (full/partial) with reason; resend receipt; edit note
- Reports tab: today/week/month/custom range; totals by tender type, by product, by category, by staff; gross margin (using cost price); Z-report / end-of-day cash-up per currency

**Offline-first & sync**
- Local SQLite is the source of truth; app is 100% functional with zero connectivity indefinitely
- Background sync to Supabase when online; visible sync status (pending count, last synced)
- Multi-device: catalogue and sales sync across devices on one merchant account

**Settings**
- Business profile (name, logo, receipt footer, phone), currency rate, tender toggles, low-stock defaults, receipt template

### 5.2 v1.1 (Phase B)
Tips; split bills; customer records + credit book ("chikwereti" ledger); barcode scanning via camera; Bluetooth thermal printing (port from Shop-Flow: service `000018f0`, characteristic `00002af1`); staff performance leaderboard; PDF report export.

### 5.3 v2+ (Phase C)
Web back-office dashboard (owner's "Yoco App" equivalent); multi-location; payment integrations (EcoCash merchant API, bank POS linkage — adapter pattern, mirroring how developer.yoco.com exposes payments separately from POS); iOS build; recipe-level inventory.

### 5.4 Explicitly out of scope
Payment processing in MVP (all tenders are record-keeping); accounting integrations; loyalty; e-commerce sync; hardware sales.

## 6. Functional requirements (numbered, testable)

- FR-01: A cashier shall complete a sale (select items → tender → done) in ≤ 6 taps for a single-item cash sale.
- FR-02: All create/update operations shall succeed with no network and persist across app restarts.
- FR-03: Stock quantity shall decrement atomically at sale completion; a failed sale shall not deduct stock.
- FR-04: A refund shall require Supervisor+ role, capture a reason, restore stock (optional toggle per refund), and appear in reports as negative revenue.
- FR-05: The ZWG price shall be derived as `round(usd_price × rate, nearest 0.05)` using the merchant's active rate; the rate used shall be stored on every sale line.
- FR-06: Sync shall be idempotent — replaying the same queued operation shall never duplicate a sale.
- FR-07: When two devices edit the same product offline, last-write-wins by server receive time; stock adjustments merge additively (never overwrite quantity).
- FR-08: Staff PIN entry shall lock for 60s after 5 failed attempts.
- FR-09: End-of-day report shall reconcile: opening float + cash sales − cash refunds = expected cash, per currency.
- FR-10: Receipts shall render the business logo, line items, tender breakdown, staff name, and a unique human-readable sale number (e.g. `TNG-2026-000123`).

## 7. Non-functional requirements

- **Performance:** cold start ≤ 3s on 2 GB RAM device; catalogue of 1,000 SKUs scrolls at 60fps; sale commit ≤ 150 ms locally.
- **Offline:** indefinite offline operation; sync queue capped only by device storage.
- **Data:** local DB survives app updates; export-everything escape hatch (CSV/JSON) so merchants are never locked in.
- **Security:** see `05_SECURITY_PLAYSTORE.md`. Row Level Security on all cloud tables; PINs hashed; no card data ever stored.
- **Battery/data:** sync payloads batched and compressed; no polling — sync on connectivity change, app foreground, and manual pull.
- **Localisation:** English at launch; strings externalised for Shona/Ndebele later.
- **Accessibility:** minimum 48dp touch targets, high-contrast mode, works in bright sunlight (high-contrast light theme default).

## 8. Assumptions & risks

| Risk | Mitigation |
|---|---|
| Supabase free tier limits (500 MB) | Sales rows are small; archive-to-CSV job at 400 MB; monitor monthly |
| ZWG rate volatility mid-day | Rate is merchant-set and versioned; each sale stores the rate used |
| Device loss = data loss before sync | Encourage daily sync; owner can restore full state to a new device from cloud |
| Play Store policy (financial app classification) | App processes no payments — it is record-keeping; declare accurately in Data Safety form |

## 9. Open questions (resolve in review)

1. Is this Tengesa itself, or a parallel product? (Recommendation: it IS Tengesa — adopt this PRD as its spec.)
2. Credit book in MVP or v1.1? (Currently v1.1; strong pull from tuckshop persona — revisit after pilot feedback.)
3. Should ZWG be a first-class stored price or always derived? (Currently derived; simpler, avoids stale prices.)
4. Merchant onboarding: self-serve only, or assisted setup (MGX Media service angle)?

## 10. Review & refinement process

- **R1 (self-review):** walk each persona through the MVP feature list; kill anything no persona needs.
- **R2 (Claude Code review):** run the qa-reviewer subagent against this PRD for ambiguity/testability of FRs.
- **R3 (field review):** show the sell-flow prototype to 2–3 real traders; log objections; update §5 and re-version to 1.1.
- Change control: any scope addition must name the persona it serves and the phase it lands in.
