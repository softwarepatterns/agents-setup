# honesty-pass (Rust)

Estate skill: hard gate on dishonest function/type contracts in **Rust**.

## Provenance

| Field | Value |
| --- | --- |
| Body | Identical across rust estate copies (stream-pro2, services-rs, clis-rs, …) |
| Catalog source | `ironspecs/stream-pro2/.agents/skills/honesty-pass/SKILL.md` |
| Language | Rust examples (` ```rust `) |

## Rules

This package MUST keep the upstream-of-catalog estate body until an explicit
content task changes it. Agents MUST NEVER “clean up” typos or rewrite examples
while only collecting or moving the skill.

Rationale: Extraction preserves real content; drift cleanup is a dedicated pass.

Examples:

- Good: byte-stable `SKILL.md` matching stream-pro2
- Bad: fixing “responsitory” mid-move

The TypeScript port MUST live at `skills/typescript/honesty-pass/`, not here.

Rationale: Same concepts, different idioms; one leaf per language topic.
