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

1. **Extract** — Identify the repeatable process from conversation history
2. **Load** — Bring skill-writing knowledge into context
3. **Interview** — Refine the extracted process with the user
4. **Write** — Produce the skill files
5. **Review** — Self-review + user review
6. **Verify** — Confirm files are correctly written

---

## Phase 1: Extract

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

## Phase 2: Load Skill-Writing Knowledge

Good skills follow specific patterns — frontmatter format, description writing, progressive disclosure, etc. Before the interview, load this knowledge so it informs both the questions you ask and the skill you write.

Check if `/skill-creator` is in your available skills list.

- **If available:** Invoke `/skill-creator`. Only load the skill-writing knowledge into context — do NOT start the eval/iterate/test-case workflow. Tell skill-creator you're here to write a skill based on an already-extracted process.
- **If not available:** Read `references/writing-guide.md` (bundled with this skill) for the essential guidelines.

---

## Phase 3: Interview

Confirm and refine the extracted process with the user. Use this approach:

- Ask questions **one at a time**
- For each question, provide your **recommended answer** so the user can agree or adjust
- If a question can be answered by **exploring the codebase**, do that instead of asking
- Walk through each branch of the decision tree, resolving dependencies sequentially

### What to Cover

Adapt depth to the complexity of the extracted process.

**Always cover** (even for simple processes):

| Dimension | What to resolve |
|---|---|
| Generalization vs. specialization | Which parts of the session are universal, which were instance-specific? |
| Trigger conditions | When should this skill activate? What would a user say or do? |
| Step adjustments | Should any steps be reordered, merged, split, added, or removed? |
| Artifacts | What does the skill produce when it's done? |
| Output path | Where should the skill files be written? Default: `{project-root}/.claude/skills/` |

**Also cover for complex processes** (multiple phases, branching, significant artifacts):

| Dimension | What to resolve |
|---|---|
| Parameters | What inputs does the skill need? |
| Completion criteria | When is each step done? When is the whole skill done? |

Batching low-controversy items into a single confirmation is fine — e.g., "I'd suggest these defaults for output path and artifacts: [X, Y]. OK?"

### Ending the Interview

Either you or the user can end it. Suggest ending when all relevant dimensions are covered. The user can also stop at any time.

---

## Phase 4: Write

Produce the skill files based on everything gathered so far.

### Output Path

Default: `{project-root}/.claude/skills/{skill-name}/`. Use whatever the user confirmed in the interview.

### Structure

Decide based on the process complexity:

- **Simple process** (single linear flow): Just `{skill-name}/SKILL.md`
- **Complex process** (multiple phases, branching logic): `{skill-name}/SKILL.md` as orchestrator + `{skill-name}/references/*.md` for each phase or module

Follow the same structural conventions as the `rpi` skill — SKILL.md contains frontmatter, overall flow, and shared behavior; references/ contain detailed instructions for each phase.

### Writing the Skill

Apply the skill-writing knowledge loaded in Phase 2. Key principles:

- **Description field** is the primary trigger mechanism. Make it specific and slightly "pushy" — include the contexts and phrasings that should activate the skill.
- **Explain the why**, not just the what. Future Claude sessions are smart — give them reasoning so they can handle edge cases, not just rote instructions.
- **Keep it lean.** Don't include instructions that aren't pulling their weight. If something can be left to Claude's judgment, leave it.
- **Use imperative form** for instructions.
- **Include examples** where the expected behavior might be ambiguous.

---

## Phase 5: Review

### Self-Review

Read through the produced skill and check:

- **Completeness** — All decisions from the interview are reflected. No steps missing.
- **Clarity** — A future Claude session with no context about this conversation can follow the instructions correctly.
- **Generalization** — No residual session-specific details (hardcoded paths, project names, etc.) that should have been parameterized.
- **Frontmatter** — Name and description are correct. Description is specific enough to trigger reliably.
- **Structure** — Progressive disclosure is appropriate for the complexity. References are used (or not) sensibly.

Fix any issues found. If something requires user input, go back to Phase 3 for that specific question.

### User Review

Present the skill to the user for confirmation.

- If the user has feedback or changes: go back to Phase 3 (for scope/design questions) or Phase 4 (for writing adjustments) as appropriate.
- If the user approves: proceed to Phase 6.

---

## Phase 6: Verify

Confirm the skill files are correctly written to the specified path:

- `{output-path}/{skill-name}/SKILL.md` exists and is complete
- If references/ were created, all referenced files exist and are complete
- Any cross-references between SKILL.md and reference files are valid

Fix any issues found, then report completion to the user.
