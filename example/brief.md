# Codebase Recon Brief — gstack

**Generated:** 2026-04-05
**Scope:** full project
**Git window:** 2026-03-11 → 2026-04-05 (full history — project is 25 days old)

---

## TL;DR

- **gstack is very young (25 days) and moving fast** — 0.0.1 → 0.15.x in 25 days, ~150 commits, 30+ skills shipped. High velocity means everything is in flux.
- **`scripts/gen-skill-docs.ts` is the single most dangerous file to touch.** It is the code-to-docs bridge that generates all 30+ SKILL.md files from templates. Changes here cascade instantly to every skill's AI instructions. It is the project's highest-fragility module (score 78/100 🔴 DANGER).
- **`browse/src/server.ts` is the load-bearing pillar of the browser subsystem.** At 1530 lines, it imports every command handler, the browser manager, config, buffers, CDP inspector, and activity layer. It is the God Module — every browse command flows through it.
- **`ship/SKILL.md` is the top bug attractor** — 21 fix commits in 25 days, 66 total commits. Changes here require tier-2 E2E eval gate (`bun run test:evals`) before landing.
- **The SKILL.md template pipeline is an architectural contract.** SKILL.md files are auto-generated from `.tmpl` files — they must never be hand-edited. This constraint is invisible from `git blame` alone and has caused recurring bugs.

---

## Architecture Overview

```
STACK
══════════════════════════════════════════════════════
Language:       TypeScript (strict ESM)
Runtime:        Bun ≥ 1.0.0
Build:          bun build --compile → single binaries (browse/dist/, design/dist/, bin/)
Test:           Bun test (3 tiers: static, E2E via claude -p, LLM-as-judge)
Dependencies:   playwright, puppeteer-core, diff | devDep: @anthropic-ai/sdk
CI/CD:          GitHub Actions (implied from PR merge commit pattern)
Deploy:         Compiled binaries + ~/.claude/skills/ symlinks
Started:        2026-03-11 (v0.0.1)
Current:        v0.15.x
```

```
LAYERS
═══════════════════════════════════════════════════════════════════════
[1] Skill Layer      */SKILL.md, */SKILL.md.tmpl         AI workflow instructions (30+ skills)
[2] Doc Gen Layer    scripts/gen-skill-docs.ts            .tmpl → SKILL.md pipeline
                     scripts/resolvers/                   per-skill template resolvers
[3] Browser Layer    browse/src/                          HTTP daemon + Chromium control
[4] Design Layer     design/src/                          Visual design tool binary
[5] Bin/Utils Layer  bin/gstack-*                         Shell utilities for skills
[6] Test Layer       test/, browse/test/                  Static + E2E + LLM-eval
```

```
INTEGRATION POINTS
══════════════════════════════════════════════════════════════════════════
Chromium          CDP via Playwright       browse/src/browser-manager.ts
Claude Code       Skill invocation         */SKILL.md (runtime contract)
Supabase          Telemetry                supabase/ + bin/gstack-telemetry-*
macOS Keychain    Cookie decryption        browse/src/cookie-import-browser.ts
Codex / Factory   Alternative AI runtime  scripts/resolvers/codex-helpers.ts
Extension         Sidebar agent           browse/src/sidebar-agent.ts + extension/
```

### Architecture Notes

**The daemon model** is load-bearing: gstack runs a long-lived Chromium process at a random port (10000–60000). The CLI discovers it via `.gstack/browse.json`, spawns it on first use, and auto-kills it after 30-minute idle. Every command is an HTTP POST — sub-second after first boot.

**SKILL.md template pipeline** (⚠ architectural constraint invisible to new contributors):
```
*/SKILL.md.tmpl   (human-written prose + {{PLACEHOLDER}} markers)
       ↓
scripts/gen-skill-docs.ts  (reads COMMAND_DESCRIPTIONS from commands.ts,
                             SNAPSHOT_FLAGS from snapshot.ts, per-skill resolvers)
       ↓
*/SKILL.md   (committed — NEVER hand-edit these files)
```
Breaking `gen-skill-docs.ts` silently corrupts all skill instructions. Breaking `commands.ts` or `snapshot.ts` cascades through `gen-skill-docs.ts` into every SKILL.md.

