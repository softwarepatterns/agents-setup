# AGENTS.md

Binding rules for `softwarepatterns/agents-setup` — the estate catalog of
shared agent skills (and agents/profiles). The same conventions apply across
`~/Projects/softwarepatterns/*` (public, business) and
`~/Projects/ironspecs/*` (personal, private). Reference files under
`.agents/references/` carry the same authority as this file.

This repository is a **catalog**. Skill payloads here are installed onto other
machines and into other repos. They are not the skills that operate *on* this
repo.

## Conventions

### Rule shape

Every rule MUST be a prose tuple of **(rule, rationale, examples)**. The rule
sentence MUST use **MUST**, **NEVER**, or **ALWAYS**. A rule that omits the
rationale, omits examples, or softens the modal (should / preferably /
generally) is illegal and MUST be rewritten or deleted before merge.

Rationale: Agents skim prose and obey modals. A rule without a rationale
becomes superstition and mutates under pressure. A rule without examples is
ambiguous at the call site, so two agents enforce two different laws. The
tuple is the minimum unit that survives handoff.

Examples:

- Legal: "Vendored skills MUST live under `vendor/skills/<github-owner>/`.
  Rationale: owner folders mirror GitHub and keep provenance in one place.
  Examples: good `vendor/skills/emilkowalski/animate/`; bad
  `skills/_vendor/animate/`."
- Illegal: "Prefer organizing vendored skills clearly." (no MUST/NEVER/ALWAYS,
  no rationale, no examples)
- Illegal: "Skills MUST be honest." (no rationale, no examples)

### Catalog layout: estate vs vendor

Estate-owned skills MUST live under `skills/`. Vendored skills MUST live under
`vendor/skills/<github-owner>/`, where `<github-owner>` is the GitHub user or
organization that publishes the upstream skill. Agents MUST NEVER place
catalog skill payloads under `.agents/`. Agents MUST NEVER use `_vendor`,
`deps`, `external`, or `third_party` as the vendor path token.

Rationale: This repo distributes skills; it does not apply them to itself.
`.agents/` is reserved for *this* repo's agent law (references). A single
`vendor/skills/<owner>/` tree mirrors GitHub ownership, keeps upstream bytes
greppable, and keeps provenance in each skill's frontmatter `metadata`.

Examples:

- Good estate: `skills/release/SKILL.md`, `skills/rust/honesty-pass-rs/SKILL.md`
- Good vendor: `vendor/skills/emilkowalski/animate/SKILL.md`
- Good vendor: `vendor/skills/shadcn/shadcn/SKILL.md`
- Bad: `.agents/skills/animate/SKILL.md` (catalog payload in `.agents`)
- Bad: `skills/vendor/animate/` (vendor under estate tree)
- Bad: `vendor/skills/web-animation/animate/` (topic bucket instead of owner)

Estate-owned skills MUST NEVER sit under `vendor/`. Vendored skill bodies MUST
NEVER be edited in place, and estate files MUST NEVER be added inside a
vendored skill package directory (the tree that contains upstream `SKILL.md`
and its upstream files); estate policy overlays are separate skills under
`skills/` or references under `.agents/references/`.

When one GitHub owner publishes multiple skill sources that would collide on
skill directory name, agents MUST insert a repo segment:
`vendor/skills/<owner>/<repo>/<skill>/`. Until a collision exists, the
two-segment form `vendor/skills/<owner>/<skill>/` is required.

Vendored provenance MUST live in each vendored skill's own frontmatter
(`metadata.github-repo`, `metadata.github-ref`/`metadata.github-pinned`,
`metadata.github-path`). Agents MUST NOT fork vendor bytes to record
provenance.

Rationale: Session and chat memory die; re-deriving pins is how wrong versions
enter the estate. Frontmatter travels with the skill bytes and survives
install, so the pin cannot be separated from the payload.

Examples:

- Good: `metadata.github-pinned: 2d228630…` inside the vendored `SKILL.md`
- Bad: an estate-authored README inside `vendor/skills/<owner>/<skill>/`
  recording the pin (duplicates frontmatter, drifts, and edits vendor trees)

### Agent home in consumer repos (not this catalog's payload tree)

In repos that *consume* skills (application and infrastructure repos),
agent-facing layout MUST use the directory name `.agents`. Skills, references,
and repo-scoped agent definitions there MUST live under `.agents/`. Agents MUST
NEVER introduce `.claude/`, `.opencode/`, `.codex/`, or other harness-named
roots as the estate's canonical tree in those repos.

