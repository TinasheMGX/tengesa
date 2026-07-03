# Tengesa POS — Master Plan
**Version:** 2.0 · **Owner:** MGX · **Status:** Active · **Supersedes:** v1 doc pack (retained in `docs/archive/v1/`)

| Changelog | |
|---|---|
| v2.0 (Jul 2026) | Restructured to the 8-phase programme brief; expanded target platforms; added review gates, versioning rules, options-analysis requirement per phase |
| v1.0 (Jul 2026) | Initial 6-document pack |

---

## 1. What we are building

A production-ready, **offline-first Android POS and stock management system** inspired by Yoco Counter — fast, beautiful, simple for small businesses, scalable, secure, modular — built for the Zimbabwean SME market (dual USD/ZWG, cash-dominant, intermittent connectivity), on a near-zero budget, developed inside **Claude Code** using Memory, Skills, and Subagents.

## 2. Programme structure — 8 sequential phases

**Rule of engagement: one phase at a time. Each phase's document is drafted → reviewed → refined → approved (gate) before the next begins.** Every phase output must: explain reasoning, present multiple options where they exist, recommend one, state trade-offs and risks, and be repo-ready.

| Phase | Deliverable | Repo file | Status |
|---|---|---|---|
| 1 | Product Requirements Document | `docs/01_PRD.md` | **Gate 1 approved (v2.1, 2026-07-04)** |
| 2 | UX & Application Flow (journeys, IA, flowcharts, state/error/offline/sync/auth flows) | `docs/02_UX_FLOWS.md` | **Gate 2 approved (v1.0, 2026-07-04)** |
| 3 | UI/UX Design System (principles, tokens, full component spec, states, Figma structure, inspiration boards) | `docs/03_DESIGN_SYSTEM.md` | Pending gate 3 |
| 4 | Technical Architecture (stack w/ justifications, clean architecture, full DB schemas + ER, REST API design) | `docs/04_TECH_ARCHITECTURE.md` | Pending |
| 5 | Offline-first & Sync Engine (delta sync, queues, conflict resolution, multi-device, multi-store) | `docs/05_OFFLINE_SYNC.md` | Pending |
| 6 | Security (auth, JWT/refresh strategy, encryption, PIN/biometric, OWASP MASVS, audit) | `docs/06_SECURITY.md` | Pending |
| 7 | Testing Strategy (unit, integration, UI, offline, load, security, UAT, QA checklist) | `docs/07_TESTING.md` | Pending |
| 8 | Google Play Release + CI/CD | `docs/08_RELEASE.md` | Pending |
| 8b | Claude Code Workspace (repo structure, Memory strategy, Skills library, Subagents, prompt library, git branching) | `docs/09_CLAUDE_WORKSPACE.md` | Pending |

The v1 pack's App Flow, Design, Backend, Security and Prompt documents are **source material** for Phases 2–8 — each gets absorbed, expanded to the new brief's coverage, and superseded at its phase gate.

## 3. Platform scope decision (input to PRD §MVP)

The brief targets Android phones, Android tablets, web dashboard, desktop, customer portal and merchant analytics in the first production release. Options analysis lives in `01_PRD.md §13`; the **release train (Option B) was adopted at Gate 1**: R1 Android (phone+tablet) → R1.5 Web dashboard + analytics → R2 Desktop (packaged web) → R3 Customer portal → R4 iOS. The architecture is chosen so each train reuses the previous one's code (shared TypeScript core, shared design system), keeping the full brief achievable serially on this budget.

## 4. Documentation standards (all phases)

- Every doc carries: version, owner, status, changelog table, open-questions section.
- Markdown, Mermaid for all diagrams (renders in GitHub/Claude Code; imports into Lucidchart via *Insert → Diagram as code → Mermaid*).
- Docs live in `docs/`, are committed with the code, and are updated in the same PR as any behaviour change ("docs-or-it-didn't-happen").
- Each doc is written to be **independently consumable by Claude Code** — self-contained context, explicit file references, no reliance on chat history.

## 5. Repository skeleton (final form defined in Phase 8b; created day one)

```
tengesa/
├── CLAUDE.md                  # project memory (golden rules, decisions log)
├── .claude/
│   ├── agents/                # engineer, ux-architect, qa-reviewer, sync-specialist
│   └── skills/                # money-math, sync-op, supabase-migration, receipt-render
├── docs/                      # phases 1–8 documents + /archive/v1/
├── prompts/                   # reusable prompt library (per-phase build prompts)
├── src/                       # app source (structure per Phase 4)
└── supabase/migrations/       # cloud schema (applied manually by MGX)
```

## 6. Budget guardrails (unchanged, binding on every phase)

Free/open-source/self-hostable first. Supabase-class free tier for cloud (final DB comparison — Firebase vs Supabase vs self-hosted PocketBase — is a Phase 4 decision with full justification). Google Play $25 one-time is the only approved spend. No paid SaaS, fonts, icons, or APIs. Any proposed spend requires a written trade-off note and MGX approval.

## 7. Current gate

**GATE 1 — PRD review.** ✓ Approved (2026-07-04). PRD v2.1 is the binding R1 contract. All §17 open questions resolved.

**GATE 2 — UX & Application Flow.** ✓ Approved (2026-07-04). 41 screens, all Must FRs covered, ux-architect reviewed.

**GATE 3 — Design System.** Phase 3 may begin on MGX approval.
