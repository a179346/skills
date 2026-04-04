# Phase 2: Plan

Goal: Design how to implement the requirements. Produce a concrete, reviewable implementation plan.

This entire phase runs inside Claude Code plan mode. All steps — interviewing, drafting, reviewing — are read-only operations that don't need to exit plan mode. The plan is written to plan mode's own plan file. Only after the user approves and you exit plan mode do you persist the plan to `.rpi/{feature-name}/plan.md`.

## Step 1 — Enter Plan Mode

Read `.rpi/{feature-name}/requirement.md` to load the requirements into context.

Enter Claude Code plan mode using `EnterPlanMode`. This restricts you to read-only actions, which is intentional — you should be exploring the codebase and thinking, not editing yet.

## Step 2 — Interview the User

Interview the user about implementation details. Follow the interview approach from SKILL.md:

- One question at a time, with recommended answers
- Explore the codebase instead of asking when possible
- Focus on: architecture choices, technology decisions, integration points, migration concerns, testing strategy

The interview ends when implementation decisions are clear enough to write a plan.

## Step 3 — Write the Plan

Write the plan to plan mode's plan file. The plan should follow whatever format Claude Code's plan mode naturally produces — typically:

- Context section explaining the change
- Step-by-step implementation approach
- Files to create/modify with descriptions of changes
- Verification/testing approach

## Step 4 — Reviews

Three sequential review passes. All reviews happen inside plan mode (they're read-only). If any review surfaces issues, discuss with the user and return to **Step 3** to update the plan.

### 4-1: Convention Review

Explore the codebase to understand existing conventions (naming, file structure, patterns, error handling, etc.). Compare them against the plan.

- **If consistent**: proceed to 4-2
- **If the plan deviates and the plan's approach is better**: explain the deviation to the user and ask if they agree. If yes, proceed. If no, return to Step 3.
- **If the plan deviates and the convention is better**: return to Step 3 and align the plan with the convention.

### 4-2: Skill-Based Review

Look at the available skills in this session and the codebase's tech stack (languages, frameworks, tools). Identify skills that could provide useful review perspectives — for example, code review skills, architecture skills, or framework-specific skills.

**Present the shortlist to the user with reasoning** for why each skill is relevant. Let the user choose which ones to use. Then invoke each selected skill directly (via the Skill tool) to review the plan. This works within plan mode since review skills only need read-only access.

If reviews surface actionable issues: discuss with the user, then return to Step 3 if changes are needed.

### 4-3: Self-Review

Review the plan yourself. Ask: does this plan have any potential issues? Consider:

- Missing edge cases
- Performance concerns
- Security implications
- Dependencies or ordering issues
- Gaps between requirements and plan

If you find issues: discuss with the user, return to Step 3 if needed.

## Step 5 — User Review

Present the plan to the user for review. Stay in plan mode during this step.

- If the user has feedback: return to **Step 2** (for new questions) or **Step 3** (for plan changes) — all still within plan mode
- If the user approves: exit plan mode using `ExitPlanMode`, then proceed to transition

### Transitioning to Phase 3

1. Write the approved plan content to `.rpi/{feature-name}/plan.md` as the handoff artifact for Phase 3
2. Read `references/phase-i.md` and begin Phase 3
