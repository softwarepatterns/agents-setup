# skills/rust/

Estate-owned skills for **Rust** codebases.

## Rules

Skills under this topic MUST apply to Rust (and Rust-shaped) contracts and
examples. Agents MUST NEVER place TypeScript- or Swift-only honesty ports here.

Rationale: Topic folders keep language-specific examples honest; mixing ports
in one leaf invites the wrong fence language at apply time.

Examples:

- Good: `skills/rust/honesty-pass/SKILL.md` with ` ```rust ` examples
- Bad: dropping the TypeScript honesty-pass body under `skills/rust/`

Leaf skill directory names under this topic MUST remain unique across the whole
catalog (`skills/` + `vendor/skills/`).

Rationale: OpenCode skill ids are leaf directory names.

Examples:

- Good: single `honesty-pass` leaf under `rust/` (TypeScript port lives under
  `skills/typescript/honesty-pass/`)
- Bad: a second unrelated skill also named `honesty-pass` under another rust
  subfolder
