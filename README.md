# Magic Spec

**Spec-driven development skills for AI coding agents.**

English | [中文](./README.zh-CN.md) | [日本語](./README.ja.md)

Turn vague ideas into shipped features through a structured workflow: Explore → Propose → Apply → Review. Every step is traceable, testable, and self-reflective.

```
 Explore          Propose              Apply             Review
┌─────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ Idea     │───▶│ proposal.md  │───▶│ RED-GREEN-   │───▶│ Code Review  │
│ Analysis │    │ specs/       │    │ REFACTOR     │    │ Security     │
│ Research │    │ design.md    │    │ Self-reflect │    │ Audit        │
│          │    │ test-design  │    │ Verify       │    │              │
│          │    │ tasks.md     │    │ Archive      │    │              │
└─────────┘    └──────────────┘    └──────────────┘    └──────────────┘
```

## Why Spec-Driven?

AI coding agents are powerful, but without structure they produce inconsistent results. They skip tests, forget requirements, drift from the design, and declare "done" without evidence.

magic-spec solves this with **5 skills** that enforce discipline at every stage:

| Problem | How magic-spec solves it |
|---------|------------------------|
| Building the wrong thing | `/magic-explore` investigates before committing |
| Vague requirements | `/magic-proposal` creates formal specs with Given/When/Then scenarios |
| Skipped tests | `/magic-proposal` designs tests BEFORE implementation |
| Drifting from the plan | `/magic-apply` self-reflects after every task against design.md |
| "It should work" claims | `/magic-apply` requires fresh verification evidence for every checkbox |
| Missed security issues | `/magic-security-review` runs 12-phase CSO-level audit |
| Shallow code review | `/magic-code-review` dispatches parallel expert reviewers |

## Skills

### `/magic-explore` — Deep Exploration & Discovery

Investigate ideas, analyze code, compare approaches, or trace root causes — all before writing a single line of code.

**4 exploration modes:**
- **Idea Exploration** — Turn a vague idea into clear understanding
- **Codebase Investigation** — Map architecture, trace data flows, find patterns
- **Root Cause Analysis** — Systematic debugging with evidence chains and hypothesis testing
- **Approach Comparison** — Compare 2-3 options with concrete trade-offs

Produces insights, not artifacts. When understanding crystallizes, transitions to `/magic-proposal`.

### `/magic-proposal` — Structured Change Proposal

Transform understanding into a complete change proposal with 5 artifacts:

```
magic-spec/changes/<change-name>/
├── proposal.md        WHY and WHAT — scope, capabilities, impact
├── specs/             WHAT changes — delta specs (ADDED/MODIFIED/REMOVED)
├── design.md          HOW — architecture decisions, key files, trade-offs
├── test-design.md     HOW TO VERIFY — spec-to-test mapping, test cases
└── tasks.md           STEPS — ordered implementation checklist
```

**Key features:**
- **Delta specs** — Describe changes, not the entire system (works for brownfield projects)
- **Given/When/Then scenarios** — Every requirement has testable scenarios
- **Spec-to-test mapping** — Every requirement maps to concrete test cases before any code is written
- **Dependency graph** — Artifacts created in order: proposal → specs ←→ design → test-design → tasks

### `/magic-apply` — Spec-Driven Implementation

Execute tasks with TDD discipline and continuous self-reflection:

```
For each task:
  1. Read task context + test-design.md
  2. Write failing test (RED)
  3. Implement minimally (GREEN)
  4. Refactor (REFACTOR)
  5. Verify with evidence
  6. Self-reflect: design compliance? spec satisfaction? test coverage?
  7. Mark complete (only after reflection passes)
```

**Self-reflection is not optional.** After every task, the agent stops and checks:
- Does this match design.md?
- Are all spec scenarios covered by tests?
- Are the test-design.md test cases actually implemented?
- Am I cutting corners?

When all tasks complete, a **Final Self-Reflection** audits the entire change before archiving.

### `/magic-security-review` — Security Posture Audit

CSO-level automated security audit with 12 phases:

| Phase | What it checks |
|-------|---------------|
| 0 | Architecture mental model + stack detection |
| 1 | Attack surface census |
| 2 | Secrets archaeology (git history) |
| 3 | Dependency supply chain |
| 4 | CI/CD pipeline security |
| 5 | Infrastructure shadow surface |
| 6 | Webhook & integration audit |
| 7 | LLM & AI security |
| 8 | OWASP Top 10 |
| 9 | STRIDE threat model |
| 10 | False positive filtering + active verification |
| 11 | Data classification |

**Two modes:** Daily (8/10 confidence, zero noise) and Comprehensive (2/10 confidence, monthly deep scan).

### `/magic-code-review` — Multi-Expert Code Review

Two-stage review with parallel expert analysis:

**Stage 1: Spec Compliance** — Did we build what was requested? Requirement-by-requirement verification against the spec.

**Stage 2: Code Quality** — Is what we built well-crafted? Architecture, testing, performance, security, maintainability.

**Expert Panel** (parallel): Testing Expert, Security Expert, Performance Expert, Maintainability Expert — findings cross-validated and deduplicated.

## Installation (Claude Code)

### 1. Clone the repository

```bash
git clone https://github.com/anthropics/magic-spec.git
```

### 2. Add skills to your project

Copy or symlink the skills into your project's `.claude/skills/` directory:

