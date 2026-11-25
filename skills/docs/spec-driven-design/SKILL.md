---
name: spec-driven-design

description: Create, maintain, review, and implement software specifications. Use when defining desired behavior, making design decisions, or changing software governed by specifications.

---

# Spec-Driven Design

## Purpose

Specifications are authoritative desired behavior, independent of implementation. They preserve decisions and rationale so agents stay consistent. Implementation follows the specification; implementation behavior does not silently redefine it.

## Repository conventions

All **specifications** MUST be stored under `./specs` in a product's root directory.

The **product root directory** is the root of the product the specifications describe. In a single-product repository, product root and repository root are the same, but in a monorepo each product will have its own product root.

Organize `/specs` into subfolders that group related material. Do not group by type of file, only group by related content. Use pillar pages to help others navigate particular topics more effectively.

Keep progress, implementation history, and current-state reporting outside desired-state specifications.

## Specification hierarchy

Specifications form a hierarchy of authority and scope via folders. Higher levels establish direction for lower levels. Lower levels apply that direction to increasingly specific parts of the system.

Specification pages will have one of the following goals:

- **Principle pages** establish product-wide direction.
- **Pillar pages** describe a general product area, making navigation of the specifications easier.
- **Capability pages** define a cross-product behavior across features.
- **Feature pages** define specific product features and outcomes.

### Principle pages

Principle pages establish product-wide direction, and include fundamental values, architectural philosophy, design positions, and non-negotiable constraints.

- Privacy and data ownership.
- Security philosophy.
- Compatibility policy.
- Promoted or banned software patterns.
- Technology constraints.
- User experience approach.

Principles establish defaults for the entire product. Lower-level specifications MUST follow these decisions unless an explicit exception is justified and accepted.

IMPORTANT: All rules must be falsifiable, capable of being validated. The validation and violations of named principles must be unambiguous.

### Pillar pages

Pillars describe a general product area, making navigation of the specifications easier. They may provide historical context, explain how principles apply within its area, or point to notable specific features or capabilities.

A pillar page must not introduce new principles, new features or new capabilities on its own, but may reference relevant pages that do.

### Capabilities

Capabilities describe reusable system behavior, contracts, or facilities that may be used by multiple features or pillars:

- Logging
- Authorization and authentication sessions.
- General file storage.
- Caching.
- Notification delivery.
- Event processing.

### Features

Features describe specific user-facing or system-facing behavior:

- Sign in with a passkey.
- Export an account.
- Share a document.
- Configure a notification.
- Import a data set.

Feature specifications define concrete behavior and outcomes—the most specific level of the hierarchy. Features MUST follow applicable principles and capability contracts.

### Inheritance and exceptions

Higher-level specifications establish the default direction for lower-level specifications.

1. A higher-level decision applies to all relevant lower-level specifications.
2. Lower-level specifications MUST NOT silently contradict higher-level decisions.
3. A lower-level specification may define a justified exception.
4. An exception MUST identify the higher-level decision being excepted and explain why the exception is necessary.
5. An exception requires explicit acceptance. An implementation agent MUST NOT create an exception independently.
6. An accepted exception applies only within its stated scope. An exception does not silently change the higher-level decision.

Important constraints should remain recognizable in lower-level prose without repeating every higher-level rule.

Example: user-data-under-user-control (principle) → sharing capability → share-document feature; any exception MUST be explicit and justified.

## Writing style

A specification is written for probabilistic agents who may read it once, partially, and under pressure.

**Be specific** The verdict is always pass or fail without deliberation. Vague principles ("handle errors gracefully", "keep it simple") license everything and detect nothing so deviance becomes indistinguishable from compliance. Good: "Clients may only branch on `code`; providing any other variance in error responses is illegal," with the exact violating string (`"detail": "pq: duplicate key value violates unique constraint"`) beside the compliant one. Bad: "errors should be consistent and informative."

**Issue verdicts** Write in the imperative present, never self-argued. A verdict followed by its defense reads as if the verdict needed defending, and what needs defending sounds negotiable. "Was rejected" reports a debate; "is banned" is law. Keep the rationale in its own sentence for the reader applying the rule's spirit; never fuse verdict and defense into one sentence. Good: "Occurrence-varying `detail` is banned: prose must not vary." Bad: "Occurrence-varying `detail` was rejected because prose that varies gets parsed and the scrutiny burden is lighter when nothing varies."

**Never ban without the replacement.** Agents never do well with a lack; a ban without a replacement leaves a vacuum the next agent fills with improvisation. The banned `detail` override was re-added repeatedly until the replacements were named. "Never do this" is half a law; "instead, do this" completes it. Good: "Instead of injecting the value into `detail`, add a catalog code; instead of diagnostics in the response, log them server-side." Bad: "Never inject dynamic content into error responses."

**Hedging is poison** Every hedge ("where possible", "generally", "preferably", "where the toolchain allows") is a hole in the rule, and an agent under pressure finds it and drives the feature through. A genuine exception is stated explicitly, scoped, and accepted (see Inheritance and exceptions) — never smuggled in as a modal verb. Good: "Catalog literals are the only source, enforced with compile-time-constant types." Bad: "…enforced with compile-time-constant types where the toolchain allows."

