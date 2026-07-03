# Next Steps — Moving Tengesa into Claude Code
**For:** MGX · **Stage:** Post-PRD draft, Gate 1 pending · **Read time:** 5 min

Important framing: chat history does not transfer to Claude Code — **the docs are the export**. Everything Claude Code needs to know lives in `CLAUDE.md` + `docs/`. That's why the pack was written to be self-contained.

---

## Step 1 — Create the repo and place the files (10 min)

```bash
mkdir tengesa && cd tengesa
git init
mkdir -p docs/archive/v1 .claude/agents prompts supabase/migrations src
```

Place your downloaded files:

| Downloaded file | Goes to |
|---|---|
| `CLAUDE.md` (v2 — provided alongside this file) | repo root `CLAUDE.md` |
| `00_MASTER_PLAN_v2.md` | `docs/00_MASTER_PLAN.md` (drop the `_v2`) |
| `01_PRD_v2.md` | `docs/01_PRD.md` |
| v1 zip contents: `02_APP_FLOW.md`, `03_DESIGN.md`, `04_BACKEND.md`, `05_SECURITY_PLAYSTORE.md`, `06_CLAUDE_CODE_PROMPTS.md` | `docs/archive/v1/` (source material for Phases 2–8) |
| v1 zip: `.claude/agents/*.md` (engineer, ux-architect, qa-reviewer, sync-specialist) | `.claude/agents/` |
| This file | `docs/NEXT_STEPS.md` |

Then:

```bash
git add -A && git commit -m "docs: programme scaffold — master plan v2, PRD v2, v1 archive, agents"
git branch -M main
# optional but recommended: create private GitHub repo TinasheMGX/tengesa and push
```

## Step 2 — Session 1 in Claude Code: workspace check + Gate 1 (30–45 min)

Open a terminal in the repo, run `claude`, then verify the workspace: type `/agents` — you should see the four subagents. Then paste:

```
Read CLAUDE.md, docs/00_MASTER_PLAN.md and docs/01_PRD.md in full.

We are at Gate 1. Act as a sceptical product reviewer and walk me through the
PRD's §17 open questions ONE AT A TIME. For each: summarise the decision needed
in two sentences, give your recommendation with the key trade-off, then stop
and wait for my answer before moving to the next.

After all six are answered:
1. Update docs/01_PRD.md to v2.1 — apply my decisions, adjust §13/§18 (MoSCoW)
   and the changelog accordingly.
2. Update CLAUDE.md: fill in the "Pending Gate 1 decisions" section with the
   outcomes, dated.
3. Use the qa-reviewer subagent to audit the updated PRD for ambiguous or
   untestable functional requirements; show me its findings; fix the BLOCKERs.
4. Summarise what changed, then stop. Do not begin Phase 2.
```

Commit: `git commit -am "docs: PRD v2.1 — Gate 1 decisions applied"`

## Step 3 — Sessions 2–8: one documentation phase per session

Start each session fresh (`/clear` if continuing a window). The pattern for every phase:

```
Read CLAUDE.md and docs/00_MASTER_PLAN.md. We are starting Phase <N>: <name>.

Source material: docs/archive/v1/<relevant v1 doc>.md — absorb what is still
correct, expand to the full Phase <N> coverage listed in the master plan table,
and correct anything the approved PRD (docs/01_PRD.md) has since changed.

Requirements for the output (docs/0<N>_<NAME>.md):
- Explain reasoning; present options where real choices exist; recommend one;
  state trade-offs and risks.
- All diagrams in Mermaid. Version header + changelog + open questions section.
- Self-contained: no reliance on any prior conversation.

Draft it, then use the <most relevant subagent> to review it, apply the
feedback, show me a summary of decisions and open questions, and stop for my
gate review. Do not start the next phase.
```

Phase → source → reviewing subagent:

| Phase | New doc | v1 source | Reviewer |
|---|---|---|---|
| 2 UX & App Flow | `02_UX_FLOWS.md` | `02_APP_FLOW.md` | ux-architect |
| 3 Design System | `03_DESIGN_SYSTEM.md` | `03_DESIGN.md` | ux-architect |
| 4 Tech Architecture | `04_TECH_ARCHITECTURE.md` | `04_BACKEND.md` | engineer |
| 5 Offline & Sync | `05_OFFLINE_SYNC.md` | `04_BACKEND.md §4` | sync-specialist |
| 6 Security | `06_SECURITY.md` | `05_SECURITY_PLAYSTORE.md Part A` | qa-reviewer |
| 7 Testing | `07_TESTING.md` | (new; AC in PRD §11 seed it) | qa-reviewer |
| 8 Release | `08_RELEASE.md` | `05_SECURITY_PLAYSTORE.md Part B` | qa-reviewer |
| 8b Claude Workspace | `09_CLAUDE_WORKSPACE.md` | `06_CLAUDE_CODE_PROMPTS.md` | engineer |

Commit after every gate: `git commit -am "docs: phase <N> <name> v1.0 approved"`

## Step 4 — After Phase 8b: switch to build mode

Phase 8b regenerates the build-prompt library into `prompts/` (superseding the archived v1 prompt pack) and seeds `.claude/skills/` (money-math, sync-op, supabase-migration, receipt-render). From there, development follows the phased build prompts — feature branches (`feat/sell-flow`), PRs into `main`, one build phase per session.

## Operating rules (all sessions)

1. **One phase per session.** Finish lines protect quality and your energy.
2. **`/agents` at session start** — confirm the four subagents loaded.
3. **`#` to remember** — any durable decision mid-session: type `#` and the decision; Claude Code appends it to CLAUDE.md. Review the diff before committing.
4. **Docs change with behaviour** — same commit, always.
5. **You apply Supabase SQL yourself** (SQL Editor); Claude Code only writes migration files.
6. **`--legacy-peer-deps`** on every npm install once build phases start.
7. Money maths, sync logic, role gates: never merged without a qa-reviewer pass.
