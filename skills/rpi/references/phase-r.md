# Phase 1: Requirement

Goal: Nail down what to build before thinking about how. No implementation details yet.

## Step 1 — Ask What We're Building

Ask the user: what do you want to build? Let them describe it in their own words. If they've already described it in the conversation (e.g., as the `/rpi` argument context), acknowledge that and move to the interview.

Update state: `{ "sub_step": "interview" }`

## Step 2 — Interview the User

Interview the user to fully understand the requirements. Follow the interview approach described in SKILL.md:

- One question at a time
- Provide your recommended answer for each
- Explore the codebase yourself when that can answer a question
- Cover all dimensions: scope, edge cases, constraints, user expectations, success criteria

The interview ends when both you and the user agree all aspects have been covered. You should proactively suggest ending when you believe the requirements are complete — don't drag it out.

## Step 3 — Write the Requirement Document

Write (or update) `.rpi/{feature-name}/requirement.md`.

Update state: `{ "sub_step": "write_requirement" }`

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

Don't show this document to the user yet — that's Step 5.

## Step 4 — Self-Review

Update state: `{ "sub_step": "self_review" }`

Re-read `requirement.md` and ask yourself: are there any requirements that are ambiguous, incomplete, or potentially contradictory? Are there decisions that were assumed but never explicitly confirmed with the user?

- If you find gaps: return to **Step 2** to ask the user about the specific gaps
- If everything is solid: proceed to Step 5

## Step 5 — User Review

Update state: `{ "sub_step": "user_review" }`

Present the full content of `requirement.md` to the user in the terminal. Ask them to review it and confirm it's correct.

- If the user has feedback or corrections: return to **Step 2** (for new questions) or **Step 3** (for writing fixes), depending on the nature of the feedback
- If the user approves: transition to Phase 2

### Transitioning to Phase 2

1. Ensure `requirement.md` is saved with all final changes
2. Update state: `{ "phase": "plan", "sub_step": "enter_plan_mode" }`
3. Read `references/phase-p.md` and begin Phase 2
