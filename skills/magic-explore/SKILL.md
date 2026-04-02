---
name: magic-explore
description: "Use when investigating ideas, analyzing code, comparing approaches, or understanding a problem space before committing to a plan. Free-form exploration that crystallizes into actionable insights."
---

# /magic-explore — Deep Exploration & Discovery

Investigate, analyze, and understand before building. This skill combines free-form exploration with structured discovery to turn vague ideas into clear understanding.

**Philosophy:** Great software starts with understanding. Explore first, propose later. The cost of exploration is minutes; the cost of building the wrong thing is days.

## When to Use

- Investigating a new feature idea or problem space
- Analyzing existing code to understand how something works
- Comparing multiple technical approaches before committing
- Researching a bug's root cause before fixing
- Understanding a codebase before making changes
- Answering "is this worth building?" or "how should we approach this?"

## Hard Gate

<HARD-GATE>
Do NOT create any artifacts (proposals, specs, designs, code) during exploration. Exploration is about UNDERSTANDING, not producing. The only output is insight and conversation. When understanding crystallizes, transition to `/magic-proposal`.
</HARD-GATE>

## Exploration Modes

### Mode 1: Idea Exploration (Default)
Turn a vague idea into a clear understanding of what to build and why.

### Mode 2: Codebase Investigation
Understand how existing code works, find patterns, map architecture.

### Mode 3: Root Cause Analysis
Systematically investigate bugs or unexpected behavior using evidence-based reasoning.

### Mode 4: Approach Comparison
Compare 2-3 technical approaches with concrete trade-offs before committing.

## Checklist

Complete these steps in order:

1. **Detect mode** — Determine which exploration mode fits the user's request
2. **Gather context** — Check files, docs, recent commits, existing specs
3. **Ask clarifying questions** — One at a time, prefer multiple choice when possible
4. **Map the landscape** — Identify key components, dependencies, constraints
5. **Investigate deeply** — Follow evidence chains, read actual code, trace data flows
6. **Propose approaches** — Present 2-3 approaches with concrete trade-offs
7. **Synthesize findings** — Summarize key insights and recommendation
8. **Transition decision** — Either continue exploring or transition to `/magic-proposal`

## The Process

### Understanding the Problem

- Read the codebase first: files, docs, recent commits, config
- Before asking detailed questions, assess scope: if the request spans multiple independent systems, flag this immediately
- Ask questions ONE at a time — never overwhelm with multiple questions
- Prefer multiple choice questions when possible (easier to answer)
- Focus on: purpose, constraints, success criteria, existing assumptions
- Listen for what the user DOESN'T say — gaps in understanding are often more important than stated requirements

### Deep Investigation

**For code analysis:**
- Map the architecture: what components exist, how they connect, where boundaries are
- Trace data flows: where does input enter? Where does it exit? What transforms happen?
- Identify invariants and assumptions the code relies on
- Look for patterns and anti-patterns in the existing code

**For root cause analysis (Iron Law: no fix without root cause):**

Phase 1 — Evidence Gathering:
- Read error messages completely, note line numbers and file paths
- Reproduce consistently: exact steps, frequency, environment
- Check recent changes: git diff, new dependencies, config changes
- Trace data flow backward from the symptom to find the original bad value

Phase 2 — Pattern Analysis:
- Find working examples in the same codebase
- Read reference implementations completely (not skimming)
- Identify ALL differences, however small
- Map dependencies: components, config, environment, assumptions

Phase 3 — Hypothesis Testing:
- Form a single specific hypothesis: "I think X is the root cause because Y"
- Test minimally: one variable at a time
- If hypothesis fails → NEW hypothesis (don't pile on more fixes)
- If 3+ hypotheses fail → Stop, question the architecture, escalate

### Exploring Approaches

- Always propose 2-3 different approaches with trade-offs
- Lead with your recommended option and explain WHY
- Use concrete details: name the files, the functions, the libraries
- Show real numbers: not "this might be slow" but "this queries N+1, ~200ms per page with 50 items"
- Consider: complexity, maintainability, performance, security, testability
- Apply YAGNI ruthlessly — remove unnecessary features from all approaches

### Working in Existing Codebases

- Explore current structure BEFORE proposing changes
- Follow existing patterns unless they're clearly broken
- Where existing code has problems that affect the work, include targeted improvements
- Don't propose unrelated refactoring — stay focused on the current goal

## Confidence Calibration

Rate your understanding at each stage:

| Score | Meaning |
|-------|---------|
| 9-10 | Deep understanding. Can explain the full system and predict edge cases. |
| 7-8 | Good understanding. Know the main paths, some edges unclear. |
| 5-6 | Partial understanding. Know the shape, details fuzzy. |
| 3-4 | Surface understanding. Know it exists, don't know how it works. |
| 1-2 | Almost nothing. Need to start from scratch. |

**Report your confidence honestly.** If you're at 5/10, say so. The user needs to know what you don't know.

## Transition to Proposal

When exploration reaches clarity (typically 7+/10 confidence), offer the transition:

> "I have a clear understanding of [topic]. Ready to crystallize this into a formal proposal with specs and tasks? I'll use `/magic-proposal` to create the structured artifacts."

Wait for user confirmation before transitioning. If the user wants to keep exploring, keep exploring.

## Key Principles

- **One question at a time** — Don't overwhelm with multiple questions
- **Evidence over speculation** — Read the actual code, don't guess
- **Concrete over abstract** — Name files, functions, line numbers
- **Explore alternatives** — Always consider 2-3 approaches
- **Know when to stop** — Exploration has diminishing returns; recognize when to crystallize
- **Honest uncertainty** — Report what you don't know as clearly as what you do know

## Anti-Patterns

- **Premature solution**: Jumping to implementation before understanding the problem
- **Infinite exploration**: Never reaching a conclusion or recommendation
- **Confirmation bias**: Only looking for evidence that supports your initial assumption
- **Surface scanning**: Skimming code instead of reading it deeply
- **Scope creep**: Letting exploration expand to unrelated areas

## Completion Status

Report exploration status as one of:
- **INSIGHT_READY** — Understanding is clear, ready to transition to `/magic-proposal`
- **EXPLORING** — Still investigating, need more information
- **BLOCKED** — Cannot proceed without external input or access
- **NEEDS_CONTEXT** — Missing critical information to continue
