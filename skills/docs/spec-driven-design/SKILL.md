---
name: spec-driven-design

description: Create, maintain, review, and implement software specifications. Use when defining desired behavior, making design decisions, or changing software governed by specifications.

---

# Spec-Driven Design

## Purpose

Specifications describe the intended system, independently of its implementation.

Specifications preserve decisions and the reasoning behind those decisions so that different agents, working at different times, can reach consistent conclusions.

The specification is authoritative for desired behavior. Implementation follows the specification; implementation behavior does not silently redefine it.

## 1. Repository conventions

All specifications MUST be stored under `/specs`.

Organize `/specs` into subfolders that group related material. Use the specification hierarchy described below to determine where documents belong.

These are repository conventions, not requirements of specification engineering. Follow existing repository organization when it differs, except that specifications MUST remain under `/specs`.

Keep progress, implementation history, and current-state reporting outside desired-state specifications.

## 2. Specification hierarchy

Specifications form a hierarchy of authority and scope. Higher levels establish direction for lower levels. Lower levels apply that direction to increasingly specific parts of the system.

Use four levels.

### 2.1 Principles and positions

Principles and positions establish product-wide direction.

Use these specifications for decisions that apply broadly across the product, including fundamental values, architectural philosophy, design positions, and non-negotiable constraints.

Examples:

- Privacy and data ownership.
- Security philosophy.
- Compatibility policy.
- Architectural principles.
- Product-wide technology constraints.
- General user experience positions.

Principles and positions establish defaults for the entire product. Lower-level specifications MUST follow these decisions unless an explicit exception is justified and accepted.

### 2.2 Pillars

Pillars establish direction for major product or system areas.

A pillar interprets product-wide principles within a particular domain and provides direction for the specifications beneath it.

Examples:

- Identity and accounts.
- Data management.
- Rendering.
- Synchronization.
- Billing.
- Developer experience.

A pillar may establish decisions that apply across multiple capabilities and features.

Pillars are not merely indexes or collections of links. A pillar MUST contain meaningful direction for its subject area.

### 2.3 Capabilities

Capabilities describe reusable system behavior, contracts, or facilities that may be used by multiple features or pillars.

Examples:

- Authentication sessions.
- File storage.
- Notification delivery.
- Authorization.
- Data synchronization.
- Event processing.

A capability may belong to one pillar or serve several pillars. Do not force capabilities into a strictly tree-shaped organization when the system has legitimate cross-cutting capabilities.

Capabilities MUST follow applicable principles and pillar direction.

### 2.4 Features

Features describe specific user-facing or system-facing behavior.

Examples:

- Sign in with a passkey.
- Export an account.
- Share a document.
- Configure a notification.
- Import a data set.

Feature specifications define concrete behavior and outcomes. Feature specifications are the most specific level of the hierarchy.

Features MUST follow applicable principles, pillar direction, and capability contracts.

### 2.5 Inheritance and exceptions

Higher-level specifications establish the default direction for lower-level specifications.

Apply the following rules:

1. A higher-level decision applies to all relevant lower-level specifications.
2. Lower-level specifications MUST NOT silently contradict higher-level decisions.
3. A lower-level specification may define a justified exception.
4. An exception MUST identify the higher-level decision being excepted and explain why the exception is necessary.
5. An exception requires explicit acceptance. An implementation agent MUST NOT create an exception independently.
6. An accepted exception applies only within its stated scope. An exception does not silently change the higher-level decision.

A lower-level specification does not need to repeat every applicable higher-level rule. However, important constraints should remain recognizable in the lower-level document's own prose.

For example:

A product-wide principle establishes that user data remains under user control. A data-management pillar defines ownership and access rules. A sharing capability defines reusable sharing behavior. A feature defines a particular sharing workflow.

The feature follows the established direction. If a feature requires an exception, the exception must be explicit and justified.

### 2.6 Choosing a level

Choose the highest level that accurately represents the scope of the decision.

Ask:

- Does the decision apply across the entire product? Use a principle or position.
- Does the decision establish direction for a major product area? Use a pillar.
- Does the decision describe reusable behavior or a shared contract? Use a capability.
- Does the decision describe a specific workflow or outcome? Use a feature.