Rationale: Harness names couple the estate to one product's on-disk fashion.
`.agents` is already the shared convention across ironspecs and
softwarepatterns consumer repos.

Examples:

- Good in a consumer repo: `.agents/skills/`, `.agents/references/`,
  `.agents/agents/`
- Bad in a consumer repo: `.opencode/skills/` as the canonical authoring home
- Good in *this* catalog repo: `.agents/references/` only for catalog law;
  payloads in `skills/` and `vendor/skills/`

Compatibility shims outside `.agents/` in consumer repos (if a harness
requires them) MUST be generated or symlinked from `.agents/` and MUST NEVER
become a second source of truth.

### Learned references

When a web search, doc fetch, or experiment yields durable facts about a
specific tech, tool, or topic, the agent MUST write those facts to
`.agents/references/<specific-name>.md` in the same change of work. The
filename MUST name the concrete tech, tool, or topic. The filename MUST NEVER
be generic or vague (`notes.md`, `research.md`, `misc.md`, `stuff.md`,
`reference.md`).

`AGENTS.md` MUST link each such file with a **trigger**: one line that states
when an agent MUST load that file into context. Triggers live in the Deep
reference table below (or an equivalent table in a consumer repo's
`AGENTS.md`).

Rationale: Session memory dies; re-deriving vendor behavior is how wrong facts
enter the estate (see `softwarepatterns/forgejo-actions` and
`ansible-infrastructure`). Specific names and explicit triggers make the
corpus discoverable without dumping every reference into every prompt.

Examples:

- Good file: `.agents/references/opencode-skills.md`
- Good file: `.agents/references/agentskills-spec.md`
- Bad file: `.agents/references/notes.md`
- Bad file: `.agents/references/research.md`
- Good trigger: "OpenCode skill discovery, nested `SKILL.md` IDs, catalogs, or
  `skills` config paths → `.agents/references/opencode-skills.md`"

### Headings are names, not numbers

Estate documents — this file, estate skills under `skills/`, agents under
`agents/`, and references under `.agents/references/` and
`agents/references/` — MUST NOT use numbered headings (`## 1. Foo`,
`### 2.3 Bar`). Headings MUST be stable names, and cross-references MUST cite
the heading name, never a number. Vendored skills under `vendor/` are
upstream bytes and are exempt; they MUST NOT be edited to comply.

Rationale: A number expires the moment a section is inserted or removed above
it, and every stale number is a silent lie that routes agents to the wrong
section. Names survive reordering, so references cannot rot.

Examples:

- Good: `## Claims and accountability`, cited as "see **Revoke expired
  claims**"
- Bad: `## 5. Claims and accountability` with "per 5.3" (dies when a section
  lands above it)
- Bad: renumbering forty headings because one was inserted

### Release tags (per skill / per thing)

Memorable releases of an estate skill, agent, or other named catalog payload
MUST be tagged `<thing>/vMAJOR.MINOR.PATCH` (optional `v` prefix on the
semver segment is required as shown: `v` + semver). For estate skills,
`<thing>` MUST be the leaf skill directory name (the OpenCode skill id)—the
directory that contains `SKILL.md`—not a topic path and not the repo name.
Agents MUST NEVER create new catalog-wide version tags of the form `vN.N.N`
for skill releases. Historical tags `v0.1.0`–`v0.6.0` are legacy catalog-wide
markers; they MUST NOT be used as the pattern for new skill releases.

Rationale: Skills version independently. A single repo `vN.N.N` cannot say
which skill moved. `<skill-id>/vN.N.N` is greppable, pushable, and matches
how consumers name and pin one skill. Topic folders (`docs/`, `rust/`) are
not identities; leaf ids are.

Examples:

- Good: `spec-driven-design/v0.7.0` on the commit that releases that skill
- Good: `issue-management/v0.2.0` when only that skill changes
- Good: `riker/v1.0.0` for an estate agent under `agents/` when agents are
  tagged the same way
- Bad: `v0.7.0` as a new skill release (catalog-wide; legacy only)
- Bad: `docs/spec-driven-design/v0.7.0` (topic path is not the skill id)
- Bad: `skills/spec-driven-design/v0.7.0` (path prefix noise)

