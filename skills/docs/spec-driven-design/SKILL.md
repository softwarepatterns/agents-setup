---
name: spec-driven-design

description: Create, maintain, review, and implement software specifications. Use when defining desired behavior, making design decisions, or changing software governed by specifications.

---

# Spec-Driven Design

## Purpose

Specifications are authoritative desired behavior, independent of implementation. They preserve decisions and rationale so agents stay consistent. Implementation follows the specification.

## Repository conventions

All **specifications** MUST be stored under `./specs` in a product's root directory.

The **product root directory** is the root of the product the specifications describe. In a single-product repository, product root and repository root are the same, but in a monorepo each product will have its own product root.

Organize `./specs` into subfolders that group related material. Do not group by type of file, only group by related content. Use pillar pages to help others navigate particular topics more effectively.

Keep progress, implementation history, and current-state reporting outside desired-state specifications.

## Specification hierarchy

Specifications form a hierarchy of authority and scope, and by directory structure. Higher levels establish direction for lower levels. Lower levels apply that direction to increasingly specific parts of the system.

Specification pages will be one of the following:

- **Principle pages** establish product-wide direction.
- **Pillar pages** describe a general product area, making navigation of the specifications easier.
- **Capability pages** define a cross-product behavior across features.
- **Feature pages** define specific product features and outcomes.

### Principle pages

Principle pages establish product-wide rules, such as fundamental values, architectural philosophy, design positions, and non-negotiable constraints. Lower-level specifications MUST follow these decisions unless an explicit exception is justified and accepted.

IMPORTANT: All rules must be falsifiable, capable of being validated. The validation and violations of named principles must be unambiguous. Any principles that are not falsifiable due to vague hand-waving must be deleted.

### Pillar pages

Pillars describe a general product area, making navigation of the specifications easier. They may provide historical context, explain how principles apply within its area, or point to notable specific features or capabilities.

A pillar page never introduces new principles, new features or new capabilities on its own, but may reference relevant pages that do.

### Capabilities

Capabilities describe reusable system behavior, contracts, or facilities that may be used by multiple features, such as logging, auth, general file storage, caching, notification delivery, or event processing.

### Features

Features describe specific user-facing or system-facing behavior. Features define concrete behavior and outcomes, and are the most specific level of the hierarchy. Features MUST follow applicable principles and capability contracts.

### Inheritance and exceptions

Higher-level specifications establish the default direction for lower-level specifications. A higher-level decision applies to all relevant lower-level specifications. Lower-level specifications never silently contradicts higher-level decisions. A lower-level specification may define a justified exception. An exception MUST identify the higher-level decision being excepted and explain why the exception is necessary. An exception requires explicit acceptance. An implementation agent never creates an exception independently. An accepted exception applies only within its stated scope. An exception does not silently change the higher-level decision.

## Writing style

A specification is written for probabilistic agents who may read it once, partially, and under pressure.

**Be specific** A rule must always pass or fail without deliberation. Vague principles ("handle errors gracefully", "keep it simple") must be deleted, or violations become indistinguishable from compliance. Good: "Clients may only branch on `code`; providing any other variance in error responses is illegal," with the exact violating string (`"detail": "pq: duplicate key value violates unique constraint"`) beside the compliant one. Bad: "errors should be consistent and informative."

**Issue verdicts** Write in the imperative present, never passive. A spec is not a negotiation, or a history.

**Never ban without the replacement.** Agents cannot handle a lack; a ban without a replacement leaves a gap the next agent fills randomly. "Never do this" is mistaken as permission to improvise, so always provide the recommended alternatives. Good: "Never inject stack traces into HTTP responses, instead add a catalog code for the situation." Good: "Never delete a user's data without showing a warning dialog." Bad: "Dynamic content in error responses is banned." because agents will try to negotiate, and a spec is not a negotiation.

**Hedging is poison** Every hedge ("where possible", "generally", "preferably", "where the toolchain allows") is a hole in the rule. An agent under pressure may erroneously interpret hedges as an emergency recommendation instead of danger. All hedging language must be immediately deleted. All genuine exceptions must be stated explicitly and scoped to avoid the danger. Bad: "All thrown errors must be caught unless needed." Good: "All thrown errors must be caught, except for specific listed errors caught by middleware."

**Never list as open a question the document has answered.** Don't relitigate the rule inside its own document with such duplication. A violation is not a question; record it as a violation. A gap in enforcement is not a question; record it as a gap. Only genuinely undecided product decisions belong in Open Questions.

