# .agents/references/

Load-bearing researched facts for agents working on this catalog. Same
authority as root `AGENTS.md` when linked from its Deep reference table.

## Rules

Each file MUST be named for a concrete tech, tool, or topic. Filenames MUST
NEVER be generic (`notes.md`, `research.md`, `misc.md`, `stuff.md`,
`reference.md`).

Rationale: Discoverability depends on specific names; vague files become
undead dumps no agent loads on purpose.

Examples:

- Good: `opencode-skills.md`, `agentskills-spec.md`
- Bad: `notes.md`, `misc.md`

When durable facts are learned from search, docs, or experiment, the agent MUST
add or update a specifically named file here in the same change and MUST add a
**trigger** row in root `AGENTS.md`.

Rationale: Session memory dies; unlinked references are invisible.

Examples:

- Good: new `forgejo-skill-install.md` + trigger line in `AGENTS.md`
- Bad: facts only in chat, or a new file with no trigger link

Reference bodies SHOULD state facts and constraints; binding behavioral rules
for the catalog still MUST use MUST/NEVER/ALWAYS tuples when they are rules
(see root `AGENTS.md` §1).

Rationale: References hold evidence; rules hold enforcement shape.

Examples:

- Good: a reference that records OpenCode discovery paths as facts
- Good: a rule in `AGENTS.md` that MUST place vendor skills under
  `vendor/skills/<owner>/`
