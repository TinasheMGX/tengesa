# Tengesa POS — Project Document Pack

Offline-first POS & stock management app for Zimbabwean SMEs, modelled on Yoco Counter / Yoco POS App (South Africa), built on a zero/near-zero budget.

## Documents

| File | Purpose |
|---|---|
| `01_PRD.md` | Product Requirements Document — what we're building and why |
| `02_APP_FLOW.md` | End-to-end app flows as Mermaid diagrams (import into Lucidchart) |
| `03_DESIGN.md` | Design doc, frontend guidelines, reference/inspiration sources |
| `04_BACKEND.md` | Tech stack, local + cloud schema, sync protocol, APIs |
| `05_SECURITY_PLAYSTORE.md` | Security checklist + Google Play Store readiness |
| `06_CLAUDE_CODE_PROMPTS.md` | Phased build prompts for Claude Code |
| `CLAUDE.md` | Drop into repo root — Claude Code project memory |
| `.claude/agents/*.md` | Subagent definitions (engineer, ux-architect, qa-reviewer, sync-specialist) |

## How to use with Claude Code

1. Create the repo (or open the Tengesa repo). Copy this entire folder in:
   - `CLAUDE.md` → repo root
   - `.claude/agents/` → repo root
   - Everything else → `docs/`
2. Start Claude Code in the repo. It auto-loads `CLAUDE.md` (project memory).
3. Work phase by phase using the prompts in `06_CLAUDE_CODE_PROMPTS.md`. One phase per session; commit at the end of each.
4. Use `/agents` to confirm the subagents loaded. Invoke explicitly, e.g. *"Use the qa-reviewer subagent to review the sync engine."*
5. Add durable decisions to `CLAUDE.md` as you go (`#` shortcut in Claude Code appends to memory).

## Budget guardrails (non-negotiable)

- Supabase **free tier** (500 MB DB, 50k MAU auth, 1 GB storage, 500k edge function invocations/mo)
- Google Play developer account: **US$25 one-time** — the only mandatory spend
- No paid SaaS, no paid APIs, no paid fonts/icons (Google Fonts + Lucide only)
- CI: GitHub Actions free tier

## Yoco feature parity map (MVP)

Sell tab (amount + catalogue) · History (search, refund, resend receipt, notes) · Reports (day/week/month; payment type, product, staff) · Catalogue (categories, variants, tile view, images) · Stock (item-level tracking, low-stock alerts) · Staff PINs + permissions · Tender types as record-keeping (cash USD/ZWG, EcoCash, Zipit, card, other) · Digital receipts (share via WhatsApp/SMS) · Saved/open bills · Discounts

Explicitly NOT in MVP: card machine integration, payment links, recipe-level inventory, multi-location, tips, split bills (v1.1).
