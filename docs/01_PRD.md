# Product Requirements Document — Tengesa POS
**Version:** 2.1 · **Owner:** MGX · **Status:** Gate 1 approved · **Phase:** 1 of 8

| Changelog | |
|---|---|
| v2.1 (Jul 2026) | Gate 1 decisions applied: Option B release train adopted; credit book (FR-E2) deferred; customer portal scoped to receipts + credit statements; Tengesa identity confirmed (Shop-Flow/Boss Kuku superseded); VAT (FR-C9) deferred, rate corrected to 15.5%; multi-store schema day 1, feature R2 confirmed. §13, §17, §18 updated. QA-reviewer audit: 5 BLOCKERs fixed (permission matrix, performance baselines, LWW spec, cash-up dual-currency formula); 8 WARNINGs tightened |
| v2.0 (Jul 2026) | Full restructure to programme brief: added Mission, Business Goals, User Stories, Acceptance Criteria, Competitive Analysis, MoSCoW, platform options analysis; expanded FR/NFR coverage to all schema entities |
| v1.0 (Jul 2026) | Initial PRD |

---

## 1. Vision

Every small trader in Zimbabwe — tuckshop, boutique, salon, takeaway, kiosk — runs their business from a beautiful, fast POS in their pocket that works all day with no internet, no monthly fee, and no confusion between USD and ZWG. When connectivity appears, everything syncs. The owner always knows what sold, what's in stock, and where the money is.

## 2. Mission

Build and ship a production-grade, offline-first Android POS and stock management system that matches the simplicity and polish of Yoco Counter, is free at the core forever, and is engineered — modular, secure, maintainable — to grow from a solo tuckshop MVP into a commercial-grade multi-store platform without rewrites.

## 3. Problem statement

Zimbabwean SMEs transact in two currencies at a moving rate, in a cash-dominant economy, over unreliable connectivity and power. They track sales in exercise books, guess their stock, and cannot separate takings by staff member or tender. Global POS products fail them structurally:

- **Payment-led products** (Square, SumUp, Yoco) are unavailable in Zimbabwe or built around card processing that doesn't exist here.
- **The one available free option** (Loyverse) charges monthly for the features this market needs most — advanced inventory, employee management, even unlimited sales history — and has no concept of dual-currency USD/ZWG trading.
- **Everything else** (Lightspeed, Shopify POS, Toast) carries subscription pricing and connectivity assumptions incompatible with this market.

The result: no product serves the Zimbabwean small trader's actual day. Tengesa does.

## 4. Target users

**Primary:** owner-operated Zimbabwean micro/small retailers and service businesses, 1–5 staff, 20–1,000 SKUs, single location, trading in USD and ZWG, on low/mid-range Android phones (2–4 GB RAM, Android 10+) or cheap tablets.
**Secondary:** multi-till single stores; owners monitoring remotely; (roadmap) small chains of 2–5 stores.
**Tertiary (roadmap):** the merchant's customers (receipts lookup, credit statements via customer portal).

## 5. User personas

| Persona | Profile | Goals | Pains today |
|---|---|---|---|
| **Mai Tino — tuckshop owner-operator** | 42, Mbare; ~150 SKUs; phone only; sells in both currencies all day | Fast checkout; end-of-day cash position per currency; know what to restock | Exercise book; rate confusion at the till; stock "disappears" |
| **Kuda — boutique owner** | 29, Avondale; variants (size/colour); 2 staff | Trust staff without being present; control refunds/discounts; margin per item | No visibility of who sold/refunded what; theft risk |
| **Tatenda — takeaway operator** | 35, CBD; lunch rush; combos & modifiers | Speed; saved orders; best-seller insight | Queue abandonment; order errors |
| **Rufaro — remote owner** | 38, runs a salon she visits twice a week | See today's numbers from anywhere; weekly reports | Calls staff for figures; no audit trail |
| **Tino — cashier (staff user)** | 22, employed at Kuda's boutique | Get through a sale without mistakes; not be blamed unfairly | Manual price lookups; disputes over till variances |

