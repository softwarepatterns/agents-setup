# agents-setup

A catalog of agent skills, OpenCode agents, and reference documents. Consumers
install from here; this repo does not apply them to itself.

## Where this lives

| Role | URL |
|---|---|
| **Primary** (source of truth, PRs, tags) | https://forgejo.softwarepatterns.com/softwarepatterns/agents-setup |
| **Mirror** (`main` only, automated) | https://github.com/softwarepatterns/agents-setup |

Day-to-day work targets **Forgejo**. GitHub is an automated mirror of `main` so
tools that only speak GitHub shorthand (`owner/repo`) can still install skills.
Prefer Forgejo git URLs when you can; use the GitHub mirror when `npx skills` /
`gh skill` require GitHub hosting.

Git over SSH on the primary host uses port **2222**:

```text
ssh://git@forgejo.softwarepatterns.com:2222/softwarepatterns/agents-setup.git
```

## Why install instead of copy

A skill written into one repo stays there; each new repo re-derives or drifts.
Install with a **pinned tag** so `skills-lock.json` records source, ref, commit,
and hash. Do not hand-copy skill bodies in place of a pinned install.

Estate layout in a **consumer** repo (OpenCode discovers these):

```text
.agents/skills/<skill-id>/SKILL.md
skills-lock.json
```

## Install with `npx skills`

Requires Node. The CLI is the [`skills`](https://www.npmjs.com/package/skills)
package (`npx skills`).

### List what this catalog ships

```bash
# Via GitHub mirror (shorthand)
npx skills add softwarepatterns/agents-setup -l --full-depth

# Via primary Forgejo (git URL)
npx skills add \
  ssh://git@forgejo.softwarepatterns.com:2222/softwarepatterns/agents-setup.git \
  -l --full-depth
```

`--full-depth` is required here: skills live under topic folders
(`skills/docs/…`, `skills/tools/…`), not only `skills/<id>/`.

### Install one or more skills (project scope, OpenCode)

Pin a **tag**. Estate skills use per-skill tags `<skill-id>/vMAJOR.MINOR.PATCH`
(example: `forgejo/v0.2.0`, `external-references/v0.2.0`). Older catalog-wide
tags `v0.1.0`–`v0.6.0` still exist but are not the pattern for new releases.

```bash
# GitHub mirror + per-skill tag (typical for npx skills)
npx skills add 'softwarepatterns/agents-setup@forgejo/v0.2.0' \
  -s forgejo -a opencode -y --full-depth

npx skills add 'softwarepatterns/agents-setup@external-references/v0.2.0' \
  -s external-references -a opencode -y --full-depth

# Several skills from one catalog-wide legacy tag (only if you intend that pin)
npx skills add 'softwarepatterns/agents-setup@v0.6.0' \
  -s external-references -s forgejo -a opencode -y --full-depth

# Primary Forgejo over SSH + tag
npx skills add \
  'ssh://git@forgejo.softwarepatterns.com:2222/softwarepatterns/agents-setup.git@forgejo/v0.2.0' \
  -s forgejo -a opencode -y --full-depth
```

Useful flags:

| Flag | Meaning |
|---|---|
| `-s, --skill` | Skill id(s) to install (leaf directory name) |
| `-a, --agent` | Target agent (`opencode`, `claude-code`, …) or placement |
| `-y, --yes` | Non-interactive |
| `-g, --global` | User-level install instead of project |
| `-l, --list` | List only |
| `--full-depth` | Search nested `skills/**/SKILL.md` (needed for this repo) |

### Lockfile and restore

Install writes `skills-lock.json` at the consumer root. Commit it.

```bash
# Restore from lock (CI / new clone)
npx skills experimental_install
# aliases: install, i
```

`ref` plus `commit` pin the bytes; restore fetches that commit, not “latest main”.

### Update

```bash
npx skills update forgejo -y
# or project-wide
npx skills update -p -y
```

## Install with `gh skill` (GitHub CLI)

[`gh skill`](https://cli.github.com/) (preview) installs from **GitHub**
repositories. Use the **mirror**:

```bash
# List
gh skill install softwarepatterns/agents-setup

# Install one skill, pin tag
gh skill install softwarepatterns/agents-setup forgejo \
  --pin forgejo/v0.2.0 \
  --agent opencode \
  --scope project

# Or skill@version form
gh skill install softwarepatterns/agents-setup forgejo@forgejo/v0.2.0 \
  --agent opencode \
  --scope project
```

Estate consumers still want skills under `.agents/skills/` (shared by OpenCode
and several other agents at project scope). Prefer `--agent opencode` or an
agent that resolves to `.agents/skills`, or `--dir .agents/skills` when you need
an explicit path.

`gh skill` does not replace Forgejo as the source of truth; it only consumes the
GitHub mirror.

## What is in the catalog

- **First-party skills:** `skills/` (docs, tools, rust, typescript, …)
- **First-party OpenCode agents:** `agents/`
- **References for consumers:** `agents/references/` (e.g. remotes, OpenCode skills layout)
- **Vendored third-party skills:** `vendor/skills/<github-owner>/` — pin and provenance in each owner `README.md`
- **Rules for this repo:** `AGENTS.md` and `.agents/references/`

## Versioning skills

Memorable releases use tag `<skill-id>/vMAJOR.MINOR.PATCH` on the release
commit (see `AGENTS.md`). Consumers pin that tag in `npx skills add …@tag` or
`gh skill install … --pin tag`.
