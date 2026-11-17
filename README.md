# softwarepatterns/agents-setup

Estate catalog of agent skills and OpenCode agents shared across machines and
repos.

- **Estate-owned skills:** `skills/`
- **Estate-owned agents:** `agents/`
- **Shipped agent references:** `agents/references/` (e.g. `git-remotes.md`)
- **Vendored skills (upstream):** `vendor/skills/<github-owner>/` — see each owner's `README.md` for pins
- **Law for work on this repo:** `AGENTS.md` and `.agents/references/`

This repository does not apply its skill or agent payloads to itself; consumers
install from here into their own harness paths (e.g. `.agents/skills/`,
`~/.config/opencode/agents/`).
