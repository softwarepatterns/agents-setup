# Agent Skills specification

Facts from the open Agent Skills standard (`https://agentskills.io/specification`).
Load when authoring or validating `SKILL.md` frontmatter, packing optional
skill files, or checking name/directory alignment.

## Directory

A skill is a directory with at least `SKILL.md`:

```text
skill-name/
├── SKILL.md       # required
├── scripts/       # optional executable code
├── references/    # optional on-demand docs
├── assets/        # optional templates/resources
└── ...
```

## Frontmatter

| Field | Required | Constraints |
| --- | --- | --- |
| `name` | yes | 1–64 chars; `a-z`, `0-9`, hyphens; no leading/trailing/consecutive hyphens; **MUST match parent directory name** |
| `description` | yes | 1–1024 chars; what it does and when to use it |
| `license` | no | License name or bundled license file reference |
| `compatibility` | no | ≤500 chars; environment requirements |
| `metadata` | no | string → string map |
| `allowed-tools` | no | experimental space-separated tool list |

Minimal:

```markdown
---
name: skill-name
description: A description of what this skill does and when to use it.
---
```

## Progressive disclosure

1. Metadata (~name + description) at startup for all skills
2. Full `SKILL.md` body on activation (keep under ~500 lines / ~5000 tokens when practical)
3. `scripts/`, `references/`, `assets/` only when the skill directs a read or run

File references from `SKILL.md` should stay one level deep.

## Validation

Reference library: `skills-ref validate ./my-skill` (see agentskills GitHub
`skills-ref`).

## Relation to OpenCode

OpenCode discovers directory `SKILL.md` skills and also flat `*.md` skills.
OpenCode's **runtime id** is path-derived (leaf directory or flat stem). Keep
spec `name`, leaf directory, and intended id aligned for portable skills.