## 6. Business goals

1. **Prove the product**: 10-merchant pilot in Harare within 60 days of R1; 50 active merchants in 6 months; 500 in year 1.
2. **Zero operating cost until revenue**: infrastructure stays on free tiers; only spend is the $25 Play account.
3. **Monetisation hypothesis (post-traction, not MVP)**: free core forever (sell, stock, staff, reports — deliberately including what Loyverse charges for, as the wedge); later paid tiers for multi-store, advanced analytics/web dashboard extras, and white-label. Validate willingness-to-pay during pilot; target first revenue in year 2.
4. **Strategic asset for MGX**: a portfolio-grade product demonstrating full-stack + AI-assisted delivery; potential MGX Media service line (assisted onboarding, catalogue setup) mirroring Yoco's "we build your menu" model.
5. **Own the category term**: "Tengesa" (Shona: *sell*) as the default answer to "what POS do Zim tuckshops use?"

## 7. Competitive analysis

| Product | Price model | Offline | Availability (ZW) | Strengths | Gaps Tengesa exploits |
|---|---|---|---|---|---|
| **Yoco POS/Counter** (inspiration) | Free software; hardware from R2,999; no monthly software fee | Partial (SIM-backed payments) | ✗ SA only | Gold-standard UX: Sell/History/Reports/More, staff PINs, tile catalogue, item stock | Not in ZW; card-processing-centric; single currency |
| **Loyverse** (direct competitor) | Free core; paid add-ons: Advanced Inventory ~$25–29/mo/store, Employee Mgmt ~$5/employee/mo, unlimited sales history paid | Yes for sales/shifts; back office cloud-only | ✓ (170 countries) | Mature, polished, proven with small traders | Charges for inventory + staff mgmt (our free wedge); no USD/ZWG dual-currency; no credit book; cloud back office |
| **Square POS** | Free plan + 2.6% + 15¢ in-person processing (2025 rates); Plus $49/mo, Premium $149/mo; Retail Plus $89/mo/location | Limited offline payments window with merchant risk | ✗ | Ecosystem breadth, free tier depth | Unavailable in ZW; processing-led model irrelevant here |
| **Shopify POS** | Requires Shopify subscription (~$29–39+/mo); POS Pro extra per location | Limited | Nominal | E-commerce integration | Subscription cost; e-com-first; overkill |
| **Lightspeed** | ~$69–89+/mo | Limited | ✗ practical | Deep retail features | Enterprise pricing class |
| **Toast** | Hardware + SaaS contracts, US restaurants | Good (restaurant-grade) | ✗ | Restaurant depth, offline resilience patterns worth studying | Wrong market entirely |
| **SumUp** | Free app + card readers; payments-led | Partial | ✗ | Simple onboarding patterns | Unavailable; payments-led |
| **Status quo: exercise book** | Free | Perfect | ✓ | Zero learning curve | No totals, no stock, no audit, no reports |

**Positioning statement:** *For Zimbabwean small traders, Tengesa is the free POS that works all day offline and speaks both currencies — giving away what Loyverse charges for, with the polish of Yoco, built for how Zimbabwe actually trades.*

## 8. Functional requirements

Grouped by module; IDs are stable and referenced by user stories, tests and MoSCoW. (M) = MVP.

**FR-A Authentication & staff**
- FR-A1 (M) Owner registers/signs in (email+password; Google OAuth optional); full offline trial mode before account creation (no feature limits, no time limit; trial data migrates into created account).
- FR-A2 (M) Staff profiles with name, 4-digit PIN, role ∈ {cashier, supervisor, admin}; fast on-screen switching.
- FR-A3 (M) Permission matrix enforced in the data layer. Default: cashier — sell, view own sales, view stock; supervisor — adds refunds, discounts > threshold % (default 10%, merchant-configurable), stock adjustments, void sale; admin — adds price edit, reports access, staff management, settings, rate changes. Merchant may customise per-role.
- FR-A4 (M) PIN lockout 60s after 5 failures; auto-lock after configurable idle.
- FR-A5 Biometric unlock for owner/admin (device-supported). *(Should)*