Do not create documents solely to fill a level. A small product may have few pillars or capabilities. A capability may serve several pillars. The hierarchy should reflect actual decision scope, not an organizational ideal.

## 3. Writing specifications

### 3.1 Choose the purpose

Organize specifications around how the information will be used, rather than forcing every document into a rigid technical type.

Common purposes include:

- Product behavior: features, workflows, user-visible outcomes, and API behavior.
- System design: architecture, component responsibilities, dependencies, and boundaries.
- Contracts: interfaces, protocols, data formats, and compatibility.
- Constraints: security, performance, platform support, and architectural invariants.
- Changes: migrations, deprecations, compatibility transitions, and bug fixes.

A document may serve several purposes. Use the organization that makes the content easiest to understand and maintain. Do not create categories merely to classify documents.

The purpose of a document and its hierarchy level are separate decisions. For example, a security constraint may be a product-wide principle, a pillar decision, or a feature-specific requirement.

### 3.2 Describe desired state

Write what the system should do, not what the implementation currently does.

Do not include:

- Implementation progress or completion status.
- Test results.
- Commit or pull request history.
- Current implementation defects.
- Incidental implementation details.

Include technical constraints when those constraints are intentional decisions about the desired system. A required protocol, database capability, runtime, or public interface can be normative. A current module layout or variable name generally is not.

### 3.3 Preserve reasoning

Important decisions should include their rationale.

Explain why a decision exists, what problem the decision addresses, and which circumstances make the decision relevant. Use recognizable concepts and terminology throughout the prose so future agents can identify when the decision applies.

Distribute important concepts naturally across the rule, rationale, and examples. Do not rely on a single occurrence of a term to communicate an important constraint.

This is particularly important for agents with limited context and probabilistic instruction-following. Repeated, relevant concepts increase the likelihood that an agent recognizes and applies the intended decision.

Do not create artificial keyword lists or require a formal rule/rationale/example tuple.

Do not merely restate a rule as its rationale.

For example:

**Rule:** Authentication logic must remain independent of the HTTP framework.

**Rationale:** Authentication is shared by HTTP handlers, background workers, and command-line tools. A framework dependency would prevent those environments from using the same authentication logic.

**Example:** A new authentication feature must not introduce a dependency on the HTTP handler framework into the shared authentication crate.

Not every statement needs a rationale. Definitions, background, diagrams, and straightforward requirements can stand on their own.

### 3.4 Write clear requirements

Use direct, normative language for mandatory behavior:

- MUST / MUST NOT for requirements and prohibitions.
- SHOULD for recommendations where justified exceptions exist.
- MAY for permitted choices.

EARS is useful for conditional requirements:

- WHEN an event occurs, the system SHALL respond in a defined way.
- WHILE a condition holds, the system SHALL maintain a defined behavior.
- IF an unwanted condition occurs, the system SHALL handle it in a defined way.

Use structured language where it improves precision. Do not force every sentence into EARS.

Requirements should be clear, necessary, consistent, and verifiable. Avoid vague requirements such as "handle errors gracefully" or "respond quickly" unless the intended behavior or measurement is defined.

Keep individual requirements focused. Do not fragment requirements unnecessarily.

**Named, not numbered.** Requirements are named, not numbered, so citations stay stable as the document changes. Rationale: a number is an unstable identifier — one insertion renumbers every requirement after it, and every comment, test, or review that cited the old number now points at the wrong rule or at nothing. A name survives reordering. Examples: good — cite "Adapters live in the port" or "the batch law"; bad — cite "R7", which becomes "R8" after one edit and erodes trust in every document that referenced it.

### 3.5 Use examples carefully

Include examples when examples help distinguish intended behavior from plausible alternatives.

Examples illustrate requirements; examples do not automatically create additional requirements. Make that distinction clear when necessary.

Use concrete examples that reinforce the important concepts in the associated rule and rationale.

Do not require every rule to have an example.

### 3.6 Include relevant boundaries

Use non-goals, assumptions, constraints, and failure behavior when they clarify the intended system.

