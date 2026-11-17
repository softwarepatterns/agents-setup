# honesty-pass (TypeScript)

Estate skill: hard gate on dishonest function/type contracts in **TypeScript**.

## Provenance

| Field | Value |
| --- | --- |
| Body | `core-ts` / `services-ts` identical body (double-quoted examples) |
| Catalog source | `ironspecs/core-ts/.agents/skills/honesty-pass/SKILL.md` |
| Excluded fork | `react-ts` `.codex` copy — same content, single-quote-only drift |
| Language | TypeScript examples (` ```ts `) |

## Rules

This package MUST keep the core-ts/services-ts body until an explicit content
task changes it. Agents MUST NEVER substitute the react-ts quote-only fork or
rewrite examples while only collecting or moving the skill.

Rationale: Extraction preserves the useful shared TS body; cosmetic forks are
not a second skill.

Examples:

- Good: `SKILL.md` byte-identical to core-ts
- Bad: “normalizing” quotes to match react-ts during collect

The Rust port MUST live at `skills/rust/honesty-pass/`, not here.

Rationale: Same concepts, different idioms; one leaf per language topic.