```bash
# From your project root
mkdir -p .claude/skills

# Option A: Symlink (recommended — auto-updates when you pull)
ln -s /path/to/magic-spec/skills/magic-explore .claude/skills/magic-explore
ln -s /path/to/magic-spec/skills/magic-proposal .claude/skills/magic-proposal
ln -s /path/to/magic-spec/skills/magic-apply .claude/skills/magic-apply
ln -s /path/to/magic-spec/skills/magic-security-review .claude/skills/magic-security-review
ln -s /path/to/magic-spec/skills/magic-code-review .claude/skills/magic-code-review

# Option B: Copy
cp -r /path/to/magic-spec/skills/* .claude/skills/
```

### 3. Add skill routing to CLAUDE.md (recommended)

Add this to your project's `CLAUDE.md` to enable automatic skill invocation:

```markdown
## Skill routing

When the user's request matches an available skill, invoke it as your first action:

- Feature ideas, brainstorming, "how does X work" → invoke magic-explore
- "Build X", "add feature", "create a plan for" → invoke magic-proposal
- "Implement", "apply the plan", "start coding" → invoke magic-apply
- Security audit, "check for vulnerabilities" → invoke magic-security-review
- Code review, "review my changes" → invoke magic-code-review
```

## Usage Examples (Claude Code)

### Example 1: New Feature — Full Workflow

```
You: I want to add dark mode to the app

Claude: [invokes /magic-explore]
  → Explores current styling system, theme architecture, user preferences storage
  → Proposes 3 approaches with trade-offs
  → Recommends CSS custom properties approach
  → Confidence: 8/10

You: Looks good, let's go with that

Claude: [invokes /magic-proposal add-dark-mode]
  → Creates magic-spec/changes/add-dark-mode/
  → Writes proposal.md (why dark mode, what changes)
  → Writes specs/theme-switching.md (ADDED requirements with scenarios)
  → Writes design.md (CSS custom properties, ThemeProvider, localStorage)
  → Writes test-design.md (8 test cases mapped to requirements)
  → Writes tasks.md (14 tasks in 4 groups)
  → "Please review the artifacts before we proceed."

You: Approved, let's build it

Claude: [invokes /magic-apply add-dark-mode]
  → Task 1.1: Write failing test for ThemeProvider → RED
  → Task 1.1: Implement ThemeProvider → GREEN
  → Task 1.1: Refactor → clean up
  → Task 1.1: Self-reflection → design compliance ✓, spec coverage ✓
  → Task 1.2: Write failing test for toggle... (continues)
  → ...
  → Final Self-Reflection: all 14 tasks verified, design match ✓, spec coverage ✓
  → DONE

You: Review the code before we merge

Claude: [invokes /magic-code-review --spec add-dark-mode]
  → Stage 1: Spec compliance — all requirements met ✓
  → Stage 2: Code quality — 1 IMPORTANT (missing error boundary), 2 MINOR
  → VERDICT: FIX_REQUIRED — 1 important issue
```

### Example 2: Quick Security Check

```
You: /magic-security-review --diff

Claude: [runs 12-phase audit on current branch changes]
  → Phase 0: Detected Next.js + TypeScript stack
  → Phase 2: No secrets in git history ✓
  → Phase 7: Found user input flowing into system prompt (CRITICAL, 9/10)
  → Phase 8: Missing rate limit on auth endpoint (HIGH, 8/10)
  → 2 findings, 0 false positives
```

### Example 3: Investigate Before Fixing

```
You: The login page is broken after the last deploy

Claude: [invokes /magic-explore in Root Cause Analysis mode]
  → Phase 1: Reads error logs, traces the auth flow
  → Phase 2: Finds working examples, compares with broken code
  → Phase 3: Hypothesis — session token format changed in dependency update
  → Verified: @auth/core 5.x changed token serialization
  → Confidence: 9/10, ready to propose fix

You: Fix it

Claude: [invokes /magic-proposal fix-auth-token]
  → Creates targeted proposal with 1 spec, 3 tasks
  → Test-design maps regression test to the exact failure
  ...
```

## Architecture

### Change Lifecycle

```
PLANNING                    IMPLEMENTATION                 COMPLETION
────────                    ──────────────                 ──────────
/magic-explore              /magic-apply                   /magic-code-review
     │                           │                              │
     ▼                           ▼                              ▼
/magic-proposal             Per-task loop:                 Spec compliance
     │                      RED → GREEN →                  Code quality
     ▼                      REFACTOR →                     Expert panel
 proposal.md                Verify →                            │
 specs/                     Self-reflect                        ▼
 design.md                       │                         Findings + Fix
 test-design.md                  ▼
 tasks.md                   Final self-reflection
                                 │
                                 ▼
                            Archive to
                            changes/archive/
```

### Delta Specs

Unlike traditional specs that describe the entire system, magic-spec uses **delta specs** — they describe only what changes:

```markdown
## ADDED Requirements
New behaviors that didn't exist before.

## MODIFIED Requirements
Changed behaviors (includes the complete updated behavior, not just the diff).

## REMOVED Requirements
Behaviors being retired (includes migration guidance).
```

This makes specs practical for real projects where you're always changing an existing system, not describing one from scratch.

### Self-Reflection

magic-apply includes two levels of introspection:

**Per-task reflection** (after every single task):
```
TASK 1.1 REFLECTION
  Design compliance:  ✓ Matches design.md decisions
  Spec satisfaction:   ✓ All scenarios covered
  Test coverage:       ✓ All test-design.md cases written
  Confidence:          8/10
  → MARKING COMPLETE
```

**Final reflection** (after all tasks):
```
FINAL REFLECTION: add-dark-mode
═══════════════════════════════
Tasks:           14/14 (all verified: yes)
Design match:    ✓
Spec coverage:   ✓
Test coverage:   ✓
Regressions:     ✓ none
Overall:         DONE
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Use magic-spec's own skills to propose and implement changes
4. Submit a PR

## License

MIT