**FR-B Catalogue**
- FR-B1 (M) Products: name, image, category, brand, cost price, selling price (USD cents), barcode, SKU, active flag; soft delete.
- FR-B2 (M) Categories with ordering; variants (e.g. size/colour) with price deltas; modifiers *(Should)*.
- FR-B3 (M) CSV import/export (column-mapped); bulk price update *(Should)*.
- FR-B4 Barcode scan via camera to add/sell. *(Should)*

**FR-C Selling**
- FR-C1 (M) Sell screen with tile catalogue + category chips AND quick-amount keypad (Yoco dual-input pattern).
- FR-C2 (M) Cart: quantities, line & bill discounts (value or %), sales note.
- FR-C3 (M) Tender types as record-keeping toggles: cash_usd, cash_zwg, ecocash, zipit, card, other.
- FR-C4 (M) Dual currency: prices stored USD cents; ZWG derived from merchant-set active rate, rounded to nearest 0.05 ZWG (commercial rounding — 0.025 rounds up). A single tender in any one currency; change calculated in tendered currency by default, cashier may toggle (mixed-currency tenders within one sale require FR-C8, Could). Every sale stores `fx_rate_used` = active rate at moment of completion, regardless of tender currency. Rounding differences absorbed by merchant.
- FR-C5 (M) Atomic completion: sale + lines + payment(s) + stock movements + outbox in one local transaction. Local commit ≤150 ms on reference device (2 GB RAM, Android 10, 1,000 SKUs, 10,000 historical sales). One-item cash sale from sell screen to completion: ≤6 taps (tile → tender → done).
- FR-C6 (M) Saved/open bills (park & resume, named). No hard limit on concurrent parked bills; survive end-of-day and app restart; device-local until completed (then synced as a sale).
- FR-C7 (M) Receipt: branded, unique human number, shareable as image via Android share sheet (WhatsApp/SMS).
- FR-C8 Split tender across methods; tips. *(Could — v1.1)*
- FR-C9 Simple tax line (e.g. VAT 15.5% toggle per merchant/product) shown on receipt. *(Should — deferred post-R1, Gate 1)*

**FR-D Stock & purchasing**
- FR-D1 (M) Stock as append-only movement log (opening, sale, refund-restock, received, damaged, count-correction); on-hand always derived.
- FR-D2 (M) Low-stock threshold per product; alert badge + list.
- FR-D3 Suppliers directory; purchase/GRV capture updating stock and cost. *(Could — v1.1)*
- FR-D4 Stock-take mode (guided count → corrections). *(Should)*

