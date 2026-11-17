# AGENTS.md

Binding rules for `softwarepatterns/skills` — the estate catalog of shared
agent skills (and, later, agents/profiles). The same four conventions apply
across `~/Projects/softwarepatterns/*` (public, business) and
`~/Projects/ironspecs/*` (personal, private). Reference files under
`.agents/references/` carry the same authority as this file.

This repository is a **catalog**. Skill payloads here are installed onto other
machines and into other repos. They are not the skills that operate *on* this
repo.

## Conventions

### 1. Rule shape

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
  Rationale: owner folders mirror GitHub and hold provenance READMEs.
  Examples: good `vendor/skills/emilkowalski/animate/`; bad
  `skills/_vendor/animate/`."
- Illegal: "Prefer organizing vendored skills clearly." (no MUST/NEVER/ALWAYS,
  no rationale, no examples)
- Illegal: "Skills MUST be honest." (no rationale, no examples)

### 2. Catalog layout: estate vs vendor

Estate-owned skills MUST live under `skills/`. Vendored skills MUST live under
`vendor/skills/<github-owner>/`, where `<github-owner>` is the GitHub user or
organization that publishes the upstream skill. Agents MUST NEVER place
catalog skill payloads under `.agents/`. Agents MUST NEVER use `_vendor`,
`deps`, `external`, or `third_party` as the vendor path token.

Rationale: This repo distributes skills; it does not apply them to itself.
`.agents/` is reserved for *this* repo's agent law (references). A single
`vendor/skills/<owner>/` tree mirrors GitHub ownership, keeps upstream bytes
greppable, and gives each owner one README for pins and provenance.

Examples:

- Good estate: `skills/release/SKILL.md`, `skills/rust/honesty-pass/SKILL.md`
- Good vendor: `vendor/skills/emilkowalski/animate/SKILL.md`
- Good vendor: `vendor/skills/shadcn/shadcn/SKILL.md`
- Bad: `.agents/skills/animate/SKILL.md` (catalog payload in `.agents`)
- Bad: `skills/vendor/animate/` (vendor under estate tree)
- Bad: `vendor/skills/web-animation/animate/` (topic bucket instead of owner)

Estate-owned skills MUST NEVER sit under `vendor/`. Vendored skill bodies MUST
NEVER be edited in place; estate policy overlays are separate skills under
`skills/` or references under `.agents/references/`.

When one GitHub owner publishes multiple skill sources that would collide on
skill directory name, agents MUST insert a repo segment:
`vendor/skills/<owner>/<repo>/<skill>/`. Until a collision exists, the
two-segment form `vendor/skills/<owner>/<skill>/` is required.

### 3. Owned-folder README

Every structural directory this catalog repo **owns** MUST contain a short
`README.md` that states what the folder is for and the local rules as
**(rule, rationale, examples)** tuples (see §1). Structural directories include
at least: `vendor/`, `vendor/skills/`, `skills/`, `agents/`, `.agents/`,
`.agents/references/`, and each `vendor/skills/<github-owner>/`. Agents MUST
add or update that README in the same change that creates or repurposes the
folder. Agents MUST NEVER add estate READMEs inside a vendored skill package
directory (the tree that contains upstream `SKILL.md` and its upstream files).

Rationale: Owned folders are estate law made navigable. A directory without a
README forces every agent to re-infer purpose and invent layout. Upstream skill
trees are not ours to annotate; owner-level and parent READMEs carry provenance
without forking vendor bytes.

Examples:

- Good: `vendor/README.md`, `vendor/skills/README.md`, `skills/README.md`,
  `agents/README.md`, `vendor/skills/emilkowalski/README.md`
- Bad: no `README.md` under a new `vendor/agents/` structural folder
- Bad: adding `vendor/skills/emilkowalski/animate/README.md` that we authored
  (that package is upstream; only upstream files belong there)

### 4. Vendor owner README

Every `vendor/skills/<github-owner>/` directory MUST contain a `README.md` that
states source repository URL(s), pin ref, how the bits were obtained, and a
table of skill directories with frontmatter `name` and upstream path. The
README MUST NOT claim consumers, mirrors, or install flows that do not exist
yet. Frontmatter `metadata.github-*` on each skill, when present, is
authoritative for pin identity; the README MUST match it. This owner README
satisfies §3 for that folder; it MUST still use honest facts only.

Rationale: Session and chat memory die; re-deriving pins is how wrong versions
enter the estate. One README per owner is the human- and agent-readable index
without editing upstream `SKILL.md` files.

Examples:

- Good: `vendor/skills/emilkowalski/README.md` lists all six pinned skills and
  the shared commit
- Bad: empty owner folder with no README
- Bad: README that lists skills not on disk, or pins that disagree with
  frontmatter

### 5. Agent home in consumer repos (not this catalog's payload tree)

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

### 6. Learned references

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

## Deep reference

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
  README.md
  references/
    README.md
    *.md                      # researched facts
agents/
  README.md                   # estate-owned OpenCode agents
  <agent>.md
skills/
  README.md                   # estate-owned skills
  <topic>/…/                  # optional topic grouping
  <skill>/SKILL.md
vendor/
  README.md
  skills/
    README.md
    <github-owner>/
      README.md               # provenance for that owner
      <skill>/…               # upstream package (no estate README inside)
```

OpenCode skill IDs are path-derived from the leaf directory that contains
`SKILL.md` (see `.agents/references/opencode-skills.md`). Leaf skill directory
names MUST stay unique across `skills/` and `vendor/skills/` combined.

Migration and collection MUST move or copy bytes without rewriting skill or
agent bodies except when the task is explicitly to change that content.

Estate OpenCode agent definitions MUST live under `agents/` as markdown agent
files. Catalog agent payloads MUST NEVER live under `.agents/` in this repo.

Rationale: Same catalog-vs-consumer split as skills; see `agents/README.md`.

Examples:

- Good: `agents/riker.md`, `agents/kino.md`
- Bad: only copy under `.agents/agents/riker.md` as the shipped source

## Handoff checklist

Before finishing a task:

1. Explain what changed.
2. List commands that were run.
3. List anything not verified.
4. Point to updated docs (`AGENTS.md`, owner READMEs, `.agents/references/*`).
5. Confirm every new or edited rule is a full (rule, rationale, examples)
   tuple with MUST/NEVER/ALWAYS.
6. Confirm any new durable tech facts landed in a specifically named
   `.agents/references/*.md` with a trigger row in this file.
7. Confirm vendored paths are `vendor/skills/<owner>/…` with an owner README,
   and that no catalog payload landed under `.agents/`.
8. Confirm every new or repurposed structural folder this repo owns has a
   short `README.md` with rule tuples; no estate README inside vendor skill
   packages.
9. Confirm estate agents live under `agents/` (not `.agents/`), copied without
   body rewrites unless the task is explicitly content work.
