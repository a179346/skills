# Phase 1: Requirement

Goal: Nail down what to build before thinking about how. No implementation details yet.

## Step 1 — Ask What We're Building

Ask the user: what do you want to build? Let them describe it in their own words. If they've already described it in the conversation (e.g., as the `/rpi` argument context), acknowledge that and move to the next step.

## Step 2 — Explore the Codebase

Before interviewing, explore the codebase to build context that will make your questions sharper. Focus on:

- **Existing architecture**: how the area the feature touches is currently structured
- **Patterns in use**: naming, file layout, similar features that set precedent
- **Constraints**: dependencies, APIs, or conventions the new feature will need to respect

This exploration is brief and targeted — you're not mapping the entire codebase, just the neighborhood the feature will live in. The goal is to ask informed questions in the interview (e.g., "the current auth uses middleware pattern — should the new feature follow that?" instead of "how should we handle auth?").

## Step 3 — Interview the User

Interview the user to fully understand the requirements. Follow the interview approach described in SKILL.md:

- One question at a time
- Provide your recommended answer for each
- Explore the codebase yourself when that can answer a question — this includes reading existing code to understand current architecture, related modules, or constraints that inform requirements
- Cover all dimensions: scope, edge cases, constraints, user expectations, success criteria

The interview ends when both you and the user agree all aspects have been covered. You should proactively suggest ending when you believe the requirements are complete — don't drag it out.

## Step 4 — Write the Requirement Document

Write (or update) `.rpi/{feature-name}/requirement.md`.

There's no rigid format, but use these sections as a starting scaffold. Adapt or skip sections as the content demands — the goal is clarity, not compliance with a template:

```markdown
# {Feature Name}

## Overview
What we're building and why.

## Requirements
The concrete things this feature must do.

## Decisions & Reasoning
Every decision made during the interview, with the reasoning behind it.
This section is critical — it preserves the "why" for future phases.

## Out of Scope
What we explicitly decided NOT to include.

## Open Questions
Anything still unresolved (ideally empty by the end of this phase).
```

Don't show this document to the user yet — that's Step 6.

## Step 5 — Self-Review

Re-read `requirement.md` and ask yourself: are there any requirements that are ambiguous, incomplete, or potentially contradictory? Are there decisions that were assumed but never explicitly confirmed with the user?

- If you find gaps: return to **Step 3** to ask the user about the specific gaps
- If everything is solid: proceed to Step 6

## Step 6 — User Review

Present the full content of `requirement.md` to the user in the terminal. Ask them to review it and confirm it's correct.

- If the user has feedback or corrections: return to **Step 3** (for new questions) or **Step 4** (for writing fixes), depending on the nature of the feedback
- If the user approves: transition to Phase 2

### Transitioning to Phase 2

1. Ensure `requirement.md` is saved with all final changes
2. Read `references/phase-p.md` and begin Phase 2