**Never list as open a question the document has answered.** An open question re-litigates the rule inside its own document. A violation is not a question; record it as a violation. A gap in enforcement is not a question; record it as a gap. Only genuinely undecided product decisions belong in Open Questions.

### Choose the purpose

Organize specifications around how the information will be used. A document may serve several purposes. Do not create categories merely to classify documents. Purpose and hierarchy level are separate decisions.

### Describe desired state

Write what the system should do, not what the implementation currently does.

Do not include:

- Implementation progress or completion status.
- Test results.
- Commit or pull request history.
- Current implementation defects.
- Incidental implementation details.

Intentional technical constraints are normative (required protocol, database capability, runtime, public interface). Incidental layout and variable names are not.

### Preserve reasoning

Important decisions include rationale: why the decision exists, what problem it addresses, when it applies. Use recognizable concepts so agents identify the decision at temptation. Do not restate the rule as its rationale. Do not invent artificial keyword lists or require a formal rule/rationale/example tuple—dense recurrence across prose is enough.

Definitions, background, diagrams, and straightforward requirements can stand without rationale. Laws and contested decisions cannot.

Example: authentication logic must remain independent of the HTTP framework. Authentication is shared by HTTP handlers, background workers, and command-line tools; a framework dependency would prevent those environments from using the same authentication logic. A new authentication feature must not introduce a dependency on the HTTP handler framework into the shared authentication crate.

### Write for retrieval

This skill wants retrieval ammo. Agents are probabilistic: a rule survives only if recognized at the moment of temptation.

**Compress.** Short documents get read; long ones get sampled. A sentence that neither states, justifies, nor demonstrates a rule is deleted. Make three sentences into one. Rule, rationale, and examples are not debate—they are keyword coverage; we're making the medicine sweet.

**Dense keywords.** The rule's vocabulary recurs across rule, rationale, and examples — including the words an agent holds at the moment of temptation: the library name, the flag, the synonym. "Detail override" catches the agent typing `detail`; "injection" catches the one constructing a string. A rule stated once, under one name, is found by no one.

**Name the outcomes.** The failure mode's name is the keyword an agent reaches for when the failure recurs.

### Write clear requirements

Use direct, normative language:

- MUST / MUST NOT for requirements and prohibitions.
- SHOULD for recommendations where justified exceptions exist.
- MAY for permitted choices.

EARS is useful for conditionals:

- WHEN an event occurs, the system SHALL respond in a defined way.
- WHILE a condition holds, the system SHALL maintain a defined behavior.
- IF an unwanted condition occurs, the system SHALL handle it in a defined way.

Use structured language where it improves precision. Do not force every sentence into EARS. Requirements MUST be clear, necessary, consistent, and verifiable—not vague ("handle errors gracefully", "respond quickly") unless behavior or measurement is defined. Keep requirements focused; do not fragment unnecessarily.

**Named, not numbered.** Requirements are named, not numbered, so citations stay stable: a number is an unstable identifier—one insertion renumbers every requirement after it, and every comment, test, or review that cited the old number points at the wrong rule or at nothing. A name survives reordering. Good — cite "Adapters live in the port" or "the batch law"; bad — cite "R7", which becomes "R8" after one edit and erodes trust in every document that referenced it.

### Use examples carefully

Examples distinguish intended behavior from plausible alternatives. They illustrate requirements; they do not create additional requirements unless stated. Reinforce the rule's concepts. Do not use violating examples as if compliant. Do not require an example on every rule.

### Include relevant boundaries

Use non-goals, assumptions, constraints, and failure behavior when they clarify the intended system:

- Record material assumptions rather than presenting assumptions as accepted decisions.
- Identify unresolved decisions instead of allowing implementation agents to invent answers.
- Describe important failure and unwanted behavior, not only successful operation.
- Include non-goals when likely interpretations could expand scope.

**Non-goals** must not name other specification files, paths, or folders; a non-goal uses only ideas that belong to this specification's subject. Naming other files turns the section into an index and defines this page by what lives elsewhere; a refusal stated as a property of this subject stays meaningful on its own. Good: "This surface does not host multi-panel docking workspaces." Bad: "Does not cover projects/ or the-shell/." Bad: "See projects.md for workshops."

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

**Design** vs **Requirements**: Design is the accepted mental model; Requirements are the normative MUST / MUST NOT / SHOULD / MAY. Do not bury obligations in design prose.

A section may be concise. Do not add empty boilerplate. Required headings give agents predictable locations and enable structural validation.

### Order content by confidence

Hard, accepted decisions MUST appear before guidance, suggestions, and speculation. The required structure encodes the order: **Design** and **Requirements** carry accepted direction, **Considerations** carries consequences, **Alternatives and Trade-offs** carries the rejected, **Open Questions** carries the unresolved. Do not present speculation as accepted design. Do not place unresolved proposals among mandatory requirements.

When a section mixes established and uncertain material, subhead: accepted first.

### Move material when confidence changes

