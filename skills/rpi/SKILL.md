---
name: rpi
description: "Guided feature development workflow with three phases: Requirement gathering, Planning, and Implementation. Use this skill whenever the user invokes /rpi, wants structured or phased feature development, mentions requirement gathering before coding, wants to avoid jumping straight into implementation, or mentions 'rpi workflow'. Triggers on /rpi {feature-name}."
---

# RPI — Requirement → Plan → Implementation

A structured workflow that guides feature development through three phases:

1. **R (Requirement)**: Interview the user to nail down exactly what to build
2. **P (Plan)**: Design the implementation with reviews and plan mode
3. **I (Implementation)**: Build it step by step with review checkpoints

## Invocation

The user provides a feature name: `/rpi {feature-name}`

The `{feature-name}` must be a short kebab-case label (e.g., `auth-login`, `csv-export`). It's used as the directory name for all artifacts. If the argument is missing or invalid, ask the user to provide one.

## File Layout

All artifacts live under `.rpi/{feature-name}/`:

```
.rpi/{feature-name}/
├── requirement.md    — Requirements document (Phase 1 output)
├── plan.md           — Implementation plan (Phase 2 output)
└── implementation.md — Deviation report (Phase 3, only if deviations occurred)
```

## On Invocation

Create `.rpi/{feature-name}/` if it doesn't exist, then always start from Phase 1. Each invocation runs the full R → P → I workflow from the beginning — there is no cross-session resume. If the directory already exists with files from a previous run, they will be overwritten as the workflow progresses.

Start by reading `references/phase-r.md` and following its instructions. When a phase completes, read the next phase's reference file:

- Phase 1 → `references/phase-r.md`
- Phase 2 → `references/phase-p.md`
- Phase 3 → `references/phase-i.md`

In the reference files, `{feature-name}` is a placeholder — substitute it with the actual feature name from the invocation.

## Phase Transitions

### Requirement → Plan (R → P)

Before advancing: ensure `requirement.md` is written and contains all decisions from the interview.

### Plan → Implementation (P → I)

Before advancing:

1. Ensure `plan.md` is written and reflects all review feedback
2. If any requirements were changed or added during planning, update `.rpi/{feature-name}/requirement.md`:
   - Apply the changes to the relevant sections
   - Add a `## Requirement Changes (Phase 2)` section at the end, listing each change with the reason it was necessary (e.g., "Added X because codebase exploration revealed Y")

## Jump-Back Mechanism

Within the same conversation, the user can request to jump back to any earlier phase at any time (e.g., "let's go back to requirements"). When this happens:

1. Keep all existing files intact — they serve as the starting point for the re-entered phase
2. If jumping back from Phase 3 (Implementation), warn the user that there may be uncommitted code changes in the working tree. Suggest they stash or revert before re-entering an earlier phase, so incomplete code doesn't interfere with revised requirements or plans.
3. Re-read the relevant artifacts and re-enter that phase's reference file

## Interview Approach

When the skill calls for interviewing the user (in Phase 1 and Phase 2), follow this approach:

- Ask questions **one at a time**
- For each question, provide your **recommended answer** so the user can simply agree or adjust
- If a question can be answered by **exploring the codebase**, explore it yourself instead of asking
- Walk through each branch of the decision tree, resolving dependencies between decisions sequentially
- Either you or the user can decide when the interview is complete — suggest ending when all dimensions have been covered
