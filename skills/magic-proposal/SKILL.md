---
name: magic-proposal
description: "Use when ready to formalize an idea into a structured change proposal with specs, design, and tasks. Creates all planning artifacts in one step. Follows exploration or can be invoked directly for well-understood changes."
---

# /magic-proposal — Structured Change Proposal

Transform understanding into a complete, actionable change proposal. Creates a change folder with all planning artifacts: proposal, delta specs, technical design, and implementation tasks.

**Philosophy:** A good proposal is a contract between intent and implementation. It captures WHY (proposal), WHAT changes (delta specs), HOW to build it (design), HOW to verify it (test design), and the exact STEPS to execute (tasks).

## When to Use

- After `/magic-explore` when understanding has crystallized
- Directly for well-understood changes that don't need exploration
- When formalizing a feature request, bug fix, or refactoring plan
- Before any significant code change (more than a trivial one-liner)

## Arguments

- `/magic-proposal [change-description]` — Create a new change proposal
- `/magic-proposal --change [name]` — Resume or edit an existing proposal

## Hard Gate

<HARD-GATE>
Do NOT write any implementation code during proposal creation. This skill produces PLANNING artifacts only: proposal, specs, design, and tasks. Implementation happens in `/magic-apply`.
</HARD-GATE>

## Change Folder Structure

Every change lives in its own isolated folder:

```
magic-spec/changes/<change-name>/
├── .magic-spec.yaml          # Change metadata
├── proposal.md               # WHY and WHAT (scope, capabilities, impact)
├── specs/                    # Delta specs (ADDED/MODIFIED/REMOVED)
│   ├── <capability-1>.md
│   └── <capability-2>.md
├── design.md                 # HOW (architecture, decisions, trade-offs)
├── test-design.md            # HOW TO VERIFY (test strategy, coverage, scenarios)
└── tasks.md                  # Implementation checklist (ordered, testable)
```

## Artifact Dependency Graph

```
proposal (no dependencies — created first)
    ↓
specs  ←→  design  (both depend on proposal, can be created in parallel)
    ↓        ↓
    └── test-design  (depends on specs + design — knows WHAT to verify and HOW it's built)
              ↓
           tasks  (depends on specs, design, AND test-design)
```

Dependencies are **enablers**, not **gates**. You can skip artifacts that aren't needed for simple changes, but the dependency order determines creation sequence.

## Workflow

### Step 1: Create Change Folder

```
magic-spec/changes/<change-name>/
└── .magic-spec.yaml
```

The metadata file:
```yaml
name: <change-name>
created: <ISO-8601-datetime>
status: planning
```

### Step 2: Write Proposal

The proposal defines WHY this change exists and WHAT it touches.

**Template:**

```markdown
# Proposal: <Change Name>

## Why

[Problem statement. What's broken, missing, or suboptimal? Why now?]

## What Changes

[High-level scope. Which parts of the system are affected?]

## Capabilities

Each capability below maps to one delta spec file:

- **<capability-1>**: [Brief description of new/changed behavior]
- **<capability-2>**: [Brief description of new/changed behavior]

## Impact

- **Users affected**: [Who sees this change?]
- **Breaking changes**: [Any backward-incompatible changes?]
- **Dependencies**: [External dependencies or prerequisites?]
```

**Key principle:** The Capabilities section is the bridge to specs. Each capability listed here becomes a separate spec file with detailed requirements and scenarios.

### Step 3: Write Delta Specs

Delta specs describe WHAT changes in the system's behavior. They use an incremental format — describing changes, not restating the entire system.

**For each capability listed in the proposal, create a spec file:**

```markdown
# <Capability Name>

## ADDED Requirements

### REQ-001: <Requirement Title>

The system SHALL [behavior description].

**Scenario: <scenario name>**
- Given [precondition]
- When [action]
- Then [expected outcome]

## MODIFIED Requirements

### REQ-002: <Requirement Title>

The system SHALL [complete updated behavior — not just the diff].

**Previous behavior:** [What it did before]
**New behavior:** [What it does now]

**Scenario: <scenario name>**
- Given [precondition]
- When [action]
- Then [expected outcome]

## REMOVED Requirements

### REQ-003: <Requirement Title>

This requirement is removed because [reason].

**Migration:** [How existing users/code should adapt]
```