### Describe desired state

Write what the system should do, not what the implementation currently does. Do not include:

- Implementation progress or completion status.
- Test results.
- Commit or pull request history.
- Current implementation defects.
- Incidental implementation details.

Intentional technical constraints are normative (required protocol, database capability, runtime, public interface). Incidental layout and variable names are not.

### Preserve reasoning

Rules without rationale are at risk, or are a risk to the product. Never delete the rationale or examples from a rule. Examples of rationale include why the decision was needed, when it applies, what it would look like if the rule didn't exist and what it looks like since it does. Use recognizable keywords and code samples so agents identify the decision at temptation. Do not restate the rule as its rationale. Do not invent artificial keyword lists or require a formal rule/rationale/example tuple or heading. Compress the rule via key words and phrases with dense recurrence across prose.

Example: authentication logic must remain independent of the HTTP framework. Authentication is shared by HTTP handlers, background workers, and command-line tools; a framework dependency would prevent those environments from using the same authentication logic. A new authentication feature never introduces a dependency on the HTTP handler framework into the shared authentication crate.

### Write for retrieval

Write for retrieval ammo. Agents are probabilistic: a rule survives only if recognized at the moment of temptation.

**Compress.** Short documents get read; long ones get sampled. Delete any sentence that neither states, justifies, nor demonstrates a rule. Present a rule with its rationale and examples to achieve keyword coverage, including matching code samples.

**Dense keywords.** The rule's vocabulary recurs across rule, rationale, and examples — including the words an agent holds at the moment of temptation: the library name, the flag, the synonym. "Detail override" catches the agent typing `detail`; "injection" catches the one constructing a string. A rule stated once, under one name, is found by no one.

All feature documents must be less than 500 lines. Do not delete keywords. Do not delete rules, rationale or examples. Compress until potential agent or human coherence risk or loss of information, then ask for permission to split the document into smaller features.

### Write clear requirements

Use direct, normative language:

- MUST for requirements.
- NEVER for prohibitions.
- SHOULD for recommendations that are not rules.
- MAY for permitted choices.

EARS is useful for conditionals:

- WHEN an event occurs, the system SHALL respond in a defined way.
- WHILE a condition holds, the system SHALL maintain a defined behavior.
- IF an unwanted condition occurs, the system SHALL handle it in a defined way.

Use structured language where it improves precision. Do not force every sentence into EARS. Requirements MUST be clear, necessary, consistent, and verifiable—not vague. BAD: "handle errors gracefully", BAD: "respond quickly". Keep requirements focused; do not fragment unnecessarily.

**Named, not numbered.** Requirements must be named, not numbered, so citations stay stable. Numbers are an unstable identifier—one insertion renumbers every requirement after it, and every comment, test, or review that cited the old number suddenly points at the wrong rule or at nothing. A name survives reordering. Good: cite "Adapters live in the port" or "the batch law"; bad — cite "R7", which becomes "R8" after one edit and erodes trust in every document that referenced it.

### Use examples carefully

Examples distinguish intended behavior from plausible alternatives. Examples reinforce the rule. They never create additional rules. Do not use violating examples. Do not require an example on every rule. Examples are an opportunity for additional triggering keywords.

### Include relevant boundaries

Use non-goals, assumptions, constraints, and failure behavior to clarify the intended feature. Record assumptions rather than presenting those assumptions as decisions. Identify unresolved decisions instead of allowing implementation agents to invent answers. Include non-goals when likely interpretations could expand scope.

**Non-goals** never name other specification files, paths, or folders; a non-goal uses only ideas that belong to this specification's subject. Bad: "Does not cover projects/ or the-shell/." Bad: "See projects.md for workshops." Good: "This surface does not host multi-panel docking workspaces." 

Do not add empty sections or boilerplate simply to satisfy a template.

## Document structure and ordering

### Required headings

Every specification MUST contain the following headings, in this order:

1. `# [Specification title]`
2. `## Summary`
3. `## Purpose and Motivation`
4. `## Scope`
5. `## Design`
6. `## Requirements`
7. `## Considerations`
8. `## Alternatives and Trade-offs`
9. `## Open Questions`

Do not omit, rename, or reorder required headings. Additional headings are permitted beneath this structure.

**Design** vs **Requirements**: Design is the accepted mental model; Requirements are the normative MUST / NEVER / SHOULD / MAY. Do not bury obligations in design prose.

