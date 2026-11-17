# skills/

**Estate-owned** agent skills — authored and versioned by this catalog, not
upstream mirrors.

## Rules

Estate skills MUST live under this tree. Vendored skills MUST NEVER be placed
here; they belong under `../vendor/skills/<github-owner>/`.

Rationale: `skills/` is estate prose and policy. Mixing vendor bytes here
destroys the greppable boundary and invites in-place “fixes” of third-party
content.

Examples:

- Good: `skills/release/SKILL.md`, `skills/rust/honesty-pass/SKILL.md`
- Bad: copying `shadcn` into `skills/shadcn/` while a vendor copy exists
- Bad: `skills/vendor/...`

Optional topic directories (e.g. `rust/`) MUST group related estate skills
without becoming a second vendor taxonomy. Leaf skill directory names MUST
remain unique across `skills/` and `vendor/skills/` combined.

Rationale: Topics help humans browse estate law; OpenCode skill ids are leaf
directory names, so collisions break install.

Examples:

- Good: `skills/rust/honesty-pass/` (id `honesty-pass`)
- Bad: two leaves both named `release` under different parents

Structural folders this repo owns under `skills/` (including this directory and
any topic directory that holds multiple skills) MUST carry a short `README.md`
with rule tuples when the folder is created or repurposed. Individual skill
packages are documented by their `SKILL.md`, not a mandatory extra estate
README.

Rationale: Same owned-folder convention as `vendor/` (root `AGENTS.md` §3);
skill packages already have `SKILL.md` as their unit of documentation.

Examples:

- Good: `skills/README.md`; topic README when `skills/rust/` is a real grouping
- Bad: empty topic dir with no README and no skills