Ordering is active maintenance. Promote accepted decisions; demote disputed or invalidated material; park new speculation below established direction. Never leave chronological order because text was added later: the strongest direction is the first direction an agent encounters.

## Document organization

### Keep documents understandable

A specification communicates its own purpose, decisions, and rationale without forcing an agent through an extensive document network. Explain important relationships locally. References may navigate; they MUST NOT be required to understand central rules. Avoid references that depend on document numbers, exact titles, or fixed directory positions.

### Avoid competing authority

Give each decision one authoritative home. Other specifications may explain use or interaction; do not copy the original decision. A feature may state that an operation requires an authenticated account; the authentication capability defines the contract. Lower levels may apply higher-level direction without reproducing it entire.

### Check consistency

Review for hierarchy conflicts, unjustified exceptions, weakened principles, and capability drift. Individually valid requirements can still form an invalid system—resolve contradictions and gaps before implementation. Avoid registries, dependency graphs, or cross-reference systems unless the repository has a practical need.

## Editing specifications

### Make focused changes

Prefer surgical edits. Preserve unrelated decisions and rationale. Restructure when organization, purpose, or meaning is wrong—do not preserve obsolete or contradictory text merely to minimize the diff.

Never rewrite a specification wholesale to make a targeted change. A full-file rewrite paraphrases every sentence it does not deliberately touch, and paraphrase silently drops content—rules, qualifiers, and rationale that were load-bearing. Default to edits; rewrite only with a diff review that proves no content was lost.

When editing, check hierarchy level, required headings, confidence ordering, higher-level relationships, and accepted exceptions.

### Do not turn specifications into history

Do not add implementation notes, completion reports, test results, change logs, or development history. Use version control and project-management documents for those.

### Do not silently adopt implementation behavior

When implementation conflicts with a specification, do not change the specification merely to match the code. Decide whether the implementation is wrong or the desired behavior changed. A specification changes only on an explicit product or design decision; record the reasoning.

### Commit specification changes separately

Specification changes MUST be committed separately from implementation changes. A commit that changes files under `./specs` MUST NOT also contain code implementation changes. An implementation commit MUST NOT contain specification changes.

If implementation reveals a missing or incorrect decision, stop at that boundary. Obtain approval, commit the specification change, then continue implementation in a separate commit. Do not combine specs and code merely to keep a feature in one commit—history must distinguish intended behavior from its implementation.

## Implementing from specifications

Read the relevant specification in full, plus applicable higher-level direction and accepted exceptions. Implement specified behavior; do not invent product behavior, weaken requirements, or treat existing code as proof of intent. Stop when a material decision is missing or ambiguous—no unilateral exceptions. Missing decision: propose, obtain approval, commit the specification change, then implement in a separate commit. Specifications govern intended behavior, contracts, and meaningful constraints—not every line of code.

## Verification and review

**Verification** asks whether implementation satisfies the specification. **Validation** asks whether the specification describes the intended system.

Evidence: tests, static analysis, benchmarks, inspection, formal analysis where justified. Keep useful links from requirements to evidence; no mandated identifier scheme.

Review for: ambiguous or unverifiable requirements; missing failure behavior; contradictions and incomplete decisions; unsupported assumptions; lost rationale; accidental implementation details; competing authority; wrong hierarchy placement; lower-level conflict with higher-level direction; unjustified exceptions; speculation above established decisions; implementation that silently redefines desired behavior. Ready to guide implementation beats well-formatted.

## Enforcement

Skills provide guidance, not deterministic enforcement. Do not claim a skill can refuse or prevent an action unless the execution environment actually enforces that restriction. Automated gates for reliable checks; human or agent review for semantic checks.

| Check | Passing outcome |
|---|---|
| Specification location | Every specification file is under `./specs` for the relevant product. |
| Document structure | Every specification contains the required headings in the required order. Additional headings are allowed. |
| Document formatting | Specification documents follow the repository's formatting rules. |
| Local references | Local references resolve when references are used. |
| Specification commit | A commit changing `./specs` contains no code implementation changes. |
| Implementation commit | A commit changing implementation files contains no specification changes. |
| Automated verification | Required automated tests and static analysis pass. |

Semantic review (not keyword counts): correct hierarchy level, higher-level direction followed, exceptions justified and accepted, requirements verifiable, rationale real, concepts recognizable, speculation separated, specification describes the intended system.

A reviewer detects problems; a CI gate can block a change; permissions can prevent unauthorized edits. These are different levels of control.

## Agent responsibilities

- Read relevant specifications before changing governed behavior.
- Preserve rationale; keep concepts recognizable; desired state only—no progress or incidental detail in specs.
- Apply higher-level direction; justify exceptions explicitly; never invent unspecified product behavior.
- Move speculation below established direction.
- Ask when decisions are missing; update specs only on accepted decision changes; commit specs separately from implementation.

Reviewers evaluate specification and implementation together. Passing tests are not proof of correctness.

## Reference methods

Useful when needed: EARS, INCOSE guidance, ISO/IEC/IEEE 29148, Specification by Example. They inform the work; they do not mandate taxonomy, template, or repository structure.