**Spec writing rules:**
- Use RFC 2119 keywords: MUST, SHALL, SHOULD, MAY for requirement strength
- Every requirement has 1+ scenarios in Given/When/Then format
- MODIFIED requirements include the COMPLETE updated behavior, not just changes
- REMOVED requirements include migration guidance
- Keep specs focused on WHAT, not HOW — implementation details go in design.md

### Step 4: Write Technical Design

The design explains HOW to implement the specs.

**Template:**

```markdown
# Design: <Change Name>

## Context

[Current architecture relevant to this change. Key files and components.]

## Goals

- [Concrete technical goal]

## Non-Goals

- [Explicitly out of scope]

## Decisions

### Decision 1: <Title>

**Options considered:**
1. [Option A] — [trade-offs]
2. [Option B] — [trade-offs]

**Chosen:** Option [X] because [reasoning].

## Architecture

[How the pieces fit together. Component diagram if helpful.]

## Key Files

| File | Change Type | Description |
|------|------------|-------------|
| `path/to/file.ts` | Modified | [What changes] |
| `path/to/new.ts` | New | [What it does] |

## Risks & Trade-offs

- [Risk 1]: [Mitigation]
- [Trade-off 1]: [Why it's acceptable]

## Open Questions

- [Question that needs resolution during implementation]
```

**Design principles:**
- Focus on architecture and decisions, NOT line-by-line implementation
- Name specific files, functions, and libraries
- Show real numbers for performance trade-offs
- Design for isolation: smaller units with clear purposes and well-defined interfaces
- Each unit should be understandable and testable independently

### Step 5: Write Test Design

The test design defines HOW to verify that the implementation satisfies the specs. It bridges requirements to concrete test cases — ensuring nothing is left to assumption during `/magic-apply`.

**Template:**

```markdown
# Test Design: <Change Name>

## Test Strategy

- **Test framework:** [e.g., Vitest, Jest, Pytest, Go testing]
- **Coverage target:** [e.g., all ADDED/MODIFIED requirements, edge cases from scenarios]
- **Test levels:** [unit | integration | e2e — which levels apply to this change]

## Spec-to-Test Mapping

Every spec requirement MUST have at least one test. Map them explicitly:

| Requirement | Spec File | Test File | Test Type | Scenarios Covered |
|------------|-----------|-----------|-----------|-------------------|
| REQ-001    | specs/capability-1.md | tests/capability-1.test.ts | Unit | Happy path, empty input |
| REQ-002    | specs/capability-1.md | tests/capability-1.test.ts | Unit | Previous→New behavior |
| REQ-003    | specs/capability-2.md | tests/integration/cap-2.test.ts | Integration | Full flow |

## Test Cases

### <Requirement ID>: <Title>

**Happy path:**
- Given [precondition from spec scenario]
- When [action]
- Then [assertion — concrete expected value]

**Edge cases:**
- [Input boundary]: [expected behavior]
- [Error condition]: [expected error/fallback]
- [Empty/null input]: [expected handling]

**Regression guard:**
- [What existing behavior MUST NOT break]
- [Specific regression test if modifying existing code]

## Test Environment

- **Setup required:** [fixtures, seed data, mocks, stubs]
- **External dependencies:** [what to mock vs. what to test live]
- **Cleanup:** [teardown steps if needed]

## Verification Commands

```bash
# Run all tests for this change
<exact command>

# Run specific test file
<exact command>

# Run with coverage
<exact command>
```
```

