---
name: magic-apply
description: "Use when ready to implement a proposal. Reads the task list, executes tasks one by one with TDD discipline, tracks progress via checkboxes, and verifies each step with evidence before proceeding."
---

# /magic-apply — Spec-Driven Implementation

Execute a proposal's tasks with test-driven discipline, progress tracking, and evidence-based verification. Every task follows RED-GREEN-REFACTOR and is verified before marking complete.

**Philosophy:** Implementation is the disciplined execution of a plan — with continuous self-reflection. No shortcuts, no skipped tests, no "it should work" claims. Every checkbox earned with evidence. After every task, stop and ask: does this actually meet the design?

## When to Use

- After `/magic-proposal` has created and approved a change proposal
- To resume implementation of a partially completed change
- To implement the next batch of tasks from an existing plan

## Arguments

- `/magic-apply` — Apply the most recent active change
- `/magic-apply [change-name]` — Apply a specific change by name
- `/magic-apply --continue` — Resume from the last incomplete task
- `/magic-apply --status` — Show current progress without executing

## Hard Gates

<HARD-GATE>
1. Do NOT start implementation without approved proposal artifacts (proposal.md + tasks.md minimum)
2. Do NOT skip tests — every task that changes behavior gets a test
3. Do NOT mark a task complete without running verification
4. Do NOT modify specs or design during apply — if changes are needed, pause and update the proposal first
</HARD-GATE>

## Pre-Flight Check

Before executing any tasks, verify readiness:

1. **Change folder exists**: `magic-spec/changes/<name>/`
2. **Required artifacts present**: proposal.md, tasks.md, and test-design.md must exist
3. **Tasks parseable**: tasks.md contains valid checkbox items (`- [ ]` / `- [x]`)
4. **Test-design loaded**: Read test-design.md to understand the spec-to-test mapping
5. **Progress state**: Count total/complete/remaining tasks

**Status display:**
```
CHANGE: <name>
STATUS: <ready | blocked | all_done>
PROGRESS: <complete>/<total> tasks (<percentage>%)
NEXT TASK: <task-id> — <description>
ARTIFACTS:
  ✓ proposal.md
  ✓ specs/ (3 files)
  ✓ design.md
  ✓ test-design.md (8 test cases mapped)
  ✓ tasks.md (12 tasks, 4 complete)
```

If any required artifact is missing → status = `blocked`, explain what's needed.
If all tasks are complete → status = `all_done`, suggest `/magic-code-review` or archiving.

## Task Execution Loop

For each incomplete task, follow this cycle:

### 1. Read Task Context

- Read the task description and verification criteria from tasks.md
- Read relevant sections from specs/ and design.md for context
- Read the corresponding test cases from test-design.md
- Identify the files that need to change

### 2. Write Failing Test First (RED)

Before writing any implementation code:
- Write the test cases specified in test-design.md for this task
- Run the test — it MUST fail (if it passes, the behavior already exists)
- If you can't write a test for this task, document WHY (infrastructure, config, etc.)

**Test quality rules:**
- Test behavior, not implementation details
- Use real assertions, not just "no error thrown"
- Cover edge cases identified in the spec scenarios AND test-design.md
- No mocking of the thing being tested — mock dependencies only

### 3. Implement Minimally (GREEN)

- Write the minimum code to make the test pass
- Follow existing code patterns in the codebase
- Keep changes focused on the current task only
- Run the test — it MUST pass

### 4. Refactor (REFACTOR)

- Clean up without changing behavior
- Verify tests still pass after refactoring
- Check for duplication, unclear names, unnecessary complexity

### 5. Verify with Evidence

**Iron Law: No completion claims without fresh verification evidence.**

Run the verification command specified in the task:
- Tests: Run the full relevant test suite, confirm 0 failures
- Build: Run build command, confirm exit 0
- Integration: Run the specific scenario from the spec
- Manual: Document exact steps taken and outcomes observed

