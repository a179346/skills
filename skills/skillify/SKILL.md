---
name: skillify
description: "Extract a repeatable process from the current conversation and produce a complete Claude Code skill. Use this whenever the user wants to capture a workflow they just did as a reusable skill, says things like 'turn this into a skill', 'skillify this', 'make a skill from what we just did', or invokes /skillify {skill-name}. Also trigger when the user finishes a multi-step workflow and mentions wanting to reuse it later."
---

# Skillify

Turn a conversation into a reusable Claude Code skill. The user has just gone through a workflow worth repeating — your job is to distill it into a well-structured skill that a future Claude session can follow.

## Invocation

`/skillify {skill-name}`

`{skill-name}` is used as the skill's directory name. It should be a short kebab-case label (e.g., `debug-memory-leak`, `review-api-endpoint`).

- If the user provides a name that isn't kebab-case, auto-convert it (e.g., `"my cool skill"` → `my-cool-skill`) and confirm with the user.
- If no name is provided, ask for one.

## Language

Write all produced skill files in English — Claude follows English instructions most reliably, and skills are instructions for Claude, not documentation for humans.

## Phases

Run these phases in order:

1. **Sanity Check** — Is this worth making into a skill?
2. **Extract** — Identify the repeatable process from conversation history
3. **Load** — Bring skill-writing knowledge into context
4. **Interview** — Refine the extracted process with the user
5. **Write** — Produce the skill files
6. **Review** — Self-review + user review
7. **Verify** — Confirm files are correctly written

---

## Phase 1: Sanity Check

Before diving in, do a quick reusability check. A good skill candidate is a workflow that:

- **Will be repeated** — across projects, repos, or over time. One-off tasks (fixing a specific bug, a one-time data migration) don't need a skill.
- **Has generalizable steps** — the core process works beyond this specific instance. If every step depends on this exact codebase/dataset/context, it's not reusable.
- **Benefits from consistency** — following the same steps each time matters (e.g., a deploy checklist, a review process).

If the workflow looks like a one-off, mention it briefly: "This looks fairly specific to this situation — are you sure you want a reusable skill for it, or was this more of a one-time thing?" If the user still wants it, proceed — they may see reuse potential you don't.

---

## Phase 2: Extract

Review the current conversation history and identify the **primary successful workflow** — the sequence of steps that actually achieved the user's goal.

Filter out:
- Failed attempts and backtracking
- Debugging/troubleshooting specific to this instance
- One-off exploratory commands

If the conversation contains multiple distinct workflows, ask the user which one to skillify.

### Output

Present a summary to the user (don't write any files yet):

- **Identified process** — What was done, which steps are reusable
- **Suggested trigger** — When should this skill activate
- **Step breakdown** — Rough steps and what each one does

This summary is the starting point for the interview.

### Existing Skill

If a skill with the same name already exists at the output path, read it and show the user what's there. Ask: revise the existing skill based on this conversation, or start fresh?

### Not Enough Context

If the conversation history has been heavily compacted or there's no clear process to extract, say so honestly. Then ask the user to describe the process they want to skillify. Don't refuse — the user likely has a clear workflow in mind even if it's not in the conversation.

---

## Phase 3: Load Skill-Writing Knowledge

Good skills follow specific patterns — frontmatter format, description writing, progressive disclosure, etc. Before the interview, load this knowledge so it informs both the questions you ask and the skill you write.

Read `references/writing-guide.md` (bundled with this skill). It covers frontmatter, descriptions, progressive disclosure, writing style, and common pitfalls — everything you need to write a well-structured skill.

---

## Phase 4: Interview

Confirm and refine the extracted process with the user. The goal is to fill gaps and resolve ambiguity — not to interrogate. Scale your questions to the complexity of the workflow.

### Approach

- Provide your **recommended answer** with each question so the user can just confirm or adjust
- If a question can be answered by **exploring the codebase**, do that instead of asking
- Batch low-controversy items into a single confirmation (e.g., "I'd suggest these defaults for output path and trigger: [X, Y]. OK?")

### Simple workflows (linear, 3-5 steps)

Ask 1-2 focused questions. You likely already know most of what you need from the Extract phase. Key things to confirm:

- **Trigger** — When should this skill activate? What would a user say?
- **Generalization** — Anything from the session that should be parameterized instead of hardcoded?

For the rest (output path, artifacts, step ordering), propose sensible defaults and move on unless the user objects.

### Complex workflows (multiple phases, branching, significant artifacts)

Ask more questions, but still one at a time. Beyond trigger and generalization, also cover:

- **Parameters** — What inputs does the skill need?
- **Step adjustments** — Should any steps be reordered, merged, split, added, or removed?
- **Completion criteria** — When is each step done? When is the whole skill done?

### Ending the Interview

Either you or the user can end it. Suggest ending when you have enough to write a solid draft — the Review phase will catch remaining issues. The user can also stop at any time.

---

## Phase 5: Write

Produce the skill files based on everything gathered so far. Follow the guidelines in `references/writing-guide.md` — it covers frontmatter, descriptions, structure decisions, writing style, and common pitfalls.

### Output Path

Default: `{project-root}/.claude/skills/{skill-name}/`. Use whatever the user confirmed in the interview.

### Structure

Decide based on the process complexity:

- **Simple process** (single linear flow): Just `{skill-name}/SKILL.md`
- **Complex process** (multiple phases, branching logic): `{skill-name}/SKILL.md` as orchestrator + `{skill-name}/references/*.md` for each phase or module

SKILL.md contains frontmatter, overall flow, and shared behavior; references/ contain detailed instructions for each phase.

---

## Phase 6: Review

### Self-Review

Read through the produced skill and check:

- **Completeness** — All decisions from the interview are reflected. No steps missing.
- **Clarity** — A future Claude session with no context about this conversation can follow the instructions correctly.
- **Generalization** — No residual session-specific details (hardcoded paths, project names, etc.) that should have been parameterized.
- **Frontmatter** — Name and description are correct. Description is specific enough to trigger reliably.
- **Structure** — Progressive disclosure is appropriate for the complexity. References are used (or not) sensibly.

Fix any issues found. If something requires user input, go back to Phase 4 for that specific question.

### User Review

Present the skill to the user for confirmation.

- If the user has feedback or changes: go back to Phase 4 (for scope/design questions) or Phase 5 (for writing adjustments) as appropriate.
- If the user approves: proceed to Phase 7.

---

## Phase 7: Verify

Confirm the skill files are correctly written to the specified path:

- `{output-path}/{skill-name}/SKILL.md` exists and is complete
- If references/ were created, all referenced files exist and are complete
- Any cross-references between SKILL.md and reference files are valid

Fix any issues found, then report completion to the user.