**Test design rules:**
- Every ADDED/MODIFIED requirement from specs/ MUST appear in the mapping table
- Each spec scenario (Given/When/Then) becomes at least one concrete test case
- Edge cases are explicitly identified — don't rely on "the developer will think of them"
- Regression guards protect existing behavior when MODIFYING requirements
- Test files and commands are concrete paths — no placeholders
- The test design is the contract that `/magic-apply` follows during RED-GREEN-REFACTOR

### Step 6: Write Task List

Break the design into ordered, testable implementation steps.

**Template:**

```markdown
# Tasks: <Change Name>

## Group 1: <Category>

- [ ] 1.1 <Task description>
  - Files: `path/to/file.ts`
  - Verify: <How to verify this task is complete>

- [ ] 1.2 <Task description>
  - Files: `path/to/file.ts`, `path/to/test.ts`
  - Verify: <Verification command or criteria>

## Group 2: <Category>

- [ ] 2.1 <Task description>
  - Files: `path/to/file.ts`
  - Verify: <Verification>
```

**Task writing rules:**
- Each task is small enough to complete in one focused session
- Tasks are ordered by dependency (can't do 1.2 before 1.1)
- Group related tasks under meaningful headers
- Every task has explicit verification criteria
- For every implementation task, include a corresponding test task BEFORE it (from test-design.md)
- Task pairs follow RED-GREEN: write test (RED) → implement (GREEN) → refactor
- NO placeholders: every task has concrete file paths and expected outcomes
- Use markdown checkboxes (`- [ ]`) for progress tracking by `/magic-apply`
- Reference the specific test cases from test-design.md in each test task

## Self-Review

After writing all artifacts, perform a quick self-review:

1. **Placeholder scan:** Any "TBD", "TODO", incomplete sections, or vague requirements? Fix them.
2. **Internal consistency:** Do specs match the proposal's capabilities? Does design address all specs? Does test-design cover all specs? Do tasks cover the full design?
3. **Spec-Test coverage:** Every ADDED/MODIFIED requirement has a test case in test-design.md? Every test case has a corresponding task in tasks.md?
4. **Scope check:** Is this focused enough for a single implementation cycle?
5. **Ambiguity check:** Could any requirement be interpreted two ways? Pick one and make it explicit.
6. **Completeness:** Does every spec requirement have scenarios? Does every task have verification? Does every test case have concrete assertions?

Fix issues inline. No need for a separate review pass.

## User Review Gate

After all artifacts are written:

> "Proposal complete at `magic-spec/changes/<name>/`. Please review the artifacts:
> - `proposal.md` — scope and intent
> - `specs/` — detailed requirements
> - `design.md` — technical approach
> - `test-design.md` — test strategy and spec-to-test mapping
> - `tasks.md` — implementation plan
>
> Let me know if you want changes before we move to `/magic-apply`."

Wait for approval. If changes are requested, make them. Only proceed to apply after explicit approval.

## Quick Path: Fast-Forward

For simple, well-understood changes, create all artifacts in a single pass:

```
/magic-proposal add-dark-mode
```

This creates the change folder and generates proposal → specs → design → test-design → tasks in dependency order, stopping for user review at the end.

## Key Principles

- **Propose before building** — Even "simple" changes benefit from 5 minutes of structured thinking
- **Design tests before code** — Test design at proposal time prevents "I'll add tests later" syndrome
- **Delta over full** — Specs describe changes, not the entire system (supports brownfield development)
- **Concrete over vague** — Name files, functions, line numbers. No hand-waving.
- **YAGNI ruthlessly** — Remove capabilities that aren't directly needed
- **Tasks are testable** — If you can't verify it, it's not a real task
- **Every spec has a test** — The spec-to-test mapping is the verification contract
- **Isolation** — Each capability is its own spec file; changes don't interfere

## Completion Status

- **PROPOSAL_COMPLETE** — All artifacts written and self-reviewed, awaiting user approval
- **PROPOSAL_APPROVED** — User approved, ready for `/magic-apply`
- **NEEDS_REVISION** — User requested changes to artifacts
- **BLOCKED** — Cannot proceed, missing critical information
