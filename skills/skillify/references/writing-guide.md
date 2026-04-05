# Skill Writing Guide

The single source of truth for writing well-structured Claude Code skills. Read this before writing or reviewing any skill.

## Anatomy of a Skill

```
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter (name, description required)
│   └── Markdown instructions
└── Bundled Resources (optional)
    ├── scripts/    - Executable code for deterministic/repetitive tasks
    ├── references/ - Docs loaded into context as needed
    └── assets/     - Files used in output (templates, icons, fonts)
```

## Frontmatter

Every SKILL.md starts with YAML frontmatter:

```yaml
---
name: my-skill
description: "What this skill does and when to use it."
---
```

- `name`: kebab-case identifier
- `description`: The primary trigger mechanism. This text is always in Claude's context (~100 words) and determines whether the skill gets invoked.

## Writing Effective Descriptions

The description field is how Claude decides whether to consult the skill. Get this wrong and the skill never fires; get it right and it activates reliably.

**Be specific.** Include both what the skill does AND the contexts/phrasings that should activate it.

**Be slightly pushy.** Claude tends to under-trigger skills. Explicitly list the situations where the skill should activate, even non-obvious ones.

**Bad:** `"A skill for working with data."`

**Good:** `"Transform, clean, and analyze tabular data files (CSV, TSV, Excel). Use this skill whenever the user mentions data cleaning, column transformations, pivot tables, data filtering, or wants to process any spreadsheet-like file — even if they don't explicitly say 'data analysis'."`

## Progressive Disclosure

Skills use a three-level loading system:

1. **Metadata** (name + description) — Always in Claude's context
2. **SKILL.md body** — Loaded when the skill triggers (keep under ~500 lines)
3. **Bundled resources** — Loaded on demand (unlimited size)

**Use references/ when:**
- The skill has multiple distinct phases that don't all need to be in context at once
- Any single phase's instructions exceed ~150 lines
- The skill supports multiple domains/frameworks (e.g., different cloud providers)

**A single SKILL.md is enough when:**
- Simple linear workflow
- Total instructions under ~200 lines
- All steps are tightly coupled

For reference files over 300 lines, include a table of contents at the top.

### Domain Organization

When a skill supports multiple variants, organize by domain:

```
cloud-deploy/
├── SKILL.md (workflow + selection logic)
└── references/
    ├── aws.md
    ├── gcp.md
    └── azure.md
```

Claude reads only the relevant reference file based on the user's context.

## Writing Style

These principles matter because skills are instructions for a smart model, not a checklist for a script runner. Writing well here means the skill handles edge cases gracefully instead of failing on anything unexpected.

### Explain the why, not just the what

Future Claude sessions are smart — give them reasoning so they can handle edge cases, not just rote instructions. If you find yourself writing ALWAYS or NEVER in all caps, reframe: explain why the behavior matters instead.

**Weaker:** `"ALWAYS use kebab-case for file names."`
**Stronger:** `"Use kebab-case for file names — it avoids cross-platform issues with spaces and capitalization, and matches the convention in this project."`

### Use imperative form

"Read the config file" not "You should read the config file."

### Keep it lean

Remove instructions that aren't pulling their weight. If something can be left to Claude's judgment, leave it. Every line in a skill competes for attention — low-value instructions dilute the important ones.

### Include examples where behavior might be ambiguous

Examples are the fastest way to communicate intent. Use them whenever the expected output format, naming convention, or decision logic isn't obvious from the prose alone.

```markdown
## Commit message format
**Example 1:**
Input: Added user authentication with JWT tokens
Output: feat(auth): implement JWT-based authentication
```

### Make it general, not narrow

Write skills that work across many situations, not just the examples you tested with. Use theory of mind — imagine a user in a different project with a different codebase encountering the same type of problem.

## Defining Output Formats

When the skill needs a specific output structure:

```markdown
## Report structure
Use this template:
# [Title]
## Executive summary
## Key findings
## Recommendations
```

## Common Pitfalls

- **Over-specifying**: Don't micro-manage every decision. Trust Claude's judgment for things that don't need to be locked down.
- **Under-triggering descriptions**: If the skill is useful in a situation, say so explicitly in the description.
- **Monolithic SKILL.md**: If you're past 500 lines, split into references.
- **Hardcoded details**: Parameterize anything that varies between uses (paths, names, URLs).
- **Duplicating Claude's built-in abilities**: Don't write instructions for things Claude already does well (reading files, writing code). Focus on the workflow-specific logic that Claude wouldn't know without the skill.