**Verification gate:**
- Identify verification command → RUN it fresh → READ full output → CONFIRM the claim

**Red flags that indicate skipping verification:**
- "This should work because..."
- "The test covered this already..."
- "It's just a config change..."
- "I'm pretty confident this is right..."

If you catch yourself thinking any of these → STOP and run the verification.

### 6. Self-Reflection (Per-Task Introspection)

**After every task, STOP and reflect before marking it complete.** This is not optional.

Ask yourself these questions and answer them honestly:

**Design compliance:**
- Does this implementation match what design.md specified?
- Did I follow the architecture decisions, or did I deviate? If I deviated, WHY?
- Does the code use the components, interfaces, and patterns described in the design?

**Spec satisfaction:**
- Does this task's output satisfy the corresponding spec requirements?
- Have ALL scenarios (Given/When/Then) from the spec been covered by tests?
- Would someone reading the spec and then the code agree they match?

**Test coverage check (against test-design.md):**
- Did I write all the test cases listed in test-design.md for this task?
- Are the edge cases from test-design.md actually tested, not just the happy path?
- Do the tests actually assert the right things, or are they weak/trivial?

**Honest assessment:**
- Am I truly done, or am I cutting corners to check the box?
- Is there anything I'm uncertain about that I should flag?
- If a fresh reviewer looked at this, would they find gaps?

**If reflection reveals gaps:** Fix them BEFORE marking the task complete. If the gap requires spec/design changes, pause and escalate.

**Reflection output format:**
```
TASK 1.1 REFLECTION
  Design compliance:  ✓ Matches design.md decisions
  Spec satisfaction:   ✓ All scenarios covered
  Test coverage:       ✓ All test-design.md cases written
  Confidence:          8/10
  Gaps found:          None
  → MARKING COMPLETE
```

Or if gaps are found:
```
TASK 1.2 REFLECTION
  Design compliance:  ⚠ Deviated from design — used Strategy pattern instead of Factory
  Spec satisfaction:   ✓ All scenarios covered
  Test coverage:       ✗ Missing edge case: empty input from test-design.md REQ-002
  Confidence:          5/10
  Gaps found:          2 — fixing before marking complete
  → FIXING GAPS
```

### 7. Update Progress

After verification AND reflection pass, update tasks.md:
```markdown
- [x] 1.1 Task description (completed)
```

Then move to the next task.

## Parallel Change Support

Multiple changes can be in-flight simultaneously. Each change lives in its own folder and is independent:

```
magic-spec/changes/
├── add-dark-mode/        (8/12 tasks complete)
├── fix-auth-flow/        (3/5 tasks complete)
└── refactor-database/    (0/7 tasks complete)
```

Specify which change to apply: `/magic-apply add-dark-mode`

## Interruption and Recovery

Implementation can be interrupted at any time. Progress is tracked via checkboxes in tasks.md.

To resume:
```
/magic-apply --continue [change-name]
```

This reads tasks.md, finds the first incomplete task, and continues from there.

## Handling Implementation Issues

### Spec Mismatch
If the spec doesn't match reality during implementation:
1. Stop the current task
2. Document the mismatch
3. Ask the user: "The spec says X but the code needs Y. Should I update the spec or change the approach?"
4. Update the proposal artifacts FIRST, then resume

### Blocked Task
If a task can't be completed due to external dependencies:
1. Mark it as blocked: `- [ ] 1.3 Task description ⚠️ BLOCKED: [reason]`
2. Skip to the next unblocked task
3. Return to blocked tasks when the blocker is resolved

### Scope Creep
If you discover additional work needed that's not in the tasks:
1. Do NOT do the extra work
2. Note it: "Discovered during task 1.3: we also need X"
3. Ask the user if they want to add it to the current change or create a separate proposal

## Archiving Completed Changes

When all tasks are complete, perform the **Final Self-Reflection** before archiving.