When releasing one skill, agents MUST tag the release commit with that
skill's next `<skill-id>/vN.N.N` and MUST push the tag with the commit.
Bump MINOR for meaningful skill behavior or law changes; PATCH for typos and
non-behavioral fixes; MAJOR when consumers must treat the skill as
incompatible.

## Deep reference

### Shipped for consumers (`agents/references/`)

| Trigger | Document |
| --- | --- |
| Git remotes, clone paths `owner/repo`, Forgejo vs GitHub mirror, `softwarepatterns` vs `ironspecs`, `forgejo` CLI, `--wait-until`, `.forgejo` workflows, bookworm LXC runner, nested dockerd, per-repo `/cache` | `agents/references/git-remotes.md` |

Consumer repos that install shipped references from this catalog MUST place
them under their own `.agents/references/` and MUST repeat the matching
trigger row in **their** `AGENTS.md`.

Rationale: A reference without a local trigger is invisible to agents.

Examples:

- Good: `.agents/references/git-remotes.md` + trigger table row in consumer
  `AGENTS.md`
- Bad: file present on disk with no trigger link

### Catalog-maintainer only (`.agents/references/`)

| Trigger | Document |
| --- | --- |
| OpenCode skill discovery paths, nested `SKILL.md` layout, path-derived IDs, HTTP catalogs, `skills` config entries, precedence | `.agents/references/opencode-skills.md` |
| OpenCode agent files, modes (`primary` / `subagent` / `all`), locations, permissions shape | `.agents/references/opencode-agents.md` |
| Agent Skills standard: `SKILL.md` frontmatter, `name` ↔ directory match, progressive disclosure, optional `scripts/` / `references/` / `assets/` | `.agents/references/agentskills-spec.md` |

## Layout (this repo)

```text
AGENTS.md
README.md
.agents/
  references/
    *.md                      # researched facts
agents/
  <agent>.md                  # estate-owned OpenCode agents
  references/
    git-remotes.md            # shipped refs for other repos/machines
skills/
  <topic>/…/                  # optional topic grouping
  <skill>/SKILL.md            # estate-owned skills
vendor/
  skills/
    <github-owner>/
      <skill>/…               # upstream package (no estate files inside)
```

OpenCode skill IDs are path-derived from the leaf directory that contains
`SKILL.md` (see `.agents/references/opencode-skills.md`). Leaf skill directory
names MUST stay unique across `skills/` and `vendor/skills/` combined.

Rationale: The leaf name is the skill's identity at install and discovery
time. Two skills sharing a leaf name collide into one ID, and installers
cannot select between them.

Examples:

- Good: `skills/rust/honesty-pass-rs/` and
  `skills/typescript/honesty-pass-ts/` (distinct leaves)
- Bad: `skills/rust/honesty-pass/` beside `skills/typescript/honesty-pass/`
  (both resolve to ID `honesty-pass`)

Migration and collection MUST move or copy bytes without rewriting skill or
agent bodies except when the task is explicitly to change that content.

Estate OpenCode agent definitions MUST live under `agents/` as markdown agent
files. Catalog agent payloads MUST NEVER live under `.agents/` in this repo.

Rationale: Same catalog-vs-consumer split as skills.

Examples:

- Good: `agents/riker.md`, `agents/kino.md`
- Bad: only copy under `.agents/agents/riker.md` as the shipped source

## Handoff checklist

Before finishing a task:

1. Explain what changed.
2. List commands that were run.
3. List anything not verified.
4. Point to updated docs (`AGENTS.md`, `.agents/references/*`).
5. Confirm every new or edited rule is a full (rule, rationale, examples)
   tuple with MUST/NEVER/ALWAYS.
6. Confirm any new durable tech facts landed in a specifically named
   `.agents/references/*.md` with a trigger row in this file.
7. Confirm vendored paths are `vendor/skills/<owner>/…`, no estate files
   landed inside a vendored skill package, and no catalog payload landed
   under `.agents/`.
8. Confirm no estate file gained or kept a numbered heading; headings are
   names (see **Headings are names, not numbers**).
9. Confirm estate agents live under `agents/` (not `.agents/`), copied without
   body rewrites unless the task is explicitly content work.
10. Confirm skill (or other payload) releases use `<thing>/vN.N.N` tags
    (see **Release tags**)—not new catalog-wide `vN.N.N` tags.