**Known tech debt (from source):** None flagged via TODO/FIXME/HACK in core source. Active TODOS.md file exists (runtime TODO tracking system used by skills themselves).

---

## Risk Map

```
FRAGILITY SCORES
════════════════════════════════════════════════════════════════════════
Module                            Churn  Coupling  BlastR  TestGap  SCORE  BAND
──────                            ─────  ────────  ──────  ───────  ─────  ────
scripts/gen-skill-docs.ts          9.0     8.0     10.0     3.0      78   🔴 DANGER
ship/SKILL.md                      9.0     7.0      5.0     7.0      71   🔴 DANGER
browse/src/server.ts               4.0     8.0      8.0     5.0      62   🟠 FRAGILE
browse/src/browser-manager.ts      3.0     5.0      8.0     4.0      50   🟠 FRAGILE
browse/src/snapshot.ts             5.0     6.0      8.0     4.0      58   🟠 FRAGILE
browse/src/commands.ts             2.0     7.0     10.0     3.0      55   🟠 FRAGILE
review/SKILL.md                    8.5     7.0      5.0     7.0      69   🔴 DANGER*
browse/src/cookie-picker-ui.ts     3.0     3.0      2.0    10.0      42   🟠 FRAGILE
browse/src/browser-manager.ts      3.0     5.0      8.0     4.0      50   🟠 FRAGILE
browse/src/write-commands.ts       2.0     3.0      5.0     3.0      32   🟡 CAUTION
browse/src/read-commands.ts        2.0     3.0      5.0     3.0      32   🟡 CAUTION
lib/worktree.ts                    1.0     1.0      2.0     6.0      23   🟡 CAUTION
browse/src/config.ts               1.0     2.0      3.0     3.0      20   🟢 STABLE
browse/src/url-validation.ts       1.0     1.0      2.0     3.0      15   🟢 STABLE
browse/src/buffers.ts              1.0     2.0      4.0     3.0      22   🟡 CAUTION

* review/SKILL.md scored together with plan-* SKILL files — all move as a cluster
Note: TestGap scores are partially estimated (no coverage tool output available).
```

**TOP 5 MOST FRAGILE**
1. `scripts/gen-skill-docs.ts`   (78) — 🔴 DANGER
2. `review/SKILL.md` cluster     (69) — 🔴 DANGER
3. `ship/SKILL.md`               (71) — 🔴 DANGER
4. `browse/src/server.ts`        (62) — 🟠 FRAGILE
5. `browse/src/snapshot.ts`      (58) — 🟠 FRAGILE

---

## Git Hotspots

