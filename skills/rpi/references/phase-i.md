# Phase 3: Implementation

Goal: Build what was planned. The code changes are the final deliverable.

## Step 1 — Implement

Read `.rpi/{feature-name}/plan.md` to load the implementation plan.

Execute the plan step by step. For each step:

1. Make the code changes described in the plan
2. At meaningful checkpoints, run related tests (if they exist). A "meaningful checkpoint" is when a coherent unit of work is complete — a function, a module, a wiring step. Don't run tests after every single line change, but don't wait until everything is done either.
3. If tests fail, fix the issue before moving on

**Do not auto-commit.** Leave committing to the user.

## Step 2 — Reviews

Two sequential review passes. If any review surfaces issues, discuss with the user and return to **Step 1** to fix the code.

### 2-1: Skill-Based Review

Look at the available skills and the codebase's tech stack. Identify skills that could review the implementation — code review, testing, security, type checking, etc.

**Present the shortlist to the user with reasoning.** Let the user choose which to run. Then invoke each selected skill directly (via the Skill tool) to review the implementation.

If reviews surface actionable issues: discuss with the user, return to Step 1 if changes are needed.

### 2-2: Self-Review

Review the implementation yourself. Ask: does this implementation have any potential issues?

- Does it match the plan?
- Are there bugs, edge cases, or error handling gaps?
- Is the code clean and consistent with the codebase's style?

If you find issues: discuss with the user, return to Step 1 if needed.

## Step 3 — Done

The implementation is complete. Summarize what was built and any notable decisions made during implementation. Remind the user they can commit when ready.

The code changes are the final output — no additional documents are produced in this phase.
