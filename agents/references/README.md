# agents/references/

**Shipped** reference docs for consumer repos and machines — operating facts
agents should load by trigger (via that repo’s `AGENTS.md` or install path).

Not the same as this catalog’s maintain-only tree (`.agents/references/`), which
is law for editing **agents-setup** itself.

## Rules

Files here MUST be named for a concrete topic agents will match on a task.
Filenames MUST NEVER be generic (`notes.md`, `misc.md`, `reference.md`).

Rationale: Consumers copy or link these into `.agents/references/`; vague names
do not trigger.

Examples:

- Good: `git-remotes.md`
- Bad: `usage.md`, `git.md` alone when the content is estate-wide remotes+CI

Each file MUST be honest estate fact, not aspirational install fiction. When
durable facts change (runner size, host URL, CLI wait law), update this tree in
the same change that learns them.

Rationale: Session surprise is the failure mode this folder exists to prevent.

Examples:

- Good: documenting Forgejo primary + `--wait-until` after verifying the CLI
- Bad: claiming GitHub is origin when origin is Forgejo

Catalog layout MUST keep these under `agents/references/`, never under
`.agents/references/` as the only shipped copy.

Rationale: `.agents/` in this repo is catalog-maintainer law; `agents/` is
payload next to agent markdown.

Examples:

- Good: `agents/references/git-remotes.md`
- Bad: only `.agents/references/git-remotes.md` with no shipped copy
