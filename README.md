# softwarepatterns/skills

The estate's agent skills — the canonical source of truth for skills shared
across the estate's repos and machines. Standard
[Agent Skills](https://agentskills.io) layout: `skills/<name>/SKILL.md`.

Forgejo is the **canonical** host; `main` is mirrored to GitHub
(`github.com/softwarepatterns/skills`) for public distribution.

## Skills

| Skill | Purpose | Origin |
| --- | --- | --- |
| `honesty-pass` | Hard gate for dishonest code contracts (hidden panics, fake fallibility, thesaurus code) | ironspecs/clis-rs |
| `release` | The five-step Forgejo release model (tag → draft → legs → cap → publish) | ironspecs/clis-rs |
| `shadcn` | shadcn/ui CLI component rules (composition, tokens, structure) | softwarepatterns/atelier-memoir |

## Install

Any git URL works with the ecosystem CLI (SSH auth already configured on
estate machines):

```sh
# all skills, for OpenCode, project scope (writes .agents/skills + skills-lock.json)
npx skills add ssh://git@forgejo.softwarepatterns.com:2222/softwarepatterns/skills.git -a opencode

# one skill, pinned to a release tag
npx skills add ssh://git@forgejo.softwarepatterns.com:2222/softwarepatterns/skills.git@v0.1.0 \
    --skill honesty-pass -a opencode -g -y
```

Via the GitHub mirror (GitHub CLI ≥ 2.90):

```sh
gh skill install softwarepatterns/skills honesty-pass
```

Skills execute with the agent's full permissions — preview before
installing, pin the tag you reviewed, and review the lockfile diff on
update (`npx skills check` / `npx skills update`).

## Versioning

Tag releases (`vX.Y.Z`). Repos pin the tag they install. Update flow:
`npx skills check` → install the new tag on a branch → review the
`skills-lock.json` diff → merge.

## Repos that vendor from here

- `softwarepatterns/atelier-memoir` — `honesty-pass`, `shadcn` (locked)
