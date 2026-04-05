# Phase 2: Plan

Goal: Design how to implement the requirements. Produce a concrete, reviewable implementation plan.

This entire phase runs inside Claude Code plan mode. All steps — interviewing, drafting, reviewing — are read-only operations that don't need to exit plan mode. The plan is written to plan mode's own plan file. Only after the user approves and you exit plan mode do you persist the plan to `.rpi/{feature-name}/plan.md`.

## Step 1 — Enter Plan Mode

Enter Claude Code plan mode using `EnterPlanMode`. This restricts you to read-only actions, which is intentional — you should be exploring the codebase and thinking, not editing yet. If `EnterPlanMode` is not available in this session, proceed without it — just stay disciplined about not making code changes during this phase.

Read `.rpi/{feature-name}/requirement.md` to load the requirements into context.

## Step 2 — Explore the Codebase

Before interviewing the user, explore the codebase to understand existing conventions and architecture that will shape the plan. Focus on:

- **Conventions**: naming, file structure, patterns, error handling, testing style
- **Architecture**: how similar features are structured, relevant modules the new feature will interact with
- **Dependencies**: existing utilities, libraries, or abstractions that the implementation could reuse

This upfront exploration makes the interview more targeted (you already know the constraints) and makes the plan naturally consistent with the codebase from the start.

## Requirement Stability

The requirements were finalized in Phase 1. This phase focuses on *how* to implement them, not *what* to build. Avoid changing requirements during planning — most "requirement changes" that surface here are actually implementation details that belong in the plan, not the requirements doc.

However, genuine requirement gaps can emerge when you dig into the codebase and discover something that wasn't considered. When this happens:

1. Flag it explicitly to the user: explain what you discovered and why it affects the requirements
2. Get the user's confirmation before treating it as a requirement change
3. Track the change — you'll need to update `requirement.md` during the phase transition (see SKILL.md Phase 2.5)

## Step 3 — Interview the User

Interview the user about implementation details. Follow the interview approach from SKILL.md:

- One question at a time, with recommended answers
- Explore the codebase instead of asking when possible
- Focus on: architecture choices, technology decisions, integration points, migration concerns, testing strategy

The interview ends when implementation decisions are clear enough to write a plan.

## Step 4 — Write the Plan

Write the plan to plan mode's plan file. The plan should follow whatever format Claude Code's plan mode naturally produces — typically:

- Context section explaining the change
- Step-by-step implementation approach
- Files to create/modify with descriptions of changes
- Verification/testing approach

## Step 5 — Reviews

Three sequential review passes. All reviews happen inside plan mode (they're read-only). If any review surfaces issues, discuss with the user and return to **Step 4** to update the plan.

### 5-1: Requirement Traceability

Open `.rpi/{feature-name}/requirement.md` and go through each requirement one by one. Verify that the plan has a concrete step covering it. Requirements can be easy to lose during planning — especially edge cases, out-of-scope boundaries, and non-functional requirements.

- If every requirement maps to a plan step: proceed
- If a requirement is missing or only vaguely covered: discuss with the user whether it was intentionally deferred or overlooked, then return to Step 4 if needed

### 5-2: Convention Review

Compare the plan against the conventions discovered in Step 2. If the codebase has changed or you need to verify specific details, explore again — but the heavy lifting should already be done.

- **If consistent**: proceed to 5-3
- **If the plan deviates and the plan's approach is better**: explain the deviation to the user and ask if they agree. If yes, proceed. If no, return to Step 4.
- **If the plan deviates and the convention is better**: return to Step 4 and align the plan with the convention.

### 5-3: Skill-Based Review

Look at the available skills in this session and the codebase's tech stack (languages, frameworks, tools). Identify skills that could provide useful review perspectives — for example, code review skills, architecture skills, or framework-specific skills.

**Present the shortlist to the user with reasoning** for why each skill is relevant. Let the user choose which ones to use. Then invoke each selected skill directly (via the Skill tool) to review the plan. This works within plan mode since review skills only need read-only access.

If reviews surface actionable issues: discuss with the user, then return to Step 4 if changes are needed.

### 5-4: Self-Review

Review the plan yourself. Ask: does this plan have any potential issues? Consider:

- Missing edge cases
- Performance concerns
- Security implications
- Dependencies or ordering issues
- Gaps between requirements and plan

If you find issues: discuss with the user, return to Step 4 if needed.

## Step 6 — User Review

Present the plan to the user for review. Stay in plan mode during this step.

- If the user has feedback: return to **Step 3** (for new questions) or **Step 4** (for plan changes) — all still within plan mode
- If the user approves: exit plan mode using `ExitPlanMode` (if you entered it), then proceed to transition

### Transitioning to Phase 3

1. Write the approved plan content to `.rpi/{feature-name}/plan.md` as the handoff artifact for Phase 3
2. Read `references/phase-i.md` and begin Phase 3
