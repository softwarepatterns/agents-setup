# OpenCode skills (V2)

Facts from OpenCode V2 docs (`https://opencode.ai/v2/docs/skills`). Load when
changing skill layout, install paths, IDs, or catalog publishing.

## Discovery locations

OpenCode searches, in addition to config-declared sources:

| Scope | Paths |
| --- | --- |
| Global | `~/.config/opencode/skills` |
| Global compatibility | `~/.claude/skills`, `~/.agents/skills` |
| Project | `.opencode/skills` |
| Project compatibility | `.claude/skills`, `.agents/skills` |

Project sources are walked from the current directory up to the project root.

Estate canonical home is still `.agents/skills/` (see root `AGENTS.md`).
Harness-named dirs are compatibility discovery only, not a second authoring
root.

## Forms on disk

Under a source root:

- Flat: `skills/review.md` → skill id `review`
- Directory: `skills/git-release/SKILL.md` → id `git-release`
- Nested groups: `skills/teams/release/SKILL.md` → id **`release`** (leaf only)

Rules:

- A directory that contains `SKILL.md` is one skill; OpenCode does not search
  inside it for more skills.
- Search stops six directories below the source root.
- Hidden dirs and `node_modules` are skipped.

## Config sources

`opencode.json(c)` may set:

```jsonc
{
  "skills": [
    "./team-skills",
    "~/shared/opencode-skills",
    "/opt/company-skills",
    "https://example.com/opencode/skills/"
  ]
}
```

- Relative paths resolve from the active working directory, not the config file
  path.
- `skills` arrays from discovered configs are combined, not replaced.

## HTTP catalogs

Base URL serves `index.json`:

```json
{
  "skills": [
    {
      "name": "git-release",
      "version": "3",
      "files": ["git-release.md", "references/release-policy.md"]
    }
  ]
}
```

Files download from `<base>/<name>/<file>`. Bump `version` when files change so
caches refresh.

## Frontmatter (OpenCode)

| Field | Role |
| --- | --- |
| `name` | Display label; defaults from path id |
| `description` | Advertised to the model; skills without it are not advertised |
| `metadata.opencode/autoinvoke: false` | Registered but not auto-listed |
| `disable-model-invocation: true` | Same effect; `opencode/autoinvoke` wins if both set |

Portable fields such as `license` and `compatibility` are accepted and not
interpreted by OpenCode V2.

## IDs and precedence

- Runtime id is **path-derived**, case-sensitive, not the frontmatter display
  name.
- Duplicate ids: later source wins. Registration order (low → high): built-ins →
  `.claude` → `.agents` → `~/.config/opencode/skills` → project `.opencode/skills`
  → explicit `skills` config entries.

## Loading

Model sees id + name + description only. Full body loads via the `skill` tool
(or user `@skill-id`). Supporting files are not auto-injected; the agent reads
them when instructed. Directory skills get a sample of up to ten neighboring
paths.