**FR-E Customers & credit**
- FR-E1 Customer records (name, phone); attach to sale. *(Should)*
- FR-E2 Credit book ("chikwereti"): sell on account, record repayments, balance & statement per customer. *(Won't — R1; deferred to future release, Gate 1)*

**FR-F History & refunds**
- FR-F1 (M) Searchable/filterable sales history; sale detail with timeline (created → synced → refunded).
- FR-F2 (M) Refunds full/partial; supervisor+ gate; mandatory reason; optional restock; immutable link to original; reflected as negative revenue.
- FR-F3 (M) Resend/reshare receipt; edit sale note.

**FR-G Reports & cash management**
- FR-G1 (M) Periods: today/week/month/custom. Totals, count, average sale, gross margin; breakdowns by tender, product, category, staff.
- FR-G2 (M) End-of-day cash-up runs independently per physical cash pool (USD and ZWG). Formula per pool: opening float + cash-in (sales tendered in that currency + paid-in) − cash-out (refunds tendered in that currency + paid-out) = expected. Cross-currency tenders affect only the pool of the currency actually received. Each pool records counted vs expected with mandatory variance note. Z-report covers both pools with totals by tender, staff and category.
- FR-G3 (M) All reports computable fully offline from local data.
- FR-G4 Cash drawer events (paid-in/paid-out with reasons) feeding G2. *(Should)*

**FR-H Offline & sync**
- FR-H1 (M) 100% functionality with zero connectivity, indefinitely; local DB survives updates/restarts.
- FR-H2 (M) Sync triggers: (1) on network connectivity gained (debounced 5 s), (2) on app foreground if >60 s since last sync, (3) manual pull-to-refresh. Backoff after 3 consecutive failures. Batched, idempotent (replay-safe); visible status (pending count, last synced).
- FR-H3 (M) Multi-device on one merchant: catalogue/settings converge via LWW using server-received timestamp (not device clock). Losing write preserved in audit log with winning value, losing value, both timestamps, and both device IDs. Sales append-only (never merged). Stock movements merge additively (SUM of all qty_delta from all devices).
- FR-H4 (M) Device registration; restore full state to a new device from cloud.
- FR-H5 Multi-store: store entity, per-store stock and reporting. *(Won't — MVP; R2 roadmap; schema allows from day 1)*

**FR-I Administration & audit**
- FR-I1 (M) Business settings: profile/logo, receipt footer, ZWG rate (versioned — each change creates a timestamped audit entry with old/new values, staff who changed it; rates apply to future sales only, never backdated), tender toggles, thresholds.
- FR-I2 (M) Immutable audit log: refunds, discounts>threshold, price changes, stock adjustments, rate changes, staff changes — synced.
- FR-I3 (M) Data export escape hatch (sales + catalogue CSV/JSON) — no lock-in.
- FR-I4 (M) Account & data deletion path (in-app + web) per Play policy.

## 9. Non-functional requirements

| # | Requirement | Target |
|---|---|---|
| NFR-1 Performance | Cold start (process killed → sell screen interactive, measured via `reportFullyDrawn()`) ≤3 s on reference device (2 GB RAM, Android 10, eMMC storage, 1,000 SKUs); catalogue scroll at 60 fps; local sale commit ≤150 ms on same reference device |
| NFR-2 Offline | Indefinite; sync queue bounded only by storage; zero loss of committed data on power cut (any committed SQLite transaction is durable; in-flight cart state persisted to local storage on each modification for recovery, but an uncommitted cart is not counted as data loss) |
| NFR-3 Reliability | Crash-free sessions ≥99.5%; atomicity proven by kill-mid-write tests |
| NFR-4 Security | Encrypted local DB; hashed PINs; RLS on all cloud tables; OWASP MASVS-L1 baseline (Phase 6) |
| NFR-5 Scalability | Architecture serves 10k merchants on one Postgres before sharding decisions; per-merchant data isolation by design |
| NFR-6 Maintainability | Clean/hexagonal layering (Phase 4); feature modules; UI never touches cloud SDK directly; docs updated with behaviour changes |
| NFR-7 Modularity | Payments, printing, portal are adapter seams — pluggable later without core changes |
| NFR-8 Usability | ≤10 min from install to first sale, unaided; sunlight-legible; 48 dp targets; system font scaling to 130% |
| NFR-9 Cost | $0/month infra at ≤500 merchants; documented headroom plan |
| NFR-10 Data & battery | No polling; batched compressed sync; product images ≤100 KB |
| NFR-11 Localisation-ready | Strings externalised (English at launch; Shona/Ndebele later) |

## 10. User stories (by epic)

Format: *As a ⟨persona⟩ I want ⟨capability⟩ so that ⟨outcome⟩.* Full Gherkin acceptance criteria for the six highest-risk stories in §11; every remaining story receives AC in its phase-build spec before implementation.

**Epic E1 Onboarding** — US-1.1 As a new owner I want to start selling in a guided setup within 10 minutes so that I don't abandon the app. · US-1.2 As a cautious owner I want a full offline trial without creating an account so that I can judge it first. · US-1.3 As Mai Tino I want to import my existing product CSV so that I don't retype 150 items.

**Epic E2 Selling** — US-2.1 As Tino (cashier) I want to tap tiles and charge so that the queue keeps moving. · US-2.2 As Tino I want the ZWG price computed from today's rate so that I never do mental maths at the till. · US-2.3 As Tatenda I want to park a bill by name and resume it so that phone orders don't block walk-ins. · US-2.4 As Tino I want change calculated even when the customer pays USD for a ZWG-priced basket so that disputes stop. · US-2.5 As a customer I want a WhatsApp receipt so that I have proof of purchase.

**Epic E3 Catalogue & stock** — US-3.1 As Kuda I want size/colour variants so that one product covers all its versions. · US-3.2 As Kuda I want low-stock alerts so that I reorder before selling out. · US-3.3 As Kuda I want every stock change logged with who/why so that shrinkage is visible. · US-3.4 As Mai Tino I want a guided stock-take so that my counts become corrections, not chaos.

**Epic E4 Staff & control** — US-4.1 As Kuda I want PIN profiles with roles so that staff sell without accessing my settings. · US-4.2 As Kuda I want refunds gated to supervisors with reasons so that "refund fraud" dies. · US-4.3 As Tino I want my sales attributed to me so that till variance isn't pinned on me unfairly.

**Epic E5 History & refunds** — US-5.1 As Rufaro I want to search any sale and see its full story so that I can resolve disputes. · US-5.2 As a supervisor I want partial refunds with optional restock so that exchanges are honest in the books.

**Epic E6 Reports & cash** — US-6.1 As Rufaro I want today's numbers by tender/staff/product so that I manage without being present. · US-6.2 As Mai Tino I want end-of-day expected-vs-counted per currency so that I know my drawer is right. · US-6.3 As Kuda I want gross margin using cost prices so that I price properly.

**Epic E7 Offline & sync** — US-7.1 As Mai Tino I want everything to work in a total blackout so that load shedding never stops trading. · US-7.2 As Rufaro I want two devices to converge after both traded offline so that the books stay one truth. · US-7.3 As an owner I want a stolen phone replaced by restoring to a new device so that my business survives the loss.

**Epic E8 Customers & credit** *(Should)* — US-8.1 As Mai Tino I want to record sales on credit per customer and their repayments so that the chikwereti book lives in the app.

## 11. Acceptance criteria — six critical stories (Gherkin)

**AC for US-2.4 (cross-currency change)**
```gherkin
Given the active rate is 30.00 ZWG per USD
And the cart totals US$10.00
When the customer tenders 400 ZWG (cash_zwg)
Then the app records payment amount_zwg 400 at rate 30.00
And displays change due of 100 ZWG (or US$3.33 if cashier toggles currency)
And the stored sale carries fx_rate_used = 30.00 permanently
```

**AC for US-7.1 (offline day)**
```gherkin
Given the device is in airplane mode from app cold start
When 50 sales, 2 refunds and 3 stock adjustments are performed
And the app is force-killed and reopened twice during the day
Then all records persist with zero loss
And the sync badge shows 55 pending operations
And no feature (sell, stock, reports, cash-up) shows a network error
```

**AC for US-7.2 (two-device convergence)**
```gherkin
Given devices A and B share one merchant account and are offline
When A sells 3 units of P1 and edits P1 price to $2.50
And B sells 2 units of P1 and edits P1 price to $2.40
And both come online in any order
Then total P1 stock deduction is exactly 5 (additive movements)
And P1 price equals the edit the server received last, with the loser in the audit log
And both devices display identical history, stock and reports after sync
```

**AC for US-4.2 (refund gate)**
```gherkin
Given Tino is signed in as cashier
When Tino taps Refund on a completed sale
Then a supervisor PIN prompt appears and cashier PINs are rejected
And on supervisor PIN success a reason must be selected before Confirm enables
And the refund record stores the supervisor's staff_id, not Tino's
```

**AC for US-3.4 (count correction)**
```gherkin
Given product P has movements summing to on-hand 18
When a stock-take records a counted quantity of 15
Then one movement of type count_correction with qty_delta −3 is appended
And on-hand derives to 15 on every device after sync regardless of arrival order
```

**AC for US-6.2 (cash-up)**
```gherkin
Given opening float US$20.00 and cash_usd sales of $145.50 and a cash_usd refund of $5.00 and a paid-out of $10.00
When the cashier runs end-of-day for USD
Then expected shows $150.50
And entering counted $148.50 records variance −$2.00 with a required note
And the Z-report image includes totals by tender, staff and category for the business date
```
*Symmetric scenario applies for ZWG pool. Cross-currency tender (e.g. ZWG payment for USD-priced sale) affects only the ZWG cash pool.*

## 12. Success metrics

| Metric | Target |
|---|---|
| Activation: install → first real sale | ≥60% within 24 h |
| Time-to-first-sale (unaided) | ≤10 min median |
| Offline integrity: offline-captured sales syncing losslessly | ≥99.9% |
| 3-item cash sale duration | ≤10 s |
| Retention: merchants active week 4 | ≥50% of activated |
| Crash-free sessions | ≥99.5% |
| Pilot NPS (10 merchants) | ≥40 |
| Infra cost | $0/mo through 500 merchants |

## 13. MVP scope & platform options analysis

The brief lists Android phones, tablets, web dashboard, desktop, customer portal and merchant analytics for the first production release. Options, honestly costed for one part-time builder on ~$0:

| Option | Contents of "Release 1" | Est. duration | Risk |
|---|---|---|---|
| **A — Everything at once** | All six targets | 9–14 months to *first* ship | High: no user feedback for a year; portal has no defined job; motivation/decay risk |
| **B — Release train (ADOPTED — Gate 1)** | R1 = Android phone+tablet POS to Play Store (3–4 months); R1.5 = web dashboard + merchant analytics, read-only→manage (+4–6 wks); R2 = desktop as installable PWA/Tauri wrap of the web dashboard (+1–2 wks); R3 = customer portal (own mini-PRD first); R4 = iOS | First ship in 3–4 months; full brief in ~7–9 months | Low: feedback compounds; each train reuses the last (shared TS core + design system) |
| **C — Ultra-lean pilot** | Phone-only APK sideloaded to 5 merchants | 6–8 weeks | Fastest learning; defers Play/tablet work that must happen anyway |

**Decision (Gate 1): B adopted**, with C's spirit inside it (closed-testing track = pilot). Trade-off accepted: web/desktop/portal users wait one train; in exchange R1 quality and learning velocity are protected. "Merchant analytics" is delivered twice: in-app Reports (R1, offline) and richer web analytics (R1.5).

**MVP (R1) =** every (M) requirement in §8 on Android phone + tablet, published via closed → production tracks.

## 14. Future roadmap

- **R1.5** Web dashboard (owner back-office: catalogue mgmt, reports/analytics, staff admin) — reuses repository layer + design tokens; Supabase reads.
- **R2** Desktop (PWA install or Tauri wrap — near-zero incremental); Bluetooth thermal printing (Shop-Flow port); barcode hardware support; suppliers/purchases; split tender & tips; taxes hardened.
- **R3** Customer portal (receipt lookup, credit statements — scoped by its own mini-PRD); multi-store; Shona/Ndebele.
- **R4** iOS (Capacitor target; Apple $99/yr + Codemagic cloud build — spend gated on revenue); payment integrations via adapter (EcoCash merchant API, bank POS linkage) mirroring how Yoco separates its payments API from POS.

## 15. Risks & mitigations

| Risk | L×I | Mitigation |
|---|---|---|
| Scope creep from six-platform ambition | H×H | §13 Option B gates; MoSCoW is contract; additions name persona + train |
| Sync bugs corrupt trust in the books | M×H | Phase 5 formal design; append-only money; convergence tests in CI; sync-specialist subagent review |
| Free-tier ceilings | L×M | ~1.5 KB/sale ⇒ years of headroom; monthly monitoring; archive job; first paid step ($25/mo Supabase Pro) is revenue-gated |
| ZWG regime change (rate/redenomination) | M×M | Rates versioned & stamped per sale; currency table not hardcoded |
| Play Console new-account friction (identity + 20-tester/14-day closed test) | M×M | Closed test = pilot; recruit testers before code-complete |
| Loyverse localises first | L×M | Ship wedge fast; credit book + dual-currency are structural, not cosmetic |
| Solo-builder burnout | M×H | Phase gates create finish lines; Claude Code leverage; calm cadence per MGX operating style |

## 16. Assumptions & out-of-scope

**Assumptions:** merchants own Android ≥10 devices; WhatsApp is the receipt channel; USD pricing with ZWG at merchant-set rate remains the trading norm; pilot merchants recruitable through MGX's personal/MGX Media network; MGX remains sole builder with Claude Code.

**Out of scope (all releases until explicitly promoted):** payment *processing* of any kind (tenders are records); PCI-scope data; hardware manufacturing/resale; accounting integrations; e-commerce storefronts; loyalty points; franchise/enterprise features; recipe-level (ingredient) inventory.

## 17. Gate 1 decisions (resolved 2026-07-04)

1. **Release train Option B adopted.** R1 = Android phone+tablet → R1.5 web dashboard → R2 desktop → R3 customer portal → R4 iOS.
2. **Credit book (FR-E2) deferred.** Stays Should; not in R1. Will be implemented in a future release.
3. **Customer portal (R3) scoped:** receipts lookup + credit statements (when credit ships). Token-authenticated links, no customer login. Full mini-PRD before R3 begins.
4. **Product identity confirmed.** This PRD is the Tengesa spec. Shop-Flow and Boss Kuku scope formally superseded; v1 archive retained as source material only.
5. **VAT/tax line (FR-C9) deferred.** Stays Should, post-R1. Zimbabwe VAT rate is 15.5%. Most pilot merchants are unregistered; building it now risks misleading receipts.
6. **Multi-store: schema day 1, feature R2.** `store_id` on key tables from Phase 4; R1 UI is single-store only. Avoids painful retroactive migration.

## 18. Feature prioritisation — MoSCoW (R1 contract)

| Priority | Features |
|---|---|
| **Must** | Owner auth + offline trial; staff PINs/roles/permission gates; audit log; products/categories/variants; CSV import/export; tile+keypad sell; cart with discounts & notes; six record-keeping tenders; dual-currency USD/ZWG with versioned rate & cross-currency change; atomic sale commit; saved bills; shareable receipts; movement-log stock; low-stock alerts; history + full/partial gated refunds; reports (periods, tender/product/category/staff, margin); end-of-day cash-up + Z-report; offline-first local DB; idempotent batched sync; multi-device convergence; device registration & restore; settings; data export; account deletion |
| **Should** | Customers (FR-E1); camera barcode scan; stock-take mode; cash drawer paid-in/out events; bulk price update; biometric owner unlock; modifiers |
| **Could** | Split tender; tips; suppliers & purchases/GRV; product images bulk tools; Shona UI; receipt printer (BT) if pilot demands early |
| **Won't (R1)** | Payment processing/integrations; web dashboard & desktop (R1.5/R2); customer portal (R3); multi-store UI (R2+; schema ready); iOS (R4); loyalty; accounting sync; recipe inventory; credit book (FR-E2, future release); VAT toggle (FR-C9, post-R1) |

---

**Gate 1 status:** ✓ Approved (2026-07-04). All §17 questions resolved. PRD v2.1 is the binding R1 contract. Phase 2 (UX & Application Flow) may begin.
