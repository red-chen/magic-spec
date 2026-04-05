---
name: magic-codebase-recon
description: "Use when first encountering an unfamiliar codebase, modernizing a legacy system, or dealing with code nobody dares to touch. Runs architecture archaeology, git hotspot analysis, dependency blast-radius assessment, fragility scoring, and produces a safe-change strategy in brief.md."
---

# /magic-codebase-recon — Codebase Reconnaissance & Safe-Change Intelligence

You've just inherited a codebase. You don't know what's fragile, what's load-bearing, or what will explode if you touch it. This skill maps the battlefield before you move.

**Philosophy:** Before you refactor, you must understand. Before you understand, you must measure. The best change strategy begins with an honest map of what exists — not what should exist.

## When to Use

- **Legacy system modernization** — rewriting, migrating, or incrementally modernizing old code
- **First contact** — onboarding to an unfamiliar codebase, understanding what you've inherited
- **"Don't touch that" code** — the module nobody dares to change, the file that breaks on Fridays
- **Pre-refactor due diligence** — before any large-scale structural change
- **Risk assessment** — evaluating how dangerous a specific change is before committing
- **Technical debt triage** — deciding where to invest cleanup effort first

## Arguments

- `/magic-codebase-recon` — Full recon (all phases, outputs `brief.md`)
- `/magic-codebase-recon --scope [path]` — Focus recon on a specific module or directory
- `/magic-codebase-recon --target [file-or-module]` — Blast-radius analysis for a specific change target
- `/magic-codebase-recon --since [date|duration]` — Limit git analysis to a time window (default: 6 months)
- `/magic-codebase-recon --hotspots` — Git hotspot analysis only (Phases 0, 2, 5)
- `/magic-codebase-recon --blast-radius [file-or-module]` — Dependency + blast-radius only (Phases 0, 3, 5)

**Scope flags are mutually exclusive.** If multiple are passed, error immediately. `--since` is combinable with any scope flag.

## Hard Gate

<HARD-GATE>
This skill is READ-ONLY. Do NOT modify any code, configs, or files during recon. The mission is intelligence gathering, not action. The only output is brief.md. Implementation decisions happen separately using /magic-proposal and /magic-apply.
</HARD-GATE>

## Recon Phases

### Phase 0: Stack & Entry Point Detection

Understand what you're dealing with before measuring anything.

**Detect:**
- Primary language(s) and runtime (Node/TypeScript, Python, Go, Java, Ruby, Rust, PHP, .NET, etc.)
- Frameworks and major libraries (Next.js, Django, Spring Boot, Rails, etc.)
- Build system, package manager, dependency manifest
- Test framework(s) and coverage tooling
- CI/CD system (GitHub Actions, Jenkins, GitLab CI, etc.)
- Deployment target (containers, serverless, VMs, etc.)

**Map entry points:**
- Main executable / entrypoint files
- Public API surface (HTTP routes, gRPC services, CLI commands, exported modules)
- Background jobs and scheduled tasks
- Event listeners and message consumers

**Express as an architecture summary** before proceeding. If the stack is unclear after reading key files, flag it and continue with best-effort detection.

---

### Phase 1: Architecture Archaeology

*Understand what was built and why it looks the way it does today.*

**Structural mapping:**
- Directory layout and layer organization (MVC, hexagonal, flat, monolith, modular monolith, microservice, etc.)
- Core domain boundaries — where does one responsibility end and another begin?
- Data layer: database(s), ORM/query patterns, migration history
- Integration points: external APIs, third-party services, internal service calls
- Shared infrastructure: logging, config management, auth, feature flags

**Historical excavation (read git log, not just code):**
- When was the codebase started? What was the original intent? (read early commits)
- Major refactors or architectural pivots — look for mass-rename commits, directory restructures
- Files/modules that started as one thing and became something else (diverged from original purpose)
- Patterns that appear organic vs. patterns that appear designed
- Long-lived TODO / FIXME / HACK comments that reveal known technical debt

**Findings to record:**
- All major architectural layers with file paths
- Architectural seams (boundaries that look designed vs. accidental coupling)
- Original design intent vs. current reality if they've diverged
- Known tech debt explicitly called out in code comments or docs

