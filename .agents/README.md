# .agents/

Agent law for work **on this catalog repo** — not the skill payloads the catalog
distributes.

## Rules

This tree MUST hold references (and later repo-local agents/commands if needed)
for agents maintaining the catalog. Catalog skill payloads MUST NEVER live
under `.agents/`; they belong in `../skills/` and `../vendor/skills/`.

Rationale: This repository is a distribution root. Putting payloads in
`.agents/skills/` confuses “skills that operate on this repo” with “skills this
repo ships to other machines.”

Examples:

- Good: `.agents/references/opencode-skills.md`
- Bad: `.agents/skills/emilkowalski/animate/`

Harness-named roots (`.claude/`, `.opencode/`, `.codex/`) MUST NEVER be the
canonical authoring home in estate consumer repos; see root `AGENTS.md`.
This catalog repo MUST keep `.agents/` minimal and honest.

Rationale: One name across the estate; no second source of truth.

Examples:

- Good: only real files under `.agents/references/`
- Bad: empty stub trees that pretend skills are installed here