A section may be concise. Do not add empty boilerplate. Required headings give agents predictable locations and enable structural validation.

### Order content by confidence

Hard, accepted decisions must appear before guidance, suggestions, and speculation. The required structure encodes the order: **Design** and **Requirements** carry accepted direction, **Considerations** carries consequences, **Alternatives and Trade-offs** carries the rejected, **Open Questions** carries the unresolved. Do not present speculation as accepted design. Do not place unresolved proposals among mandatory requirements.

When a section mixes established and uncertain material, subhead: accepted first.

### Move material when confidence changes

Ordering is active maintenance. Elevate accepted decisions; press disputed or invalidated material downward; park new speculation below established direction. Never leave chronological order because text was added later: the strongest direction is the first direction an agent encounters.

## Document organization

### Avoid competing authority

Give each decision one authoritative home. Other specifications may explain use or interaction; do not copy the original decision. A feature may state that an operation requires an authenticated account; the authentication capability defines the contract. Lower levels may apply higher-level direction without reproducing it entirely.

### Check consistency

Review for hierarchy conflicts, unjustified exceptions, weakened principles, and capability drift. Individually valid requirements can still form an invalid system—resolve contradictions and gaps before implementation. Avoid registries, dependency graphs, or cross-reference systems unless the repository has a practical need.

## Editing specifications

### Make focused changes

Only use surgical edits after initial file creation. Never rewrite an entire file after initial write. A full-file rewrite causes damaging paraphrases and loses content. Preserve all decisions and rationale, do not paraphrase when editing. Restructure when organization, purpose, or meaning is wrong—do not preserve obsolete or contradictory text merely to minimize the diff. Never save history of spec changes in the document.

### Do not turn specifications into history

Do not add implementation notes, completion reports, test results, change logs, or development history. Use version control and project-management documents for those.

### Do not silently adopt implementation behavior

When implementation conflicts with a specification, never change the specification merely to match the code. Decide whether the implementation is wrong or the desired behavior changed. A specification changes only on an explicit product or design decision; record the reasoning.

### Commit specification changes separately

Specification changes must be committed separately from implementation changes. A commit that changes files under `./specs` never also contains code implementation changes. An implementation commit never contains specification changes.

If implementation reveals a decision, stop at that boundary. Obtain approval, commit the specification change, then continue implementation in a separate commit. Do not combine specs and code merely to keep a feature in one commit—history must distinguish intended behavior from its implementation.

## Implementing from specifications

Read the relevant specification in full, plus applicable higher-level direction and accepted exceptions. Implement specified behavior; do not invent product behavior, weaken requirements, or treat existing code as proof of intent. Stop when a material decision is missing or ambiguous—no unilateral exceptions. Missing decision: propose, obtain approval, commit the specification change, then implement in a separate commit. Specifications govern intended behavior, contracts, and meaningful constraints—not every line of code.

## Verification and review

**Verification** asks whether implementation satisfies the specification. **Validation** asks whether the specification describes the intended system.

Evidence: tests, static analysis, benchmarks, inspection, formal analysis where justified. Keep useful links from requirements to evidence; no mandated identifier scheme.

Review for: ambiguous or unverifiable requirements; missing failure behavior; contradictions and incomplete decisions; unsupported assumptions; lost rationale; accidental implementation details; competing authority; wrong hierarchy placement; lower-level conflict with higher-level direction; unjustified exceptions; speculation above established decisions; implementation that silently redefines desired behavior. Ready to guide implementation beats well-formatted.

## Agent instructions

- Read relevant specifications before changing governed behavior.
- Preserve rationale; keep concepts recognizable; desired state only—no progress or incidental detail in specs.
- Apply higher-level direction; justify exceptions explicitly; never invent unspecified product behavior.
- Move speculation below established direction.
- Ask when decisions are missing; update specs only on accepted decision changes; commit specs separately from implementation.

Ensure AGENTS.md includes requirements at least as strong as the following, or propose to the user adding the requirements if missing:

"Agents require explicit user approval of exact proposed changes before any specification file is edited. Reviewers are required to verify specification changes comply with the spec-driven-design skill, that all changes are in the appropriate sections, and that paragraphs are coherent, compressed, and consistent with the rest of the document."

## Reference methods

Useful when needed: EARS, INCOSE guidance, ISO/IEC/IEEE 29148, Specification by Example. They inform the work; they do not mandate taxonomy, template, or repository structure.