**Output format:**
```
ARCHITECTURE MAP
════════════════
Stack:             [languages, frameworks, runtime]
Pattern:           [MVC | Hexagonal | Flat | Modular Monolith | etc.]
Age:               [inferred from first commit date]
Last major pivot:  [date + description if found]

LAYERS
  [Layer Name]       [path glob]                  [purpose]
  ...

INTEGRATION POINTS
  [Name]             [type: HTTP|DB|Queue|etc.]   [path]
  ...

KNOWN TECH DEBT (from source comments / docs)
  [file:line]        [comment excerpt]
  ...
```

---

### Phase 2: Git Hotspot Analysis

*Find the files that change most often, break most often, and always touch each other.*

**Churn analysis — files that change most:**
```bash
# Overall churn (all time or --since window)
git log --format=format: --name-only [--since=<date>] | grep -v '^$' | sort | uniq -c | sort -rg | head -30

# Narrow to a path scope if --scope was specified
git log --format=format: --name-only [--since=<date>] -- [path] | grep -v '^$' | sort | uniq -c | sort -rg | head -30
```

Report top 20 files by commit count. High churn = high maintenance cost and likely instability.

**Coupling analysis — files that always change together:**

Co-change coupling reveals hidden dependencies not visible in the import graph:
```bash
# Sample recent commits, extract file pairs that co-change
git log --format="%H" [--since=<date>] | head -200 | \
  xargs -I {} git diff-tree --no-commit-id -r --name-only {} | \
  awk 'NF' | sort | uniq -c | sort -rg
```

Flag file pairs with >30% co-change rate as **implicitly coupled**.

**Bug fix concentration:**
```bash
# Commits with fix/bug/hotfix/patch/revert in message
git log --format="%H %s" [--since=<date>] | \
  grep -iE '\b(fix|bug|hotfix|patch|revert)\b'
```

Extract which files these commits touch. Files appearing in 3+ bug-fix commits = **bug attractors**.

**Recency of activity:**
- Files not touched in >1 year = **dormant** (low collision risk, but may contain stale assumptions)
- Files touched in last 2 weeks = **active** (high collision risk for concurrent changes)

**Output format:**
```
GIT HOTSPOT ANALYSIS
════════════════════
Window:                  [date range analyzed]
Total commits analyzed:  N

TOP CHURNING FILES
#   Commits   File                              Signal
──  ───────   ────                              ──────
1   47        src/api/auth.ts                   🔥 HIGH CHURN
2   31        src/db/migrations/index.ts        🔥 HIGH CHURN
...

IMPLICIT COUPLING (co-change pairs, >30% rate)
  [File A]  ↔  [File B]    (N% co-change rate, M commits)
  ...

BUG ATTRACTORS (3+ bug-fix commits)
  [file]    [N bug-fix commits]    [last bug date]
  ...

DORMANT FILES (>1 year since last commit)
  [file]    [last commit date]
  ...
```

---

### Phase 3: Dependency Tracking & Blast Radius Assessment

*Map what depends on what, then calculate the damage radius if something breaks.*

**Internal dependency graph:**

Trace import/require/use relationships to build a module dependency profile.

For each module, identify:
- **Fan-in**: how many modules import this (load-bearing pillar if high)
- **Fan-out**: how many modules this imports (complex leaf if high)
- **Dependency cycles**: circular imports = architectural smell

```
INTERNAL DEPENDENCY PROFILE
  Module                    Fan-In   Fan-Out   Cycles   Role
  ──────                    ──────   ───────   ──────   ────
  src/core/user.ts            12        3        0      PILLAR (high fan-in)
  src/utils/helpers.ts         8        1        0      SHARED UTILITY
  src/api/checkout.ts          1       11        0      COMPLEX LEAF
  src/db/conn.ts               9        0        1      ⚠ CYCLE DETECTED
```

**External dependency exposure:**
- All external dependencies with version pinning status
- Dependencies used directly by more than 3 internal modules (high blast radius if changed/removed)
- Dependencies with no type definitions or poor maintenance signals
- Cross-reference with Phase 2: do hotspot files heavily rely on a single external library?

**Blast radius calculation:**

For any module M, its blast radius = all modules that transitively depend on M.

When `--target` is specified, compute exact blast radius:
1. Find all direct dependents of target
2. Recursively find their dependents
3. Classify by risk tier:
   - **Tier 1 (Direct):** Immediately broken if interface changes
   - **Tier 2 (Transitive):** Broken via chain of dependency
   - **Tier 3 (Runtime):** Only broken at runtime (dynamic requires, plugin loading, reflection)