```
GIT HOTSPOT ANALYSIS
════════════════════
Window:                  2026-03-11 → 2026-04-05 (full history)
Total commits analyzed:  ~150

TOP CHURNING FILES (excluding CHANGELOG.md / VERSION — always move together)
#   Commits   File                              Signal
──  ───────   ────                              ──────
1    66       ship/SKILL.md                     🔥 TOP BUG ATTRACTOR
2    62       scripts/gen-skill-docs.ts         🔥 HIGH CHURN + bug attractor
3    59       review/SKILL.md                   🔥 HIGH CHURN
4    58       plan-eng-review/SKILL.md          🔥 HIGH CHURN
5    58       plan-ceo-review/SKILL.md          🔥 HIGH CHURN
6    51       SKILL.md (root)                   HIGH CHURN
7    51       qa/SKILL.md                       HIGH CHURN
8    48       plan-design-review/SKILL.md       HIGH CHURN
9    45       retro/SKILL.md                    HIGH CHURN
10   44       browse/SKILL.md                   HIGH CHURN
11   43       test/skill-validation.test.ts     HIGH CHURN (follows gen-skill-docs)
12   40       office-hours/SKILL.md             MODERATE
13   39       qa-only/SKILL.md                  MODERATE
14   39       design-consultation/SKILL.md      MODERATE
15   37       setup-browser-cookies/SKILL.md    MODERATE

IMPLICIT COUPLING (always move together)
  gen-skill-docs.ts    ↔  any SKILL.md file         (template regeneration)
  gen-skill-docs.ts    ↔  test/gen-skill-docs.test.ts
  browse/src/commands.ts ↔  browse/src/snapshot.ts  (COMMAND_DESCRIPTIONS + SNAPSHOT_FLAGS
                                                      both exported to gen-skill-docs)
  ship/SKILL.md        ↔  review/SKILL.md            (shared workflow semantics)
  plan-eng-review      ↔  plan-ceo-review            (always land together)
  CHANGELOG.md         ↔  VERSION                    (every release)
  package.json         ↔  VERSION                    (version bumps)

BUG ATTRACTORS (3+ fix commits)
  ship/SKILL.md              21 fix commits  (top attractor — idempotency, PR body, branch scope)
  scripts/gen-skill-docs.ts  20 fix commits  (freshness checks, template drift)
  review/SKILL.md            17 fix commits
  plan-ceo-review/SKILL.md   16 fix commits
  qa/SKILL.md                15 fix commits
  plan-eng-review/SKILL.md   15 fix commits
  codex/SKILL.md             12 fix commits  (Codex compat edge cases)
  autoplan/SKILL.md          11 fix commits
  setup/                     11 fix commits

DORMANT FILES
  None — project is 25 days old, all files recently modified.
```

---

## Blast Radius Ledger

```
INTERNAL DEPENDENCY PROFILE (browse/src/)
  Module                    Fan-In   Fan-Out   Cycles   Role
  ──────                    ──────   ───────   ──────   ────
  server.ts                   2*      13+        0      GOD MODULE (all commands route through it)
  browser-manager.ts           7        3        0      PILLAR (Playwright lifecycle)
  snapshot.ts                  7        1        0      PILLAR (ref system + SNAPSHOT_FLAGS export)
  commands.ts                  5        0        0      PILLAR (command registry, exported to gen-skill-docs)
  cookie-picker-ui.ts          1        2        0      LEAF (no tests)
  read-commands.ts             2        2        0      LEAF
  write-commands.ts            2        2        0      LEAF
  meta-commands.ts             2        3        0      LEAF
  buffers.ts                   3        0        0      SHARED UTILITY
  config.ts                    3        0        0      SHARED UTILITY
  url-validation.ts            2        0        0      SHARED UTILITY
  sidebar-agent.ts             1        3        0      FEATURE MODULE
  cdp-inspector.ts             1        0        0      FEATURE MODULE
  activity.ts                  2        0        0      SHARED UTILITY

* server.ts fan-in of 2 (cli.ts + find-browse.ts) is misleading — it IS the runtime,
  everything executes inside it.

BLAST RADIUS: scripts/gen-skill-docs.ts
══════════════════════════════════════════
Direct dependents (code):  6
  scripts/dev-skill.ts
  scripts/skill-check.ts
  test/skill-validation.test.ts
  test/touchfiles.test.ts
  test/helpers/touchfiles.ts
  test/gen-skill-docs.test.ts

Runtime blast (all SKILL.md files regenerated):  30+ SKILL.md files
  → every skill's AI instructions are at risk if gen-skill-docs output changes

TOTAL BLAST RADIUS:  CRITICAL
  Any interface change in commands.ts or snapshot.ts cascades:
  commands.ts → gen-skill-docs.ts → all 30+ SKILL.md files

BLAST RADIUS: browse/src/browser-manager.ts
══════════════════════════════════════════════
Direct dependents (Tier 1):  7
  buffers.ts, snapshot.ts, cookie-picker-routes.ts,
  meta-commands.ts, write-commands.ts, read-commands.ts, server.ts

Transitive (Tier 2):  cli.ts (via server.ts)
TOTAL BLAST RADIUS:  HIGH — entire browse runtime depends on it

BLAST RADIUS: browse/src/commands.ts
══════════════════════════════════════
Direct dependents:  server.ts, meta-commands.ts, cli.ts, sidebar-agent.ts,
                    gen-skill-docs.ts (cross-layer!)
TOTAL BLAST RADIUS:  CRITICAL — code change + skill doc change in one shot
```

