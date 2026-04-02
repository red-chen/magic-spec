---
name: magic-code-review
description: "Use for Staff Engineer level code review with multi-expert parallel analysis. Two-stage review: spec compliance verification, then code quality assessment. Dispatches specialized reviewers for testing, security, performance, maintainability."
---

# /magic-code-review — Multi-Expert Code Review

Staff Engineer level automated code review combining spec compliance verification with multi-dimensional quality assessment. Multiple expert reviewers analyze the code in parallel, findings are cross-validated, and actionable feedback is provided.

**Philosophy:** Review early, review often. A 10-minute review catches issues that cost 10 hours to fix in production. Never skip review because "it's simple."

## When to Use

**Mandatory:**
- After completing `/magic-apply` (all tasks done)
- Before merging any feature branch to main
- After any significant code change

**Recommended:**
- After each task batch during implementation
- When stuck (fresh perspective)
- Before refactoring (baseline check)
- After fixing a complex bug

## Arguments

- `/magic-code-review` — Review current branch changes against base
- `/magic-code-review --spec [change-name]` — Review against a specific proposal's specs
- `/magic-code-review --quick` — Fast review (skip parallel experts, single-pass)
- `/magic-code-review --scope [path]` — Review only files under a specific path
- `/magic-code-review --base [ref]` — Custom base reference (default: main/master)

## Hard Gate

<HARD-GATE>
This skill is READ-ONLY during its review phases. Do NOT modify code during review. Collect all findings first, present them, and let the user decide what to fix. The only exception is the Fix Phase at the end, when the user explicitly approves fixes.
</HARD-GATE>

## Review Architecture

The review has two stages and a parallel expert layer:

```
Stage 1: Spec Compliance Review
    "Did we build what was requested?"
         ↓
Stage 2: Code Quality Review
    "Is what we built well-crafted?"
         ↓
Expert Panel (parallel)
    Testing | Security | Performance | Maintainability
         ↓
Cross-Validation & Synthesis
         ↓
Findings Report
         ↓
Fix Phase (user-approved)
```

## Stage 1: Spec Compliance Review

**Goal:** Verify the implementation matches the spec exactly — nothing more, nothing less.

### Process

1. **Load the spec:** Read `magic-spec/changes/<name>/specs/` and `proposal.md`
2. **Load the diff:** Get the full diff between base and HEAD
3. **Requirement-by-requirement check:**
   - For each ADDED requirement in specs: Is it implemented? Is the test present?
   - For each MODIFIED requirement: Does the change match the spec's new behavior exactly?
   - For each REMOVED requirement: Is the old code actually removed? Migration applied?
4. **Scope creep detection:** Are there changes NOT covered by any spec requirement? Flag them.
5. **Missing implementation:** Are there spec requirements with no corresponding code?

**CRITICAL: Do NOT trust any self-assessment.** Read the actual code independently. Verify each claim by reading the source.

**Output:**
```
SPEC COMPLIANCE
═══════════════
Requirement  Status       Notes
───────────  ──────       ─────
REQ-001      ✓ COMPLIANT  Implemented in auth.ts:42, test in auth.test.ts:15
REQ-002      ✗ PARTIAL    Missing edge case: empty input handling
REQ-003      ✓ COMPLIANT  Removed as specified, migration applied
SCOPE        ⚠ EXTRA      utils/helper.ts:20 — unrelated refactoring not in spec
```

If spec compliance fails → report issues and STOP. Do not proceed to Stage 2 until specs are met. The user must fix spec compliance issues first.

## Stage 2: Code Quality Review

**Goal:** Assess the craftsmanship of the implementation.

### Review Dimensions

**Architecture & Design:**
- Separation of concerns — each module has one clear purpose
- Dependency direction — no circular dependencies, proper layering
- Interface design — clear contracts between components
- Error boundaries — errors handled at appropriate levels

**Code Quality:**
- Naming — clear, consistent, descriptive
- Complexity — functions under 40 lines, cyclomatic complexity reasonable
- Duplication — DRY applied appropriately (not over-abstracted)
- Type safety — proper types, no `any` escape hatches without justification

**Error Handling:**
- All error paths covered
- Errors are descriptive and actionable
- No swallowed errors (empty catch blocks)
- Graceful degradation where appropriate

**Testing:**
- Tests verify behavior, not implementation
- Edge cases covered (from spec scenarios)
- No mocking of the thing under test
- Tests are readable and maintainable
- Test names describe the behavior being verified

**Performance:**
- No obvious N+1 queries
- No unnecessary re-renders or recomputation
- Appropriate caching where needed
- No blocking operations on hot paths

**Security (quick check — for deep analysis use `/magic-security-review`):**
- No SQL injection vectors
- No command injection
- Input validation present
- Auth checks on protected routes
- No hardcoded secrets

## Expert Panel (Parallel Analysis)

For thorough reviews (non-`--quick`), dispatch parallel expert analyses:

### Testing Expert
- Test coverage completeness against spec scenarios
- Test quality (behavior vs implementation testing)
- Missing edge cases and boundary conditions
- Test isolation and reliability