```
BLAST RADIUS: src/core/auth.ts
══════════════════════════════
Direct dependents (Tier 1):      N files
  src/api/login.ts
  src/middleware/session.ts
  ...

Transitive dependents (Tier 2):  N files
  src/pages/dashboard.tsx  (via middleware/session.ts)
  ...

Runtime dependents (Tier 3):     N files (estimated)

TOTAL BLAST RADIUS:  N files  →  [LOW | MEDIUM | HIGH | CRITICAL]
Public API surface exposed:      N endpoints
Test coverage of dependents:     N% (estimated — flag if unavailable)
```

**Blast radius thresholds:**

| Blast Radius | Direct Dependents | Risk Level |
|---|---|---|
| LOW | 0–3 | Safe to modify with tests |
| MEDIUM | 4–10 | Proceed carefully, thorough tests required |
| HIGH | 11–25 | Major surgery, requires incremental approach |
| CRITICAL | 26+ | Load-bearing pillar — do not modify without isolation strategy |

---

### Phase 4: Fragility Scoring

*Combine churn, coupling, blast radius, and test coverage into a single risk score per module.*

**Fragility Score formula (0–100):**
```
Fragility = (Churn Score × 0.30)
          + (Coupling Score × 0.25)
          + (Blast Radius Score × 0.25)
          + (Test Gap Score × 0.20)
```

Where:
- **Churn Score (0–10):** normalized rank from Phase 2 churn table
- **Coupling Score (0–10):** implicit coupling pairs count × severity
- **Blast Radius Score (0–10):** LOW=2, MEDIUM=5, HIGH=8, CRITICAL=10
- **Test Gap Score (0–10):** estimated test coverage gap (no tests=10, full coverage=0)

When test coverage data is unavailable, use a conservative heuristic: scan for test files co-located with the module. If none found, assume Test Gap Score = 8. Flag this assumption explicitly in the report.

**Risk bands:**

| Score | Band | Meaning |
|---|---|---|
| 0–20 | 🟢 STABLE | Low risk. Changes are predictable and contained. |
| 21–40 | 🟡 CAUTION | Moderate risk. Add tests before touching. |
| 41–65 | 🟠 FRAGILE | High risk. Isolate and test thoroughly before changing. |
| 66–85 | 🔴 DANGER | Very high risk. Requires isolation strategy + feature flags. |
| 86–100 | ⛔ CRITICAL | Do not modify without full regression harness. Consider strangler fig. |

**Output format:**
```
FRAGILITY SCORES
════════════════
Module                      Churn   Coupling  BlastR  TestGap   SCORE   BAND
──────                      ─────   ────────  ──────  ───────   ─────   ────
src/core/auth.ts             8.2      7.1      8.0     9.0       81    🔴 DANGER
src/api/payments.ts          6.1      4.0      5.0     8.5       60    🟠 FRAGILE
src/utils/format.ts          2.0      1.0      2.0     3.0       20    🟢 STABLE
...

TOP 5 MOST FRAGILE MODULES
1. src/core/auth.ts       (81) — 🔴 DANGER
2. ...
```

---

### Phase 5: Safe Change Strategy

*Translate all findings into a concrete, prioritized plan for safely approaching this codebase.*

**For each DANGER / CRITICAL module, recommend one of:**

| Strategy | When to Use |
|---|---|
| **Strangler Fig** | High blast radius, no clear seam. Wrap the module, redirect traffic incrementally. |
| **Characterization Tests** | No existing tests. Write tests capturing current behavior before any change. |
| **Isolation Layer** | Implicitly coupled to many things. Add an interface/adapter to decouple first. |
| **Feature Flag Guard** | Module is on a hot path. Gate all changes behind a flag; deploy before activating. |
| **Parallel Run** | Business-critical logic. Run old + new implementations side-by-side, compare output. |
| **Incremental Extract** | God object. Extract one responsibility at a time, one PR per extraction. |

**Entry point recommendation:**

Given all findings, recommend WHERE to start:
1. Highest fragility modules → what **not** to touch first
2. Stable modules that sit at the edge of fragile modules → start here (build test harness from the outside in)
3. Implicit coupling pairs → separate before modifying either side
4. Bug attractors → need characterization tests before any new work proceeds

