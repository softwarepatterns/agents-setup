# vendor/

Estate-owned root for **third-party bytes** this catalog pins and redistributes.
Nothing under `vendor/` is authored skill law for the estate; estate skills live
in `../skills/`.

## Rules

Vendored content MUST live under this tree (today: `skills/`). Estate-owned
skills MUST NEVER be placed here.

Rationale: One greppable root separates upstream truth from estate prose so
reviews, upgrades, and “do not edit” policy have a hard boundary.

Examples:

- Good: `vendor/skills/emilkowalski/animate/`
- Bad: `vendor/skills/` holding an estate-only honesty-pass fork as if it were
  upstream
- Bad: estate skill at `skills/vendor/...` instead of under `vendor/`

Agents MUST NEVER edit vendored package bodies in place. Overlays and estate
policy MUST live under `skills/` or `.agents/references/`.

Rationale: An in-place edit forks the pin; the owner README and
`metadata.github-*` would lie about the bytes on disk.

Examples:

- Good: leave `vendor/skills/shadcn/shadcn/SKILL.md` untouched; add estate
  guidance as `skills/…` or a reference doc
- Bad: “small fix” inside `vendor/skills/pbakaus/impeccable/SKILL.md`

Each structural child this repo owns under `vendor/` MUST have its own
`README.md` (see root `AGENTS.md` §3). Upstream skill package directories MUST
NOT receive estate-authored READMEs.

Rationale: Parent folders explain layout; owner folders explain pins; package
interiors stay byte-identical to upstream.

Examples:

- Good: `vendor/skills/README.md`, `vendor/skills/emilkowalski/README.md`
- Bad: `vendor/skills/emilkowalski/animate/README.md` authored by us
