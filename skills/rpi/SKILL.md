---
name: rpi
description: "Guided feature development workflow with three phases: Requirement gathering, Planning, and Implementation. Use this skill whenever the user invokes /rpi, wants to develop a feature from scratch with structured requirement gathering and planning, or mentions 'rpi workflow'. Triggers on /rpi {feature-name}."
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
├── state.json        — Current phase and sub-step
├── requirement.md    — Requirements document (Phase 1 output)
└── plan.md           — Implementation plan (Phase 2 output)
```

## State Management

### State File Format

```json
{
  "phase": "requirement",
  "sub_step": "interview",
  "feature_name": "auth-login"
}
```

### On Invocation

1. Check if `.rpi/{feature-name}/` exists
2. **If new**: Create the directory, initialize `state.json` with phase `requirement` and sub_step `interview`, then start Phase 1
3. **If exists**: Read `state.json`, re-read all existing artifacts (`requirement.md`, `plan.md` if they exist) to rebuild context, then resume at the recorded phase and sub-step

### Updating State

Update `state.json` whenever you enter a new sub-step. This is your bookmark — if the conversation ends and resumes later, you'll pick up here.

### Valid Sub-Steps by Phase

- **requirement**: `interview`, `write_requirement`, `self_review`, `user_review`
- **plan**: `enter_plan_mode`, `interview`, `write_plan`, `review_convention`, `review_skills`, `review_self`, `user_review`
- **implementation**: `implement`, `review_skills`, `review_self`, `done`

## Phase Dispatching

Based on the current phase in `state.json`, read the corresponding reference file and follow its instructions:

- `requirement` → read `references/phase-r.md`
- `plan` → read `references/phase-p.md`
- `implementation` → read `references/phase-i.md`

## Phase Transitions

### Requirement → Plan (R → P)

Before advancing: ensure `requirement.md` is written and contains all decisions from the interview. Update `state.json` to `{ "phase": "plan", "sub_step": "enter_plan_mode" }`.

### Plan → Implementation (P → I)

Before advancing: ensure `plan.md` is written and reflects all review feedback. Update `state.json` to `{ "phase": "implementation", "sub_step": "implement" }`.

## Jump-Back Mechanism

The user can request to jump back to any earlier phase at any time (e.g., "let's go back to requirements"). When this happens:

1. Update `state.json` to the target phase's first sub-step
2. Keep all existing files intact — they serve as the starting point for the re-entered phase
3. Re-read the relevant artifacts and continue from there

## Language

Respond in whatever language the user is using. The user may write in English, Chinese, or any other language — match them.

## Interview Approach

When the skill calls for interviewing the user (in Phase 1 and Phase 2), follow this approach:

- Ask questions **one at a time**
- For each question, provide your **recommended answer** so the user can simply agree or adjust
- If a question can be answered by **exploring the codebase**, explore it yourself instead of asking
- Walk through each branch of the decision tree, resolving dependencies between decisions sequentially
- Either you or the user can decide when the interview is complete — suggest ending when all dimensions have been covered