**Safe change sequence:**
```
RECOMMENDED CHANGE SEQUENCE
════════════════════════════
Step 1 — BEFORE touching anything:
  [ ] Write characterization tests for: [list DANGER/CRITICAL modules]
  [ ] Add feature flag infrastructure if missing

Step 2 — Low-risk foundations (safe to start now):
  [ ] [file]  →  [why it's safe]  →  [suggested strategy]

Step 3 — Medium-risk work (after Step 2 tests are green):
  [ ] [file]  →  [strategy]  →  [verification criteria]

Step 4 — High-risk surgery (only with full harness in place):
  [ ] [file]  →  [strategy]  →  [specific isolation approach]

AVOID until last:
  ⛔ [file]  —  [reason]
```

---

### Phase 6: Brief Synthesis & Output

Compile all findings into **`brief.md`** at the root of the project (or inside `--scope` directory if scoped).

**`brief.md` structure:**

```markdown
# Codebase Recon Brief

**Generated:** [ISO timestamp]
**Scope:** [path or "full project"]
**Git window:** [date range analyzed]

---

## TL;DR

[3–5 bullet executive summary. The most important things anyone needs to know
 before touching this codebase.]

## Architecture Overview

[Phase 1 output — architecture map, layers, integration points, known tech debt]

## Risk Map

[Phase 4 output — fragility scores table, top 5 most fragile modules with risk band]

## Git Hotspots

[Phase 2 output — top churning files, implicit coupling pairs, bug attractors]

## Blast Radius Ledger

[Phase 3 output — PILLAR vs LEAF classification, blast radius for key modules]

## Safe Change Strategy

[Phase 5 output — recommended change sequence, strategy per DANGER/CRITICAL module]

## Known Tech Debt

[Phase 1 archaeology — explicit debt from code comments, docs, long-lived TODOs]

## Appendix: Raw Metrics

[Full fragility score table, full churn table, full coupling table]
```

**`brief.md` MUST be written to disk.** After writing, print the full file path and a TL;DR of the top findings to the conversation.

---

## Confidence Calibration

| Score | Meaning |
|---|---|
| 9–10 | Verified by reading code + git log. Metrics computed from actual data. |
| 7–8 | High confidence. Pattern clearly evident, minor gaps in coverage. |
| 5–6 | Moderate. Static analysis only, some assumptions made. Flag with caveat. |
| 3–4 | Low. Inferred from structure, not deeply verified. |
| 1–2 | Speculation. Only include if the consequence would be CRITICAL. |

Fragility scores are **estimates, not measurements.** Err toward a higher score when data is incomplete — it is always safer to overstate risk than to understate it.

## Key Principles

- **Measure before judging** — Opinions about code quality are cheap. Churn data, blast radius, and coupling metrics are not. Base every claim on evidence.
- **Respect the load-bearing pillars** — High fan-in modules may look messy but exist for a reason. Don't recommend removing them; recommend isolating them.
- **Churn is the real signal** — A complex file that never changes is safer than a simple file that breaks weekly. Prioritize churn data over aesthetic code quality.
- **Blast radius shapes strategy** — The right change strategy depends entirely on how many things depend on what you're changing. Calculate it; don't guess.
- **Test gaps amplify all other risks** — Untested code with high churn and high blast radius is the most dangerous combination. Flag it loudly.
- **Implicit coupling is invisible debt** — Files that always change together are coupled even if the import graph says otherwise. This coupling must be made explicit before any large refactor.
- **brief.md is the deliverable** — Insights in conversation are ephemeral. Write the brief. Future teammates will thank you.

## Anti-Patterns

- **Aesthetic archaeology**: Flagging code as fragile because it looks old or messy, without churn/coupling evidence to back it up
- **Blast radius theater**: Claiming high blast radius without tracing the actual import or call graph
- **Git log skimming**: Sampling 5 commits and extrapolating churn — use the full window
- **Recommending full rewrites**: Rewrite recommendations belong in a proposal. Recon produces intelligence, not verdicts.
- **Ignoring dormant code**: Old, unmodified code often contains critical business logic. "Nobody touches it" may mean "it works perfectly," not "it's dead."
- **Overconfident fragility scores**: Test coverage data is rarely derivable from static analysis alone — flag every assumption used in scoring

## Completion Status

- **RECON_COMPLETE** — All phases done, brief.md written, key findings summarized in conversation
- **RECON_PARTIAL** — Some phases incomplete (no git history, no dep manifest, etc.) — brief.md written with gaps explicitly noted
- **RECON_TARGET_COMPLETE** — `--target` or scoped run complete, blast radius and change strategy for specific module ready
- **BLOCKED** — Cannot proceed (no git repo, no file access, insufficient project context)
- **NEEDS_CONTEXT** — Missing critical information to continue (specify exactly what is needed)
