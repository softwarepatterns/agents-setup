# skills/typescript/

Estate-owned skills for **TypeScript** (and Bun/TS) codebases.

## Rules

Skills under this topic MUST apply to TypeScript contracts and examples. Agents
MUST NEVER place Rust- or Swift-only ports here.

Rationale: Topic folders keep language-specific examples honest; the honesty
pass concepts are shared, but fences and idioms are not interchangeable.

Examples:

- Good: `skills/typescript/honesty-pass/SKILL.md` with ` ```ts ` examples
- Bad: copying the Rust honesty-pass body here without a real TS port

Leaf skill directory names under this topic MUST remain unique across the whole
catalog (`skills/` + `vendor/skills/`).

Rationale: OpenCode skill ids are leaf directory names; rust and typescript
both use leaf `honesty-pass` under different parents — install profiles must
not flatten both into one id namespace without a deliberate choice.

Examples:

- Good: `skills/typescript/honesty-pass/` alongside `skills/rust/honesty-pass/`
- Bad: a third cosmetic fork (e.g. quote-only drift) as a separate skill