### Security Expert
- Input validation and sanitization
- Authentication and authorization checks
- Data exposure risks
- Injection vulnerabilities in changed code

### Performance Expert
- Query efficiency and N+1 patterns
- Memory allocation patterns
- Caching opportunities
- Hot path analysis

### Maintainability Expert
- Code readability and clarity
- Documentation completeness
- Naming consistency
- Dependency management

Each expert produces findings independently. Cross-validation happens in the synthesis phase.

## Cross-Validation & Synthesis

Merge findings from all reviewers:

1. **Deduplication:** Multiple reviewers flagging the same issue → merge into one finding
2. **Confidence boost:** Finding flagged by 2+ reviewers → increase confidence
3. **Conflict resolution:** Reviewers disagree → present both perspectives to user
4. **Severity calibration:** Normalize severity across reviewers

## Findings Report

### Issue Severity

| Severity | Meaning | Action |
|----------|---------|--------|
| **CRITICAL** | Blocks merge. Security vulnerability, data loss risk, or spec violation. | Must fix before merge |
| **IMPORTANT** | Should fix. Maintainability issue, missing test, or performance concern. | Fix before proceeding |
| **MINOR** | Nice to have. Style preference, optimization opportunity. | Fix when convenient |
| **PRAISE** | Well done. Particularly clean implementation or clever solution. | Acknowledge |

### Finding Format

```
## Finding N: [Title]

* **Severity:** CRITICAL | IMPORTANT | MINOR | PRAISE
* **Confidence:** N/10
* **Source:** [Stage 1/2, Expert name, or multiple]
* **File:** path/to/file.ts:42
* **Category:** [Spec Compliance | Architecture | Testing | Security | Performance | Maintainability]
* **Description:** [What's wrong / what's great]
* **Suggestion:** [Specific fix with code example]
* **Effort:** [Trivial | Small | Medium | Large]
```

### Summary Table

```
CODE REVIEW SUMMARY
═══════════════════
Branch: feature/add-dark-mode → main
Files changed: 12
Lines: +342 / -87

FINDINGS
#   Sev        Conf   Category        Finding                          File:Line
──  ─────────  ────   ────────        ───────                          ─────────
1   CRITICAL   9/10   Spec            Missing empty input handling      auth.ts:42
2   IMPORTANT  8/10   Testing         Edge case not covered             auth.test.ts
3   IMPORTANT  7/10   Performance     N+1 query in user list            users.ts:88
4   MINOR      6/10   Maintainability Magic number needs constant       config.ts:15
5   PRAISE     10/10  Architecture    Clean separation of concerns      api/routes/

VERDICT: ⚠ FIX REQUIRED — 1 CRITICAL, 2 IMPORTANT issues before merge
```

### Verdict

- **APPROVED** — No critical or important issues. Ready to merge.
- **APPROVED_WITH_NOTES** — No critical issues, minor suggestions. Merge at discretion.
- **FIX_REQUIRED** — Critical or important issues must be addressed.
- **RETHINK** — Fundamental approach issues. Consider redesign.

## Fix Phase

After presenting findings, offer to fix issues:

**Auto-fixable:** Issues with clear, mechanical fixes (missing type annotation, unused import, simple refactor)
- Present the fix: "I can auto-fix these N issues. Approve?"
- Apply fixes only with user approval

**Manual review needed:** Issues requiring judgment (architecture changes, approach decisions)
- Present options with trade-offs
- User decides direction

**Fix priority:**
1. CRITICAL issues (must fix)
2. IMPORTANT issues (should fix)
3. MINOR issues (at user's discretion)

After fixes, re-run relevant verification to confirm nothing broke.

## Confidence Calibration

| Score | Meaning |
|-------|---------|
| 9-10 | Verified by reading code. Concrete issue with specific fix. |
| 7-8 | High confidence. Strong pattern match. |
| 5-6 | Moderate. Might be intentional or contextual. |
| 3-4 | Low. Possibly a false positive. Include caveat. |

## Pushback Guidelines

If you disagree with a finding or the user pushes back:

- **Valid pushback:** User provides technical reasoning or test evidence → accept and note
- **Invalid pushback:** "It's fine" / "Ship it" without evidence → maintain finding
- **Honest disagreement:** Present both sides, let user decide with full information

## Key Principles

- **Read the code, don't trust reports** — Verify everything independently
- **Spec compliance first** — No point reviewing quality if the wrong thing was built
- **Concrete over abstract** — Name files, lines, and specific fixes
- **Proportional review** — Quick for small changes, thorough for large ones
- **Praise good work** — Acknowledge clean code, not just flag problems
- **Fix priority** — CRITICAL first, MINOR last. Don't block merges on style preferences.

## Completion Status

- **DONE** — Review complete, all findings reported
- **DONE_WITH_CONCERNS** — Review complete, but some areas couldn't be fully analyzed
- **BLOCKED** — Cannot review (no diff, no base branch, etc.)
- **NEEDS_CONTEXT** — Missing spec or context to perform spec compliance check