---

## Safe Change Strategy

```
RECOMMENDED CHANGE SEQUENCE
════════════════════════════

Step 0 — UNDERSTAND THE TEMPLATE PIPELINE FIRST (mandatory for anyone new):
  [ ] Read ARCHITECTURE.md section "SKILL.md template system"
  [ ] Run: bun run gen:skill-docs --dry-run && git diff --stat
      (this tells you what would regenerate — run it after ANY change to
       gen-skill-docs.ts, commands.ts, snapshot.ts, or any resolver)
  [ ] Understand: SKILL.md files are never hand-edited, always generated

Step 1 — Low-risk (safe to start now):
  [ ] browse/src/url-validation.ts     — isolated utility, well-tested
  [ ] browse/src/config.ts             — low coupling, config only
  [ ] browse/src/buffers.ts            — ring buffer impl, bounded scope
  [ ] lib/worktree.ts                  — single-purpose utility
  [ ] bin/gstack-* shell utilities     — standalone scripts, low coupling

Step 2 — Medium-risk (understand blast radius first):
  [ ] browse/src/read-commands.ts      — add verification: bun test browse/test/
  [ ] browse/src/write-commands.ts     — add verification: bun test browse/test/
  [ ] browse/src/meta-commands.ts      — impacts snapshot path, test thoroughly
  [ ] browse/src/sidebar-agent.ts      — feature-flagged effectively by sidebar UX
  [ ] browse/src/activity.ts           — ring buffer, low coupling

Step 3 — High-risk (treat as surgery):
  [ ] browse/src/snapshot.ts           — SNAPSHOT_FLAGS is exported to gen-skill-docs.
                                         After any change: run gen:skill-docs --dry-run,
                                         then bun test, then EVALS tier.
  [ ] browse/src/commands.ts           — COMMAND_DESCRIPTIONS exported to gen-skill-docs.
                                         Same gate as snapshot.ts. One command rename =
                                         all skill docs regenerate.
  [ ] browse/src/browser-manager.ts    — Playwright lifecycle, 7 dependents.
                                         Characterization tests before refactor.

Step 4 — DANGER zone (full harness required):
  [ ] scripts/gen-skill-docs.ts        — Strategy: Characterization Tests.
                                         Add snapshot tests for every resolver output
                                         BEFORE changing template logic. Gate: CI must
                                         run gen:skill-docs --dry-run + git diff --exit-code.
  [ ] ship/SKILL.md.tmpl (not .md!)    — Strategy: Feature Flag Guard via EVALS tier.
                                         Change tmpl, regenerate, run:
                                         bun run test:evals (costs ~$3.85, ~20min).
                                         Never land a ship/review/qa template change
                                         without passing tier-2 E2E.
  [ ] browse/src/server.ts             — Strategy: Incremental Extract.
                                         1530 lines is too large to change atomically.
                                         Extract one handler group at a time (e.g. move
                                         CDP inspector routes to their own router).
                                         Each extraction = one PR, one test run.

AVOID until last:
  ⛔ browse/src/browser-manager.ts (Playwright session lifecycle) — changes here
     can silently break cookie state, ref staleness detection, and tab persistence.
     Requires full integration testing including actual browser interaction.
  ⛔ scripts/gen-skill-docs.ts (without characterization tests) — a bad template
     change can corrupt AI instructions for all 30+ skills simultaneously, with no
     automated safety net beyond the freshness check.
```

---

## Known Tech Debt

From source analysis and commit history:

