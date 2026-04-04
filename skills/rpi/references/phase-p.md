# Phase 2: Plan

Goal: Design how to implement the requirements. Produce a concrete, reviewable implementation plan.

## Step 1 — Enter Plan Mode

Read `.rpi/{feature-name}/requirement.md` to load the requirements into context.

Enter Claude Code plan mode using `EnterPlanMode`. This restricts you to read-only actions, which is intentional — you should be exploring the codebase and thinking, not editing yet.

Update state: `{ "sub_step": "enter_plan_mode" }`

## Step 2 — Interview the User

Interview the user about implementation details. Follow the interview approach from SKILL.md:

- One question at a time, with recommended answers
- Explore the codebase instead of asking when possible
- Focus on: architecture choices, technology decisions, integration points, migration concerns, testing strategy

Update state: `{ "sub_step": "interview" }`

The interview ends when implementation decisions are clear enough to write a plan.

## Step 3 — Write the Plan

Exit plan mode using `ExitPlanMode`.

Write (or update) `.rpi/{feature-name}/plan.md` with the implementation plan. The plan should follow whatever format Claude Code's plan mode naturally produces — typically:

- Context section explaining the change
- Step-by-step implementation approach
- Files to create/modify with descriptions of changes
- Verification/testing approach

Update state: `{ "sub_step": "write_plan" }`

## Step 4 — Reviews

Three sequential review passes. If any review surfaces issues, discuss with the user and return to **Step 3** to update the plan.

### 4-1: Convention Review

Update state: `{ "sub_step": "review_convention" }`

Explore the codebase to understand existing conventions (naming, file structure, patterns, error handling, etc.). Compare them against the plan.

- **If consistent**: proceed to 4-2
- **If the plan deviates and the plan's approach is better**: explain the deviation to the user and ask if they agree. If yes, proceed. If no, return to Step 3.
- **If the plan deviates and the convention is better**: return to Step 3 and align the plan with the convention.

### 4-2: Skill-Based Review

Update state: `{ "sub_step": "review_skills" }`

Look at the available skills in this session and the codebase's tech stack (languages, frameworks, tools). Identify skills that could provide useful review perspectives — for example, code review skills, architecture skills, or framework-specific skills.

**Present the shortlist to the user with reasoning** for why each skill is relevant. Let the user choose which ones to use. For each selected skill, spawn an Agent to review the plan.

If reviews surface actionable issues: discuss with the user, then return to Step 3 if changes are needed.

### 4-3: Self-Review

Update state: `{ "sub_step": "review_self" }`

Review the plan yourself. Ask: does this plan have any potential issues? Consider:

- Missing edge cases
- Performance concerns
- Security implications
- Dependencies or ordering issues
- Gaps between requirements and plan

If you find issues: discuss with the user, return to Step 3 if needed.

## Step 5 — User Review

Update state: `{ "sub_step": "user_review" }`

Present the plan to the user. Since the plan was written to `.rpi/{feature-name}/plan.md`, show its contents and ask the user to review.

- If the user has feedback: return to **Step 2** (for new questions) or **Step 3** (for plan changes)
- If the user approves: transition to Phase 3

### Transitioning to Phase 3

1. Ensure `plan.md` is saved with all final changes
2. Update state: `{ "phase": "implementation", "sub_step": "implement" }`
3. Read `references/phase-i.md` and begin Phase 3
