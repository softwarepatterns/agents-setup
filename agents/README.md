# agents/

Estate-owned **OpenCode agent** definitions (markdown agent files). This catalog
ships them; it does not apply them to itself via `.agents/`.

## Provenance

| Field | Value |
| --- | --- |
| Collected from | `~/.config/opencode/agents/` on the authoring machine |
| Files | `kino.md`, `lumen.md`, `rhyme.md`, `riker.md` |
| References | `references/` — shipped operating docs (e.g. `git-remotes.md`) for consumers |
| Not included | OpenCode built-ins (`build`, `plan`, `general`, `explore`); skill-nested `agents/` junk; `opencode.jsonc`; retired `teach` |

## Permissions (all estate agents)

Every agent file MUST declare this ordered `permissions` block (last match
wins, OpenCode V2):

```yaml
permissions:
  - action: "*"
    resource: "*"
    effect: allow
  - action: question
    resource: "*"
    effect: deny
  - action: shell
    resource: "git *"
    effect: ask
```

Rationale: Full tool access is the default working mode; the question tool is
never allowed (agents must not block on interactive Q&A); any `git` shell
invocation requires human approval.

Examples:

- Good: all four agents share the block above
- Bad: omitting `question` deny (tool becomes default-ask or allow)
- Bad: allowing `git *` without `ask` (unattended history mutation)

## Rules

Estate agent definitions MUST live under this tree as `*.md` agent files.
Agents MUST NEVER place catalog agent payloads under `.agents/` in this repo.

Rationale: Same catalog-vs-consumer split as `skills/`. `.agents/` here is law
for maintaining the catalog; `agents/` is what other machines install.

Examples:

- Good: `agents/riker.md`
- Bad: `.agents/agents/riker.md` as the only copy of a shipped agent

`kino.md` and `lumen.md` MUST both remain until an explicit task decides
otherwise. They share an architect role and differ by model (and minor prompt
detail).

Rationale: Two primaries on the source machine is real configuration, not
accidental duplication to collapse casually.

Examples:

- Good: both files present with distinct `model` frontmatter
- Bad: deleting one as “redundant” without an explicit task

Consumer install targets (later) are `~/.config/opencode/agents/` or a
consumer repo’s harness agent path. This folder MUST NOT embed secrets from
`opencode.jsonc`.

Rationale: Agent markdown is shareable role law; machine tokens are not.

Examples:

- Good: agent files with description, mode, model, permissions, body only
- Bad: copying MCP tokens or API keys into this tree