### Final Self-Reflection (All-Tasks Introspection)

When ALL tasks are marked complete, perform a comprehensive self-reflection across the entire change. This is the last gate before declaring the work done.

**Task completion audit:**
- Re-read tasks.md end to end. Is every checkbox genuinely earned?
- Are there tasks that were marked complete but feel uncertain?
- Did any task get "soft-completed" (marked done without full verification)?

**Design fulfillment review:**
- Re-read design.md. Does the final implementation match the architecture described?
- Were all Decisions followed? If any were changed during implementation, is the deviation documented?
- Are the Key Files table and the actual changed files consistent?

**Spec coverage verification:**
- Re-read each spec file in specs/. For every ADDED requirement, does matching code exist?
- For every MODIFIED requirement, does the new behavior work as specified?
- For every REMOVED requirement, is the old code actually gone?

**Test-design compliance:**
- Re-read test-design.md. Is every test case in the mapping table actually implemented?
- Run the full verification commands from test-design.md. All pass?
- Are there spec scenarios with NO corresponding test? If so, that's a gap.

**Regression check:**
- Run the full test suite (not just tests for this change). Any failures?
- Did this change break anything outside its scope?

**Honest final assessment:**
```
FINAL REFLECTION: <change-name>
═══════════════════════════════
Tasks:           <complete>/<total> (all verified: yes/no)
Design match:    ✓/⚠/✗ [notes]
Spec coverage:   ✓/⚠/✗ [notes]  
Test coverage:   ✓/⚠/✗ [notes]
Regressions:     ✓ none / ⚠ [list]
Overall:         DONE / DONE_WITH_CONCERNS
Concerns:        [list any, or "none"]
```

**If the final reflection reveals issues:** Fix them. Do not archive with known gaps. If the issues require spec/design changes, go back to `/magic-proposal --change <name>` first.

### Archive Process

After the final self-reflection passes:

1. Run `/magic-code-review` for independent verification
2. Move the change folder to archive:
   ```
   magic-spec/changes/<name>/
   → magic-spec/changes/archive/YYYY-MM-DD-<name>/
   ```
3. Sync delta specs to main specs (if maintaining a spec library):
   - Read each delta spec's ADDED/MODIFIED/REMOVED sections
   - Apply changes to the corresponding main spec files
   - This keeps the project's spec library up to date

## TDD Anti-Patterns to Avoid

- **Testing implementation**: Asserting on internal state instead of behavior
- **Mock everything**: Mocking the thing under test, not just dependencies
- **Happy path only**: Missing edge cases from spec scenarios
- **Tests that can't fail**: Assertions so weak they always pass
- **Test-after**: Writing tests after implementation (loses the RED-GREEN signal)
- **Flaky tests**: Tests that pass/fail randomly due to timing or state

## Key Principles

- **One task at a time** — Complete, verify, reflect, and mark done before moving on
- **Tests first** — RED-GREEN-REFACTOR for every behavior change, guided by test-design.md
- **Evidence over claims** — Run verification, don't assume
- **Reflect after every task** — Self-reflection catches gaps that tests miss
- **Design is the contract** — If the code doesn't match the design, it's not done
- **Stay on plan** — Follow the tasks. Scope creep is the enemy of completion.
- **Honest progress** — Checkboxes mean "verified, reflected, and complete"
- **Final introspection** — The all-tasks reflection is the last gate before archiving
- **Interrupt safely** — Progress is always saved in tasks.md

## Completion Status

Report apply status as one of:
- **DONE** — All tasks completed and verified. Evidence provided.
- **DONE_WITH_CONCERNS** — Completed, but with issues to note. List each concern.
- **IN_PROGRESS** — Some tasks complete, more remaining. Report progress.
- **BLOCKED** — Cannot proceed. State blocker and what was tried.
- **NEEDS_CONTEXT** — Missing information to continue. State exactly what's needed.