In particular:

- Record material assumptions rather than presenting assumptions as accepted decisions.
- Identify unresolved decisions instead of allowing implementation agents to invent answers.
- Describe important failure and unwanted behavior, not only successful operation.
- Include non-goals when likely interpretations could expand scope.

**Non-goals:** The Non-goals section must not name other specification files,
paths, or folders. It may only use ideas that belong to this specification's
subject.

Rationale: Non-goals bound this subject. Naming other files turns the section
into an index and defines this page by what lives elsewhere. A refusal stated
as a property of this subject stays meaningful on its own.

Examples:

- Good: "This surface does not host multi-panel docking workspaces."
- Bad: "Does not cover projects/ or the-shell/."
- Bad: "See projects.md for workshops."

Do not add empty sections or boilerplate simply to satisfy a template.

## 4. Document structure and ordering

### 4.1 Required headings

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

Additional headings and subheadings are permitted. Use additional headings when they make the document easier to understand, navigate, or maintain. Do not omit, rename, or reorder the required headings.

The headings have distinct purposes:

- **Summary** gives a concise statement of the specification's subject and intended outcome. A reader should understand the central direction without first reading the entire document.
- **Purpose and Motivation** explains why the specification exists, what problem or need it addresses, and why the subject matters.
- **Scope** establishes what the specification covers and, where useful, what it does not cover. Identify the relevant system boundaries and affected areas. Non-goals follow §3.6.
- **Design** explains the accepted conceptual model, structure, approach, and important interactions. Give readers the understanding needed to interpret the requirements. This section describes the intended design, not incidental implementation details.
- **Requirements** states the normative obligations. Use direct, precise, verifiable language. This section is the authoritative statement of required behavior and constraints.
- **Considerations** explains relevant consequences, constraints, failure conditions, compatibility concerns, operational concerns, and other factors needed to understand or evaluate the design.
- **Alternatives and Trade-offs** records meaningful alternatives and explains why the accepted direction was chosen. Include drawbacks and costs of the accepted approach where relevant.
- **Open Questions** records decisions that remain unresolved and must not be invented by an implementation agent.

The separation between **Design** and **Requirements** is intentional. Design explains the accepted approach and establishes a useful mental model. Requirements state what the system MUST, MUST NOT, SHOULD, or MAY do. Do not bury normative obligations in explanatory design prose.

The headings do not require every specification to contain the same amount of material. A section may be concise when the subject needs little explanation. Do not add empty boilerplate simply to satisfy the structure.

The required headings provide predictable locations for agents and make structural validation possible. Additional headings may organize complex subjects beneath the required structure.

### 4.2 Order content by confidence

Within a document, arrange material in descending order of confidence and authority.

Hard, accepted decisions MUST appear before guidance, suggestions, and speculative material.

Use this general order:

1. Accepted decisions and mandatory requirements.
2. Established design direction and supported recommendations.
3. Illustrative examples and possible approaches.
4. Speculative ideas, hypotheses, and exploratory discussion.
5. Unresolved questions.

Apply this ordering within the required document structure:

- Present the accepted design in **Design**.
- State the authoritative obligations in **Requirements**.
- Use **Considerations** to explain relevant consequences and constraints.
- Use **Alternatives and Trade-offs** for meaningful alternatives, rejected approaches, and their trade-offs.
- Keep unresolved decisions and speculative questions in **Open Questions**.

Do not present a speculative approach as accepted design. Do not place an unresolved proposal among mandatory requirements.

Where a section needs to discuss both established and uncertain material, use subheadings to distinguish the two. Put accepted decisions and established direction before exploratory discussion.

### 4.3 Move material when confidence changes

Ordering is an active maintenance responsibility.

When a speculative idea becomes an accepted decision, move the decision into the appropriate higher-confidence section and update the relevant design and requirements.

When a decision becomes uncertain, disputed, or invalidated, move the affected material out of the authoritative requirements and identify the unresolved decision.

When new speculative material is added, place it below established direction.

Do not leave material in chronological order merely because the material was added later.

The objective is to make the strongest and most authoritative direction the first direction an agent encounters.

## 5. Document organization

