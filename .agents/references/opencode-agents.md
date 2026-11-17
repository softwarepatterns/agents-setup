# OpenCode agents (V2)

Facts from OpenCode V2 docs (`https://opencode.ai/v2/docs/agents`). Load when
adding or wiring shared agents, modes, or agent permissions.

## Locations

| Scope | Path |
| --- | --- |
| Global | `~/.config/opencode/agents/<name>.md` |
| Project | `.opencode/agents/<name>.md` |

Nested paths become part of the agent id:

```text
.opencode/agents/team/reviewer.md  →  team/reviewer
```

Estate authoring home is `.agents/agents/` (see root `AGENTS.md`). Install or
symlink into harness discovery paths; do not fork content into harness-named
trees as a second source of truth.

## Formats

### Markdown agent

Frontmatter mirrors an `agents` config entry. Body = `system` prompt.

```md
---
description: Reviews changes for correctness and regressions
mode: subagent
model: anthropic/claude-sonnet-5-5#high
permissions:
  - action: edit
    resource: "*"
    effect: deny
---

Review the current changes. List findings in severity order.
```

### JSONC

Under `agents` in `opencode.json(c)`:

```jsonc
{
  "agents": {
    "reviewer": {
      "description": "Reviews current changes",
      "mode": "subagent",
      "system": "Report findings in severity order.",
      "permissions": [
        { "action": "edit", "resource": "*", "effect": "deny" }
      ]
    }
  }
}
```

## Modes

| Mode | Behavior |
| --- | --- |
| `primary` | Main session agent (default for new custom agents) |
| `subagent` | Child session via `subagent` tool only |
| `all` | Either |

Subagents run with fresh context. Parent `subagent` permissions gate which
child agents may launch.

## Built-ins (override by same id)

| Id | Mode | Role |
| --- | --- | --- |
| `build` | primary | Default coding agent |
| `plan` | primary | Plan without editing normal project files |
| `general` | subagent | Multi-step research; cannot launch further subagents |
| `explore` | subagent | Read/search only |

## Options worth pinning in shared agents

- `description` — required for subagent discovery by the parent model
- `model` — `provider/model` or `provider/model#variant`
- `permissions` — ordered rules; last match wins (`allow` / `ask` / `deny`)
- `steps` — max model steps before forced summary
- `hidden` / `disabled` — visibility vs removal

Common permission actions: `shell`, `edit`, `read`, `subagent`, `skill`,
`webfetch`, `websearch`.
