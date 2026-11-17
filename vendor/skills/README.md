# vendor/skills/

Vendored **agent skills**, grouped by **GitHub owner** (user or org), matching
the upstream publishing identity.

## Rules

Each immediate child directory MUST be a GitHub owner name (`emilkowalski`,
`anthropics`, `shadcn`, …). Agents MUST NEVER use topic names (`web-design`,
`web-animation`), skill names, or synonym tokens (`_vendor`, `deps`) as the
owner segment.

Rationale: Owner folders mirror `github.com/<owner>/…`, keep multi-skill packs
together, and give provenance one home per upstream identity.

Examples:

- Good: `vendor/skills/emilkowalski/animate/`
- Good: `vendor/skills/shadcn/shadcn/`
- Bad: `vendor/skills/web-animation/animate/`
- Bad: `vendor/skills/emil-design-eng/emil-design-eng/` as a fake owner

Every owner directory MUST contain a `README.md` with source URL(s), pin ref,
how bits were obtained, and a table of skill dirs → frontmatter `name` →
upstream path. The README MUST match on-disk skills and frontmatter
`metadata.github-*` when present. The README MUST NOT claim install flows,
mirrors, or consumers that do not exist.

Rationale: Pins and provenance die in chat history; the owner README is the
durable index without editing upstream `SKILL.md` files.

Examples:

- Good: `vendor/skills/emilkowalski/README.md` lists six skills and one shared
  commit
- Bad: owner folder with skills but no README
- Bad: README pin that disagrees with `github-pinned` in frontmatter

Skill packages MUST be the upstream directory shape (`SKILL.md` + upstream
companions). Agents MUST NEVER add estate-authored files inside those
packages.

Rationale: Vendor packages stay re-installable and diffable against upstream;
estate annotation belongs in the owner README or outside `vendor/`.

Examples:

- Good: only upstream files under `vendor/skills/leonxlnx/taste-skill/`
- Bad: adding `NOTES.md` beside vendored `SKILL.md`

When one owner publishes multiple skill sources that collide on skill
directory name, agents MUST use `vendor/skills/<owner>/<repo>/<skill>/`.
Until a collision exists, agents MUST use `vendor/skills/<owner>/<skill>/`.

Rationale: Avoid path ambiguity without premature nesting.

Examples:

- Good today: `vendor/skills/pbakaus/impeccable/`
- Good if needed later: `vendor/skills/acme/skills-a/foo/` and
  `vendor/skills/acme/skills-b/foo/`
