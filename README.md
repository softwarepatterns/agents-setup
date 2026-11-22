# agents-setup

A catalog of agent skills, OpenCode agents, and reference documents. Consumers
install from here; this repo does not apply them to itself.

A skill written into one repo stays there; each new repo re-derives or drifts.
One versioned source to install from: an `opencode.json` path, `npx skills
add` with this repo's git URL, or a copy into `.agents/skills/`.

- **First-party skills:** `skills/`
- **First-party OpenCode agents:** `agents/`
- **References for consumers:** `agents/references/` (e.g. `git-remotes.md`)
- **Vendored third-party skills:** `vendor/skills/<github-owner>/` — pinned version and provenance in each owner's `README.md`
- **Rules for this repo:** `AGENTS.md` and `.agents/references/`