| Signal | Location | Note |
|---|---|---|
| Undocumented constraint | All SKILL.md files | Must not be hand-edited — only via .tmpl + gen-skill-docs. No in-file warning. |
| Untested module | `browse/src/cookie-picker-ui.ts` (688 lines) | Only file in browse/src/ with no corresponding test |
| God Module | `browse/src/server.ts` (1530 lines) | Imports all command handlers + all cross-cutting concerns. Should be split into sub-routers. |
| Recurring bug surface | `ship/SKILL.md` + `review/SKILL.md` | 21 and 17 fix commits respectively in 25 days. Template logic for PR scoping is complex and fragile. |
| Windows support incomplete | `browse/src/cookie-import-browser.ts` | Cookie decryption only supports macOS Keychain. Windows DPAPI not implemented. |
| Multi-window ELI16 mode | `scripts/resolvers/preamble.ts` | Session count detection via mtime heuristic (files modified in last 2 hours) — fragile on slow machines |

---

## Appendix: Raw Metrics

```
FULL CHURN TABLE (non-metadata files, 6-month window = full project history)
  Rank  Commits  File
  ────  ───────  ────
  1      66      ship/SKILL.md
  2      62      scripts/gen-skill-docs.ts
  3      59      review/SKILL.md
  4      58      plan-eng-review/SKILL.md
  5      58      plan-ceo-review/SKILL.md
  6      53      package.json
  7      51      SKILL.md
  8      51      qa/SKILL.md
  9      50      test/gen-skill-docs.test.ts
  10     48      plan-design-review/SKILL.md
  11     45      retro/SKILL.md
  12     44      browse/SKILL.md
  13     43      test/skill-validation.test.ts
  14     40      office-hours/SKILL.md
  15     39      qa-only/SKILL.md
  16     39      design-consultation/SKILL.md
  17     37      setup-browser-cookies/SKILL.md
  18     36      ship/SKILL.md.tmpl
  19     36      README.md
  20     36      CLAUDE.md

FULL FRAGILITY SCORE TABLE
  Module                              Score  Band
  ──────                              ─────  ────
  scripts/gen-skill-docs.ts            78    🔴 DANGER
  ship/SKILL.md                        71    🔴 DANGER
  review/SKILL.md (cluster)            69    🔴 DANGER
  browse/src/server.ts                 62    🟠 FRAGILE
  browse/src/snapshot.ts               58    🟠 FRAGILE
  browse/src/commands.ts               55    🟠 FRAGILE
  browse/src/browser-manager.ts        50    🟠 FRAGILE
  browse/src/cookie-picker-ui.ts       42    🟠 FRAGILE
  browse/src/write-commands.ts         32    🟡 CAUTION
  browse/src/read-commands.ts          32    🟡 CAUTION
  browse/src/buffers.ts                22    🟡 CAUTION
  lib/worktree.ts                      23    🟡 CAUTION
  browse/src/config.ts                 20    🟢 STABLE
  browse/src/url-validation.ts         15    🟢 STABLE
  browse/src/sidebar-utils.ts          12    🟢 STABLE

BLAST RADIUS SUMMARY
  Module                       Direct  Transitive  Risk
  ──────                       ──────  ──────────  ────
  scripts/gen-skill-docs.ts      6     30+ SKILLs  CRITICAL
  browse/src/commands.ts         5     all skills  CRITICAL
  browse/src/snapshot.ts         7     all skills  CRITICAL (via gen-skill-docs)
  browse/src/browser-manager.ts  7     all browse  HIGH
  browse/src/server.ts           2*    all browse  HIGH (*runtime hub)
  browse/src/meta-commands.ts    2     limited     MEDIUM
  browse/src/read-commands.ts    2     limited     MEDIUM
  browse/src/write-commands.ts   2     limited     MEDIUM
  browse/src/cookie-picker-ui.ts 1     none        LOW
  browse/src/config.ts           3     config only LOW
  browse/src/url-validation.ts   2     none        LOW
```