### 5.1 Keep documents understandable

A specification should communicate its own purpose, decisions, and rationale without requiring an agent to discover an extensive network of other documents.

Explain important relationships locally. Avoid references that depend on document numbers, exact titles, or fixed directory positions.

References are useful for navigation, but should not be required to understand the central rules.

### 5.2 Avoid competing authority

Give each decision one authoritative home.

Other specifications may explain how a capability is used or how components interact. Do not copy the original decision into multiple documents.

For example, a feature specification may state that an operation requires an authenticated account. The authentication capability defines the authentication contract.

A lower-level specification may explain the relevant higher-level direction without reproducing the entire higher-level specification.

### 5.3 Check consistency

Review specifications for conflicting decisions, assumptions, boundaries, and requirements.

Pay particular attention to the hierarchy:

- Does a lower-level specification follow applicable higher-level direction?
- Does a lower-level specification introduce an exception?
- Is the exception explicitly justified and accepted?
- Does a proposed lower-level decision accidentally weaken a product-wide principle?
- Are shared capabilities consistent with the pillars and features that use them?

Do not assume individually valid requirements form a valid system. Resolve contradictions and important gaps before implementation.

Avoid creating registries, dependency graphs, or cross-reference systems unless the repository has a practical need for them.

## 6. Editing specifications

### 6.1 Make focused changes

Prefer surgical edits. Preserve unrelated decisions and rationale.

Restructure or rewrite a document when necessary to correct its organization, purpose, or meaning. Do not preserve obsolete or contradictory text merely to minimize the diff.

Never rewrite a specification wholesale to make a targeted change. A full-file rewrite paraphrases every sentence it does not deliberately touch, and paraphrase silently drops content - rules, qualifiers, and rationale that were load-bearing. Default to edits; rewrite only with a diff review that proves no content was lost.

When editing, also check whether the change affects:

- The document's hierarchy level.
- Its required headings and ordering.
- The confidence ordering of its content.
- Its relationship to higher-level decisions.
- Any explicitly accepted exceptions.

### 6.2 Do not turn specifications into history

Do not add implementation notes, completion reports, test results, change logs, or development history.

Use version control and project-management documents for those purposes.

### 6.3 Do not silently adopt implementation behavior

When implementation behavior conflicts with a specification, do not change the specification merely to match the code.

Determine whether the implementation is incorrect or whether the desired behavior needs to change.

A specification may change when an explicit product or design decision establishes a different desired outcome. Record the reasoning for meaningful changes.

### 6.4 Commit specification changes separately

Specification changes MUST be committed separately from implementation changes.

A commit that changes files under `/specs` MUST NOT contain code or implementation changes.

An implementation commit MUST NOT contain specification changes.

For a change that requires both specification and implementation updates, use separate commits:

1. **Specification commit:** establishes or revises the accepted desired behavior.
2. **Implementation commit:** implements that behavior without modifying specifications.

If implementation work reveals a missing or incorrect decision, stop implementation at that decision boundary. Obtain approval, make and commit the specification change, then continue implementation in a separate commit.

Do not combine specification changes with code changes merely to keep a feature in one commit.

This separation makes the history auditable: reviewers can distinguish changes to intended behavior from changes that implement intended behavior.

## 7. Implementing from specifications

Before implementing a change:

1. Read the relevant specification in full.
2. Read relevant supporting material and dependencies.
3. Identify applicable decisions, requirements, and constraints.
4. Determine the specification's hierarchy level.
5. Identify applicable higher-level direction.
6. Check whether the requested behavior is sufficiently specified.
7. Identify any accepted exceptions that apply.

During implementation:

- Implement specified behavior rather than inventing product behavior.
- Do not weaken requirements to simplify implementation.
- Do not treat existing implementation behavior as proof of intended behavior.
- Keep incidental implementation details in code, tests, or implementation documentation.
- Stop and request clarification when a material product or design decision is missing or ambiguous.
- Do not create an exception to a higher-level decision without explicit acceptance.

If implementation work reveals a missing or incorrect decision, propose the decision and explain the consequences. Obtain approval before changing desired behavior. Update and commit the specification before implementation of the accepted decision, using separate commits.

