# Claude Code Brief — Rebuild a cleaner POS app for Zimbabwean SMEs (multi-agent, autonomous)

> Paste this whole file into Claude Code as your task, or save it in the repo and run:
> "Read `tengesa-rebuild-prompt.md` and execute it end to end without stopping."

---

## 1. Mission

I have an existing point-of-sale app (Shopley). Its UI feels too **complex and cluttered**. Your job is to produce the **best, simplest, fastest online and offline version** of this app — same capabilities, far cleaner UI/UX — by setting up **3 specialist subagents** and **3 skills** that audit the current app, research best-in-class POS UX, agree on a design, and **implement it**.

The work must run **autonomously to completion** (no stopping to ask me questions — make reasonable decisions and log them).

---

## 2. Operating mode (read this carefully)

- **Do not interrupt me with questions.** Where something is ambiguous, choose the most sensible option, write it in `docs/DECISIONS.md`, and keep going.
- Work the full loop: audit → research → design → build → test → fix → finalize.
- Use your subagents and skills throughout; don't shortcut them.
- Only stop when the **Definition of Done** (section 12) is fully met.

---

## 3. Hard constraints (non-negotiable)

1. **Never modify, move, or delete the existing app.** Treat it as read-only reference. Inspect it freely; change nothing in it.
2. **Build in a brand-new sibling folder**, e.g. `C:/Users/ssd/tengesa` (next to the existing project at `C:/Users/ssd/shop-flow`). Confirm the real existing path from disk; do not assume.
3. **Supabase safety:** read the repo's `.env` files to learn the **test** and **live/production** Supabase project refs. Use the **test** project for all development. **Never run destructive migrations or `truncate/drop` against the live/production project.** If a new dev project is cleaner, create one and document it — but do not touch live data.
4. **Feature parity is mandatory.** "Simpler" means simpler *information architecture, layout, and flows* — **not** fewer features. Nothing functional may be dropped.
5. npm installs in this stack require `--legacy-peer-deps`.
6. New repo only — initialize fresh git, `.gitignore`, one clean initial commit. **Do not** force-push, and **do not** create the GitHub remote (I'll create the repo from the name you suggest and push it myself). Just leave it commit-ready and print the exact push commands.

---

## 4. What to build

A redesigned POS app — working name **Tengesa** (you may finalize the name; see section 11) — for Zimbabwean SMEs (tuckshops, spazas, small retailers, semi-informal traders). Tech stack should match/modernize the existing one: **React + TypeScript + Vite + Tailwind + Capacitor + Supabase**, mobile/tablet-first, offline-first.

Start **fresh** (clean scaffold) and **deliberately port** the business logic and Supabase schema across — do not copy-paste the cluttered UI. Redesign the UI/UX from the ground up while keeping every capability.

Capabilities to preserve (verify against the old app — this list may be incomplete):
- **Sale process:** product search / barcode-or-QR scan / grid; cart; quantities; discounts; currency selection; tenders (cash USD, cash ZWG, mobile money e.g. EcoCash/Omari, card); change; receipt **print + share**.
- **Restocking / inventory:** add/edit products, stock levels, min-stock alerts, receive/restock stock entries, stock adjustments, dual-currency cost & price.
- **Cashup / end-of-day reconciliation:** open/close till, expected vs counted per currency, variances, daily summary.
- **Roles:** cashier / supervisor / admin (three-tier), with role-appropriate home screens; Google OAuth for manager login.
- **Reporting:** daily sales, top products, stock value.
- **Bluetooth thermal printing** (58mm). This is the known pain point in the old app — **improve** its reliability (persistent device, reconnect, retry, print queue, graceful failure).
- **Dual currency** USD + ZWG at a configurable fixed rate; **offline-first** Supabase sync with clear online/offline/sync status.

---

## 5. Setup (do this first)

1. Locate and **fully read** the existing codebase (read-only). Map every screen, route, component, and user flow. Note where the UI is cluttered or over-engineered.
2. Create the new sibling project folder and scaffold a clean Vite + React + TS + Tailwind + Capacitor project there.
3. Create the **3 subagents** (section 6) and **3 skills** (section 7).
4. Create a `docs/` folder in the new project for the shared working artifacts the agents hand off through.

> Note on locations: place subagents in `.claude/agents/<name>.md` and skills in `.claude/skills/<name>/SKILL.md` **within the new project**. If your current Claude Code version expects different paths for skills/agents, adapt to whatever your version actually loads, and note it in `docs/DECISIONS.md`.

---

## 6. Create 3 subagents

Author each as a markdown file with YAML frontmatter (`name`, `description`, `tools`) and a full system prompt in the body. Expand the bullets below into proper, detailed system prompts.

### 6.1 `senior-engineer` — Senior Software Engineer / Architect
```yaml
---
name: senior-engineer
description: Senior engineer who audits the old codebase, designs a clean architecture, and implements the new app with full feature parity. Use for all architecture and code-writing work.
tools: Read, Write, Edit, Bash, Grep, Glob
---
```
System prompt should cover: clean modular architecture; strict TypeScript; sensible state management; Supabase integration + offline sync; Capacitor + Bluetooth printing; performance budgets (fast cold start, instant tab switches, optimistic UI); no dead code; **feature parity with the old app**; honour all hard constraints in section 3 (never touch the old app, `--legacy-peer-deps`, test-Supabase only). Draws on the `zim-sme-retail-domain` and `pos-ux-principles` skills.

### 6.2 `ux-architect` — UI/UX Designer & Information Architect
```yaml
---
name: ux-architect
description: UX expert who diagnoses UI clutter, researches best-in-class POS UX, and designs a simplified information architecture, screen flows, and component system. Reviews the built UI against the design.
tools: Read, Write, Grep, Glob, WebSearch, WebFetch
---
```
System prompt should cover: diagnose what makes the old UI cluttered; **research modern POS UX** (reference Loyverse, Square, Shopify POS, Lightspeed/Vend — what makes their sale flows fast and clean); produce a simplified IA, screen list, and key flows; define a restrained component system / design tokens (high-contrast for sunlight readability, large touch targets ≥44–48px, generous spacing, minimal chrome); role-based home screens; unambiguous dual-currency display; offline/sync status patterns. Output goes to `docs/DESIGN.md`. Reviews implemented screens against the design and files UX issues. Draws on the `pos-ux-principles` skill.

### 6.3 `qa-reviewer` — QA / App Tester & Reviewer
```yaml
---
name: qa-reviewer
description: Rigorous, adversarial QA reviewer who writes and runs test plans across the sale, restock, and cashup flows, verifies feature parity vs the old app, and gates the build. Use to test and review everything.
tools: Read, Write, Bash, Grep, Glob
---
```
System prompt should cover: define and execute end-to-end test plans for every critical path and edge case; verify **feature parity** against the old app (nothing lost); review code and UX for defects; check performance (cold start, tab switch, search latency with 1,000+ products); be adversarial. Output goes to `docs/TEST-REPORT.md` with pass/fail, severity, repro steps, and recommended fix. Draws on the `pos-qa-testing` skill.

---

## 7. Create 3 skills

Author each as `.claude/skills/<name>/SKILL.md` with frontmatter (`name`, `description`) and substantive content. These are the shared expertise all three subagents draw on.

### 7.1 `pos-ux-principles`
Principles for POS/retail UX in low-resource, mobile/tablet-first, low-connectivity contexts: speed-of-sale first (minimize taps to complete a sale, prominent numeric keypad, one-handed reach); clarity over density (one primary action per screen, progressive disclosure, rare/admin actions tucked away); offline-first signalling (clear online/offline/sync state; never block a sale on connectivity); dual-currency display rules (show USD + ZWG unambiguously, default per setting, fixed-rate conversion, no ambiguity at payment); receipt/printing UX (quick print + share, reconnect/retry, print queue, graceful failure); role-tailored home screens; restrained high-contrast visual system; performance budgets; accessibility (large fonts, contrast, error prevention). Include a short competitor-pattern reference (Loyverse, Square, Shopify POS).

### 7.2 `zim-sme-retail-domain`
The business domain so all agents share one mental model: user types (tuckshop/spaza owners, small retailers, semi-informal traders; often one device, sometimes tablet cashier + manager phone/desktop); the **sale**, **restock/inventory**, **cashup/EOD**, **roles & permissions**, and **reporting** workflows in detail; money realities (dual currency USD + ZWG at configurable fixed rate, mobile money ubiquity — EcoCash/Omari, cash-heavy, rounding); operating constraints (intermittent power/data, low-end Android, 58mm Bluetooth thermal printers, offline-first with Supabase sync). Pull the concrete schema/flows from the existing repo so this reflects reality.

### 7.3 `pos-qa-testing`
How to test this app: end-to-end critical paths (complete a sale for each tender type × each currency; restock flow; cashup with a deliberate variance; role permission boundaries); edge cases (network lost mid-sale; printer disconnected; app killed mid-transaction → data integrity; zero/negative stock; very large cart; decimal/rounding; currency switch mid-cart); a regression checklist and acceptance-criteria template; performance checks (cold start, tab switch, search latency at scale); and the `TEST-REPORT.md` output format (pass/fail, severity, repro, fix).

---

## 8. The collaboration workflow

You (the main session) orchestrate. Subagents hand off through shared markdown in `docs/`. Run the phases in order and loop where indicated. **Do not pause between phases to ask me — proceed.**

- **Phase 1 — Audit.** `senior-engineer` + `qa-reviewer` review the existing app. Produce `docs/AUDIT.md`: full feature inventory (the parity checklist), screen/flow map, and a list of what makes the current UI cluttered or over-engineered.
- **Phase 2 — Research & design.** `ux-architect` researches best-in-class POS UX and designs the simplified solution. Produce `docs/DESIGN.md`: new information architecture, screen list, key flows, role-based home screens, component system/design tokens, and a before→after rationale tied to the audit.
- **Phase 3 — Plan.** All three agree on the architecture, screen list, and build order. Produce `docs/BUILD-PLAN.md`.
- **Phase 4 — Build.** `senior-engineer` implements in the new folder against the plan and the parity checklist. `ux-architect` reviews screens as they land.
- **Phase 5 — Test.** `qa-reviewer` executes the test plan and writes `docs/TEST-REPORT.md`, explicitly confirming feature parity vs `AUDIT.md`.
- **Phase 6 — Iterate.** Fix every blocker/major issue and re-test until QA passes and parity is confirmed. Loop Phases 4–5 as needed.
- **Phase 7 — Finalize.** Write `README.md` (setup, env vars, run, build, Capacitor/Android, printing notes), confirm the app name, init git, make the initial commit, and print push instructions.

---

## 9. Technical guidelines

- Mobile/tablet-first; optimize for low-end Android and variable connectivity; offline-first with clear sync status.
- Dual currency USD + ZWG, configurable fixed rate (the old project used $1 = ZWG 40 — read the real current value from the repo).
- Reuse the Supabase **schema** (port it cleanly); develop only against the **test** project; never touch live data.
- `npm install --legacy-peer-deps` everywhere.
- Improve Bluetooth thermal printing reliability specifically (persistent device, reconnect, retry, queue, graceful failure).
- Keep dependencies lean; prefer clean, well-typed code over cleverness.

---

## 10. Deliverables

In the new sibling folder:
- The clean, working app (feature parity, simplified UI).
- `.claude/agents/` — the 3 subagents. `.claude/skills/` — the 3 skills.
- `docs/AUDIT.md`, `docs/DESIGN.md`, `docs/BUILD-PLAN.md`, `docs/TEST-REPORT.md`, `docs/DECISIONS.md`.
- `README.md` with full setup/run/build instructions.
- Git initialized with a clean initial commit; exact `git remote add` + `git push` commands printed for me to run after I create the GitHub repo.
- A final summary message: chosen app name + rationale, what changed vs the old app, how to run it, and any assumptions logged.

---

## 11. App name

Working name: **Tengesa** (Shona, "to sell") — folder `tengesa`. You may finalize after the audit; if you keep a different one, justify it briefly in `docs/DECISIONS.md`. Backup options: **Musika** ("market"), **MariTill** ("mari"=money + till), **KwikTill** (speed-focused). Pick something clean, ownable, and easy across Shona/Ndebele/English speakers for the SME Zimbabwean market.

---

## 12. Definition of Done

- [ ] New app builds and runs from the new sibling folder; existing app untouched.
- [ ] Feature parity confirmed in `docs/TEST-REPORT.md` against `docs/AUDIT.md`.
- [ ] Sale, restock, and cashup flows pass QA across both currencies and all tender types; role permissions enforced.
- [ ] Bluetooth printing reliability improved and tested.
- [ ] UI is demonstrably simpler/cleaner, with the rationale in `docs/DESIGN.md`.
- [ ] 3 subagents + 3 skills present and used.
- [ ] All `docs/` files + `README.md` written; git committed; push commands printed.
- [ ] Final summary delivered. No outstanding blockers.

**Now execute all of the above end to end, autonomously, without stopping to ask me questions.**
