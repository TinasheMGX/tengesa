# Security Checklist & Play Store Readiness — Tengesa POS

## Part A — Security checklist (OWASP MASVS-informed, right-sized)

### A1. Authentication & access
- [ ] Owner auth via Supabase Auth only; enforce min 8-char passwords; enable leaked-password protection in Supabase settings
- [ ] Staff PINs: **hashed** (bcrypt/argon2 via JS lib) — never plaintext, never synced as plaintext
- [ ] PIN lockout: 60s after 5 failures (FR-08); auto-lock screen after configurable idle (default 5 min)
- [ ] Role gates enforced in the **repository layer**, not just the UI (a hidden button is not security)
- [ ] Supabase session tokens stored via Capacitor Secure Storage / Android Keystore, not localStorage
- [ ] Sign-out wipes session tokens; "remove device" available from owner account (revokes refresh token)

### A2. Data protection
- [ ] SQLite encryption enabled (`@capacitor-community/sqlite` secret + Keystore-held key)
- [ ] Android `allowBackup=false` (prevent adb backup extraction); rely on cloud sync for recovery instead
- [ ] No card numbers, no wallet PINs, no customer payment credentials stored — tenders are labels + amounts only (keeps us out of PCI scope entirely)
- [ ] Product images stripped of EXIF on upload
- [ ] Export files (CSV) written to app-scoped storage and shared via share-sheet only

### A3. Cloud (Supabase)
- [ ] **RLS enabled on every table** — verify with `select * from pg_tables where rowsecurity = false` (must return none in public schema)
- [ ] RLS policies tested per role with Supabase's policy tests + run `Supabase:get_advisors` security lints before each release
- [ ] Service-role key exists **only** inside Edge Functions env — never in the app bundle
- [ ] Anon key in app is fine (it's designed for that) but paired with RLS as above
- [ ] Storage buckets private; images served via signed URLs or RLS'd public-read on `product-images` only
- [ ] Edge Function `/sync` validates: auth JWT → merchant_id → every op's merchant_id matches (no cross-tenant writes)
- [ ] Rate limit sync endpoint (Supabase built-in + per-device throttle)

### A4. App & code hygiene
- [ ] HTTPS only; Capacitor `androidScheme: 'https'`; no cleartext traffic (`usesCleartextTraffic=false`)
- [ ] No secrets in the repo — `.env` gitignored; scan with gitleaks in CI
- [ ] Dependencies audited (`npm audit` in CI); lockfile committed; `--legacy-peer-deps` documented
- [ ] ProGuard/R8 minification on release builds
- [ ] Sentry configured to scrub PII (default scrubbers + no logging of payloads)
- [ ] Audit log: refunds, discounts > threshold, stock adjustments, staff changes, rate changes — immutable, synced

### A5. Business-logic integrity
- [ ] Money as integer cents everywhere; unit tests on rounding, change calc, cross-currency change, discount math (mandatory before release)
- [ ] Sale commit is a single SQLite transaction — verified by kill-app-mid-sale test
- [ ] Idempotency keys on all sync ops — replay test in CI
- [ ] Clock skew tolerated: server timestamps are authoritative for ordering; device time only for display

---

## Part B — Google Play Store readiness

### B1. Account & identity
- [ ] Google Play Console developer account — US$25 one-time
- [ ] Individual accounts now require identity verification + (since 2024) **20 testers for 14 days of closed testing** before production access for new personal accounts — plan the tester group early (staff, family, pilot merchants). *Verify current requirement at launch time — Play policies change.*
- [ ] App identity: package `com.mgx.tengesa` (immutable — decide once), app name, 512px icon, feature graphic 1024×500

### B2. Build & signing
- [ ] Release as **AAB** (App Bundle), not APK
- [ ] Play App Signing enrolled (Google holds signing key; you keep upload key in a password manager + offline backup)
- [ ] `targetSdkVersion` at Play's current requirement (API 35 as of 2025 — check console warning banner)
- [ ] versionCode strategy: `YYMMDDNN`; CI bumps automatically
- [ ] Signed build reproducible from CI (GitHub Actions with secrets for upload keystore)

### B3. Policy & compliance
- [ ] **Data Safety form** — accurate: collects email (account), business data (products, sales); encrypted in transit; user can request deletion. No ads, no data sold, no location, no financial *payment* data processed
- [ ] **Privacy policy** — hosted free (GitHub Pages), linked in listing and in-app (More → Legal). Must name data collected, purpose, retention, deletion contact
- [ ] **Account deletion** — Play requires an in-app + web path to delete account and data (Supabase edge function: cascade delete merchant). Non-negotiable since 2024
- [ ] Not a "financial services" app for policy purposes — it records sales, processes no payments; describe it as business/productivity POS record-keeping in the declaration
- [ ] Content rating questionnaire (IARC) — will come out "Everyone"
- [ ] Permissions minimal: INTERNET, CAMERA (barcode, v1.1 — request at time of use), BLUETOOTH_CONNECT (printing, v1.1). No location, no contacts

### B4. Quality bar (pre-submission)
- [ ] Tested on a real 2 GB RAM device + Android 10 minimum + one tablet
- [ ] Offline cold-start test: airplane mode from install → full day of sales → sync — zero data loss
- [ ] Monkey test 30 min, no crash; crash-free rate ≥ 99.5% in closed testing
- [ ] Screenshots: 4–8 phone (1080×1920+) + 2 tablet; captions sell the offline-first + dual-currency story
- [ ] Store listing copy: title ≤ 30 chars ("Tengesa POS — Sell & Stock"), short description ≤ 80 chars, keyword-aware long description (POS Zimbabwe, tuckshop, stock, offline, EcoCash record)

### B5. Launch sequence
1. Internal testing track (you + 2 devices) → fix
2. Closed testing (20 testers × 14 days — satisfies new-account requirement AND doubles as merchant pilot)
3. Pre-launch report review in Console (auto device-lab run)
4. Production, staged rollout 20% → 100%
5. Post-launch: monitor ANRs/crashes in Android Vitals weekly; Sentry alerts

### B6. iOS (later workstream — parked)
Capacitor makes this a build target, not a rewrite. Costs to plan for: Apple Developer US$99/yr, a Mac or cloud build (Codemagic free tier — already researched), App Store review's stricter data rules. Do not spend on this until Android proves demand.