Not every implementation detail needs a specification requirement. Specifications govern intended behavior, contracts, and meaningful constraints—not every line of code.

## 8. Verification and review

Verification asks whether implementation satisfies the specification. Validation asks whether the specification describes the intended system.

Use appropriate evidence:

- Tests for executable behavior.
- Static analysis for enforceable code constraints.
- Benchmarks for performance requirements.
- Inspection and review for architectural and semantic requirements.
- Formal analysis where justified.

Maintain useful connections between requirements and verification evidence. Do not require a particular identifier scheme or test naming convention.

Review specifications and implementations for:

- Ambiguous or unverifiable requirements.
- Missing failure behavior.
- Contradictions and incomplete decisions.
- Unsupported assumptions.
- Lost or unclear rationale.
- Accidental implementation details.
- Duplicated or competing authority.
- Incorrect hierarchy placement.
- Lower-level decisions that conflict with higher-level direction.
- Unjustified or unaccepted exceptions.
- Speculative material appearing above established decisions.
- Implementation changes that silently redefine desired behavior.

A specification review should establish whether the document is ready to guide implementation, not merely whether the document is well formatted.

## 9. Enforcement

Skills provide guidance, not deterministic enforcement. Do not claim a skill can refuse or prevent an action unless the execution environment actually enforces that restriction.

Use automated gates where checks are reliable. Use human or agent review for semantic checks that cannot be reliably automated.

Recommended repository checks:

| Check | Passing outcome |
|---|---|
| Specification location | Every specification file is under `/specs`. |
| Document structure | Every specification contains the required headings in the required order. Additional headings are allowed. |
| Document formatting | Specification documents follow the repository's formatting rules. |
| Local references | Local references resolve when references are used. |
| Specification commit | A commit changing `/specs` contains no code or implementation changes. |
| Implementation commit | A commit changing implementation files contains no specification changes. |
| Automated verification | Required automated tests and static analysis pass. |

The checks above provide concrete pass/fail outcomes for repository structure, document structure, commit contents, and automated verification.

Semantic review remains necessary for matters such as:

- Whether a document belongs at the correct hierarchy level.
- Whether the design follows higher-level direction.
- Whether an exception is necessary, justified, and accepted.
- Whether requirements are clear and verifiable.
- Whether rationale explains the decision rather than repeating it.
- Whether important concepts are recognizable throughout the prose.
- Whether speculative material is clearly separated from accepted direction.
- Whether the specification describes the intended system.

Do not substitute mechanical keyword counts for semantic review. A vocabulary list can help trigger review, but cannot establish whether a rule was followed.

A reviewer detects problems; a CI gate can block a change; permissions can prevent unauthorized edits. These are different levels of control.

## 10. Agent responsibilities

Agents must:

- Read relevant specifications before changing governed behavior.
- Preserve the reasoning behind important decisions.
- Recognize that specifications describe desired state.
- Apply higher-level direction to lower-level work.
- Identify and justify exceptions rather than silently introducing them.
- Keep important concepts recognizable throughout the prose.
- Move speculative material below established direction.
- Avoid inventing unspecified product behavior.
- Ask for clarification when important decisions are missing.
- Update specifications when an explicitly accepted decision changes desired behavior.
- Keep implementation progress and incidental details out of specifications.
- Commit specification changes separately from implementation changes.

Reviewers should evaluate the specification and implementation together, rather than treating passing tests as proof of correctness.

## 11. Reference methods

Use established requirements-engineering practices where useful:

- EARS for structured requirements.
- INCOSE guidance for writing requirements.
- ISO/IEC/IEEE 29148 for requirements engineering.
- Specification by Example for connecting requirements with concrete behavior.

These methods inform the work. They do not require a particular document taxonomy, template, or repository structure.

---

Specifications preserve desired behavior and the reasoning behind it. Principles establish product-wide direction. Pillars apply that direction to major areas. Capabilities define reusable behavior. Features define specific outcomes. Plans describe intended work. Status describes progress. Implementations follow specifications, and explicit decisions—not accidental implementation behavior—change them.