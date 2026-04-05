# Phase 3: Implementation

Goal: Build what was planned. The code changes are the final deliverable.

## Requirement & Plan Stability

Both the requirements and plan were finalized in earlier phases. This phase focuses on executing the plan, not revisiting decisions. Most issues that surface during implementation can be resolved within the plan's framework.

However, deviations can happen. Handle them differently depending on what changed:

**Requirement changes** — these always need user confirmation. Stop, explain what you discovered and why the requirement can't hold, and get explicit approval before proceeding.

**Plan changes** — use judgment based on impact:
- *Minor adjustments* (e.g., reordering steps, using a slightly different utility, adjusting a function signature to fit the actual code): proceed without asking, but still track the change
- *Significant deviations* (e.g., different architecture approach, skipping a planned step, adding a major component): stop and confirm with the user

Track all deviations regardless of size — you'll document them in `implementation.md` at the end (see Step 3).

## Step 1 — Implement

Read `.rpi/{feature-name}/requirement.md` to understand what the user wants and why, then read `.rpi/{feature-name}/plan.md` to load the implementation plan.

Before starting each step, read the actual source files you're about to modify. The plan describes *what* to change, but the code has details (local variables, edge cases in existing functions, recent changes) that you need fresh context on.

Execute the plan step by step. For each step:

1. Make the code changes described in the plan
2. At meaningful checkpoints, run related tests (if they exist). A "meaningful checkpoint" is when a coherent unit of work is complete — a function, a module, a wiring step. Don't run tests after every single line change, but don't wait until everything is done either.
3. If tests fail, fix the issue before moving on

**Do not auto-commit.** Leave committing to the user.

## Step 2 — Reviews

Sequential review passes. If any review surfaces issues, discuss with the user and return to **Step 1** to fix the code.

### 2-1: Skill-Based Review

Look at the available skills and the codebase's tech stack. Identify skills that could review the implementation — code review, testing, security, type checking, etc.

**Present the shortlist to the user with reasoning.** Let the user choose which to run. Then spawn one agent per selected skill, running all agents in parallel. Each agent only needs the code changes as context — don't pass requirement.md or plan.md.

Wait for all agents to complete, then summarize all review findings together. If an agent fails, note which skill's review didn't complete — don't block the flow. If different agents give conflicting recommendations, present both sides and let the user decide.

If reviews surface actionable issues: discuss with the user, return to Step 1 if changes are needed.

### 2-2: Requirement Traceability

Open `.rpi/{feature-name}/requirement.md` and go through each requirement one by one. For each, verify that the implementation actually covers it. The plan may have inadvertently dropped or under-specified a requirement, so check against requirements — not just the plan.

- If a requirement is fully covered: move on
- If a requirement is partially covered or missing: discuss with the user whether it was intentionally deferred or accidentally missed, then return to Step 1 if needed

### 2-3: Self-Review (pre-simplify)

Review the implementation yourself before simplification. Focus on correctness:

- Are there bugs, edge cases, or error handling gaps?
- Is the code clean and consistent with the codebase's style?

Fix any issues before proceeding — it's easier to verify correctness against code you just wrote than code that's been reshaped by simplification.

If you find issues: discuss with the user, return to Step 1 if needed.

### 2-4: Simplify

Run `/simplify` on the files modified during this implementation. Scope it explicitly to only the changed files — don't let it touch unrelated code. This runs automatically — no need to ask the user.

If `/simplify` is not available in this session, do a manual simplification pass yourself: look for duplicated logic, overly complex conditionals, or unnecessary abstractions in the changed files, and clean them up.

### 2-5: Self-Review (post-simplify)

Review the code again after simplification. Simplification can occasionally alter behavior in subtle ways. Check that:

- The logic is still correct after the refactoring
- The code is consistent with the codebase's style

If you find issues: discuss with the user, return to Step 1 if needed.

### 2-6: Full Test Suite

If the project has a test suite, run it in full. Step 1 only ran related tests at checkpoints — this catches integration regressions across the broader codebase. If the suite is too large to run entirely, run at minimum the test files related to modules touched by this implementation.

If tests fail: diagnose whether the failure is caused by this implementation or is pre-existing. Fix implementation-caused failures before proceeding; flag pre-existing failures to the user.

### 2-7: Format & Lint

Check if the project has a formatter or linter (look for config files like `.eslintrc`, `.prettierrc`, `biome.json`, etc., and scripts in `package.json` / `pyproject.toml`). If found, run them on the changed files. If none exist, skip this step.

When handling linter errors:
- Won't affect logic: fix directly
- Might affect logic: confirm with the user first

## Step 3 — Done

The implementation is complete. Summarize what was built and any notable decisions made during implementation. Remind the user they can commit when ready.

### Deviation Report

If any requirements or plan steps were changed during implementation, write `.rpi/{feature-name}/implementation.md` documenting the deviations:

```markdown
# Implementation Deviations

## Requirement Changes
- **[requirement description]**: [what changed] — **Reason:** [why it was necessary]

## Plan Changes
- **[plan step description]**: [what changed] — **Reason:** [why it was necessary]
```

Only produce this file if deviations occurred. If the implementation followed the requirements and plan exactly, skip it — the code changes speak for themselves.
