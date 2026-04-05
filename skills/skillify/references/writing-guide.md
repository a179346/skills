# Skill Writing Guide

This is a fallback reference for when `/skill-creator` is not available. It covers the essential knowledge for writing well-structured Claude Code skills.

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

The description field is critical — it's how Claude decides whether to consult the skill. Two principles:

1. **Be specific.** Include both what the skill does AND the contexts/phrasings that should activate it.

2. **Be slightly pushy.** Claude tends to under-trigger skills. Explicitly list the situations where the skill should activate, even non-obvious ones.

Bad: `"A skill for working with data."`

Good: `"Transform, clean, and analyze tabular data files (CSV, TSV, Excel). Use this skill whenever the user mentions data cleaning, column transformations, pivot tables, data filtering, or wants to process any spreadsheet-like file — even if they don't explicitly say 'data analysis'."`

## Progressive Disclosure

Skills use a three-level loading system:

1. **Metadata** (name + description) — Always in Claude's context
2. **SKILL.md body** — Loaded when the skill triggers (keep under ~500 lines)
3. **Bundled resources** — Loaded on demand (unlimited size)

When to use references/:
- The skill has multiple distinct phases that don't all need to be in context at once
- Any single phase's instructions exceed ~150 lines
- The skill supports multiple domains/frameworks (e.g., different cloud providers)

When a single SKILL.md is enough:
- Simple linear workflow
- Total instructions under ~200 lines
- All steps are tightly coupled

For reference files over 300 lines, include a table of contents at the top.

## Writing Style

**Explain the why.** Today's LLMs are smart. When given reasoning, they can handle edge cases instead of blindly following rules. If you find yourself writing ALWAYS or NEVER in all caps, reframe — explain why the behavior matters instead.

**Use imperative form** for instructions: "Read the config file" not "You should read the config file."

**Keep it lean.** Remove instructions that aren't pulling their weight. If something can be left to Claude's judgment, leave it.

**Include examples** where the expected behavior might be ambiguous:

```markdown
## Commit message format
**Example 1:**
Input: Added user authentication with JWT tokens
Output: feat(auth): implement JWT-based authentication
```

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

## Domain Organization

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

## Common Pitfalls

- **Over-specifying**: Don't micro-manage every decision. Trust Claude's judgment for things that don't need to be locked down.
- **Under-triggering descriptions**: If the skill is useful in a situation, say so explicitly in the description.
- **Monolithic SKILL.md**: If you're past 500 lines, split into references.
- **Hardcoded details**: Parameterize anything that varies between uses (paths, names, URLs).
