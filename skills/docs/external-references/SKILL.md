---
name: external-references
description: >
  Maintain repository-local external references in .agents/references/.
  Trigger when creating, editing, renaming, or removing reference files, or
  changing their entries in AGENTS.md.
---

# External References

## Purpose and Motivation

Maintain a repository-local collection of external discoveries that can be reused by future agents.

External references preserve information that required research outside the repository: documentation, standards, tool usage, working examples, dependency behavior, bugs, limitations, and discrepancies between documented and observed behavior.

The collection exists to prevent agents from repeatedly investigating the same external information. References should preserve the specific findings, evidence, conditions, and working examples that made the original research valuable.

The reference collection is not general project documentation, a second specification hierarchy, or a repository knowledge base.

The skill is responsible for maintaining references, not consuming them. Trigger this skill when creating, modifying, renaming, or removing files in `.agents/references/`, or when modifying the corresponding entries in `AGENTS.md`.

Do not trigger this skill merely because an agent reads, consults, or uses an existing reference.

## Scope

### Included information

A reference may preserve:

- External documentation for tools, libraries, frameworks, services, and dependencies.
- External standards, protocols, specifications, and technical requirements.
- Working CLI commands and configuration examples discovered through research.
- Working code patterns using external APIs and libraries.
- External bugs, limitations, and compatibility problems.
- Differences between official documentation and actual behavior.
- Maintainer explanations, issue discussions, release notes, and other authoritative external findings.
- Reproducible observations about external software.
- Workarounds and implementation constraints arising from external behavior.

The defining characteristic is external provenance: the information originates outside the repository or was discovered by investigating an external system.

### Excluded information

Do not use references to document:

- Internal project architecture or implementation details discovered solely by inspecting the repository.
- Product requirements, principles, pillars, capabilities, or features.
- Decisions about intended product behavior.
- Internal coding conventions or repository-specific rules.
- Issue descriptions, implementation plans, or PR histories.
- General explanations that duplicate existing project documentation.
- Information that belongs in the product specifications.
- Specific local constants, environment values, or current project configuration values that may change independently of the external finding.

Do not record values merely because they appeared in a working local command or example. A reference must not preserve transient local configuration as if it were durable external knowledge. Use placeholders or describe the configuration requirement generically when an example needs to show where a value belongs.

The defining test is whether the information remains useful if the repository's local configuration changes.

An external discovery may inform a specification or implementation decision. The reference preserves the external evidence; the appropriate project artifact records the resulting decision.

References do not establish product requirements or authorize changes to specifications.

## Design

### 1. Reference identity and metadata

Every Markdown reference file must begin with YAML front matter containing `name` and `description`.

Example:

```yaml
---
name: Tokio Runtime Patterns
description: >
  Tokio runtime configuration, task scheduling, blocking operations,
  and practical examples for avoiding runtime starvation.
---
```

The front matter is part of the reference's discovery mechanism.

**Name**

The name identifies the subject of the reference. Use a concise, recognizable name that distinguishes the reference from other entries.

**Description**

The description serves two purposes:

1. It provides vocabulary that can probabilistically trigger retrieval when an agent encounters a related problem.
2. It promises the useful information contained in the reference.

Descriptions should include relevant technical terms, tool names, concepts, and problem areas. Prefer specific vocabulary over generic summaries.

For example:

```yaml
description: >
  Tokio runtime configuration, task scheduling, blocking operations,
  spawn_blocking, runtime starvation, and working examples for
  asynchronous Rust services.
```

Avoid descriptions such as:

```yaml
description: Information about Tokio.
```

Do not add keywords unrelated to the actual contents merely to increase retrieval frequency.

The description must accurately represent the reference. Do not promise examples, findings, or coverage that the file does not contain.

### 2. File naming and organization

Store references under:

```text
.agents/
  references/
    reference-name.md
```

Use descriptive, stable, lowercase kebab-case filenames.

The filename, front matter name, and subject matter should be consistent. Avoid renaming an existing reference merely to improve its wording.

Before creating a file, inspect the existing references. Extend an existing reference when the new discovery belongs to the same subject and can be added without making the reference confusing or excessively broad.

Create a separate reference when the discovery represents a distinct subject that agents should be able to discover independently.

Do not create multiple references that compete to describe the same subject without a clear reason.

### 3. AGENTS.md discovery entries

Every reference must have a corresponding entry in the repository's `AGENTS.md`.

The entry must contain:

- A link to the reference file.
- The exact `name` from the reference's YAML front matter.
- The exact `description` from the reference's YAML front matter.

Example:

```markdown
## External References

- [Tokio Runtime Patterns](.agents/references/tokio-runtime-patterns.md)
  — Tokio runtime configuration, task scheduling, blocking operations,
  and practical examples for avoiding runtime starvation.
```

The reference front matter is the authoritative source for the name and description. The AGENTS.md entry is a discovery index, not an independently maintained description.

Whenever a reference is created, modified in a way that changes its metadata, renamed, or removed, inspect and update the corresponding AGENTS.md entry.

Whenever an AGENTS.md reference entry is modified, verify the referenced file and its front matter.

Maintain exact consistency:

- Every reference has one corresponding discovery entry.
- Every discovery entry points to an existing reference.
- The entry name matches the front matter name.
- The entry description matches the front matter description.
- The link points to the correct relative path.

Do not add reference entries that point to files that do not exist.

Do not remove an entry merely because a reference has become obsolete. Preserve discoverability while the reference remains useful, and describe its obsolete status in the reference itself.

### 4. Preservation of existing content

**Existing reference content must never be rewritten.**

An agent may make surgical edits, but must not regenerate, paraphrase, summarize, reorganize, or stylistically rewrite existing content.

This rule protects the specific information accumulated through external research.

Specifics are more valuable than prose quality. Preserve:

- Exact commands and arguments.
- Working code examples.
- Configuration requirements that are intrinsic to the external tool or API.
- Error messages.
- Version numbers.
- URLs and source references.
- Reproduction steps.
- Conditions and environment details.
- Important ordering and sequencing.
- Explanations of why a workaround is necessary.
- Details that may appear redundant but preserve independent evidence.

Do not replace a detailed example with a shorter example merely because the shorter version appears equivalent.

Do not consolidate multiple findings into a summary. Do not remove details to make a reference more concise. Do not change section organization as a stylistic improvement.

#### Append before rewriting

Prefer adding new findings in new sections or subsections.

When a new discovery extends an existing finding, add the new information alongside the original information rather than replacing the original text.

When a correction is necessary, change only the smallest relevant passage and preserve the original evidence wherever possible.

Do not use a correction as an opportunity to rewrite surrounding prose.

#### Findings later proven false

Preserve findings that are later proven false.

Do not silently delete or rewrite a previous finding because new evidence contradicts it.

Instead, append a correction or superseding finding that explains:

- Which earlier finding is affected.
- What new evidence was discovered.
- Under which versions or conditions the earlier finding applies.
- What behavior applies under the newly established conditions.
- Relevant external sources or reproduction evidence.

**Never use a heading to identify information as false, incorrect, erroneous, or obsolete.** Agents and humans may read only part of a reference, so a heading cannot reliably communicate that the information beneath it is no longer valid.

Every individual statement that presents a false or superseded finding must explicitly identify the information as false, incorrect, erroneous, or no longer applicable. A correction must not depend on a reader having seen another section or a heading.

For example, do not write:

```markdown
## Incorrect: Runtime Configuration

Tool X requires the `--legacy` option.
```

Instead, write:

```markdown
## Runtime Configuration

The earlier finding that Tool X requires the `--legacy` option is
incorrect for versions 4.0 and later. The requirement applies only
to versions before 4.0.

The original observation remains valid for versions before 4.0.
Versions 4.0 and later use the new configuration mechanism.

See:
- https://example.com/releases/4.0
- https://example.com/documentation/configuration
```

The original finding and supporting evidence must remain available. The correction must make the current applicability clear within the statements themselves.

#### Exceptional removal

Do not delete an existing reference or remove existing findings as routine maintenance.

Removal or redaction may be necessary for exceptional reasons, including:

- Exposed credentials, tokens, passwords, or other secrets.
- Material that should never have been included.
- Accidental duplication or unrelated content.
- Explicit authorization to remove the material.

When removal is necessary, preserve the useful historical context where possible without retaining the harmful content. Do not preserve secrets merely to satisfy the append-preservation rule.

If the reason for removal is not clear, stop and request clarification rather than silently deleting content.

### 5. Evidence and provenance

A reference must distinguish what an external source says from what an agent has independently observed or concluded.

**Maintain links to the sources of reference material.** Every externally sourced finding should retain a direct link to the source supporting the finding. Link to the specific documentation page, standard section, issue, discussion, release note, or source example whenever possible, rather than only linking to a general project homepage.

Preserve source links when editing or correcting findings. When new evidence changes an earlier conclusion, link the correction to the new evidence without removing the original source.

Useful sources include:

- Official documentation.
- Standards and protocol specifications.
- Maintainer discussions.
- Upstream issue trackers.
- Release notes and changelogs.
- Source code and examples maintained by the external project.
- Reproducible tests and experiments.

Do not present an agent's interpretation as an established external fact.

Distinguish at least these evidence types:

**Documented**

An external source explicitly describes the behavior.

**Observed**

An agent independently reproduced or tested the behavior. Record the relevant environment and conditions.

**Maintainer-confirmed**

An external maintainer or project representative confirmed the behavior or limitation.

**Inferred**

An agent reached a conclusion from available evidence but has not independently established the conclusion.

Use explicit wording when the distinction matters. Do not describe unverified behavior as tested or confirmed.

When practical, include the relevant source location, version, date, configuration requirements, or reproduction conditions.

Avoid copying large portions of external documentation. Preserve the specific information needed to understand and reuse the finding, with links to the original source.

### 6. Examples and practical findings

Prefer concise, working code and CLI examples over lengthy prose.

A useful reference should allow a future agent to apply a discovery without repeating the original research.

Examples should preserve all details needed for correct use, including:

- Required imports and dependencies.
- Relevant types and function signatures.
- Required configuration.
- Environment variables, when they describe external tool requirements rather than current local values.
- CLI flags and argument ordering.
- Version constraints.
- Required setup and execution order.
- Error handling when it affects the demonstrated behavior.

Do not simplify an example by removing details whose importance has not been established.

**Do not include specific local constants or current repository configuration values in examples.** Such values can become stale when local configuration changes. A stale value can erode trust in the surrounding reference and cause future agents to waste time verifying otherwise correct information.

Use descriptive placeholders for local values. Preserve exact values only when the values are intrinsic to the external tool, API, standard, or behavior being documented and are not merely copied from the current repository configuration.

Identify the verification status of examples when relevant:

- **Verified:** Tested against the identified version or environment.
- **Documented:** Provided by an external source but not independently tested.
- **Observed:** Demonstrates behavior reproduced by an agent.
- **Untested:** A plausible example that has not been verified.

Do not label an example verified unless the example was actually tested.

CLI examples should preserve exact working invocations. Code examples should favor complete, usable patterns over abstract pseudocode.

Prose should explain context, constraints, and consequences. Avoid lengthy introductory explanations that do not help an agent use the finding.

### 7. Confidence and authority ordering

Arrange material within every reference in descending order of confidence and authority.

Proven and actively used information belongs near the beginning of the document. Speculative, inferred, uncertain, or unknown information belongs lower in the document.

Use the following considerations when ordering material:

- Information confirmed by authoritative external sources and verified through actual use has the highest confidence.
- Information documented by authoritative external sources but not independently verified follows.
- Reproduced observations should identify their conditions and evidence.
- Inferences and interpretations should be clearly distinguished from established facts.
- Speculation, unresolved questions, and unknowns belong near the end.

Confidence and authority are related but distinct. An official source may have high authority while an agent's testing establishes whether the documented behavior applies under the conditions relevant to the finding. Preserve both distinctions rather than treating either as sufficient on its own.

When new evidence establishes a finding with greater confidence, place the new finding appropriately without rewriting or silently removing the earlier material.

Do not use document order to imply that a lower-confidence statement is established fact. State uncertainty explicitly in the statement itself.

This ordering helps agents encounter the most reliable and useful information first while retaining less certain findings for investigation.

### 8. Versioning and obsolete findings

External information may become obsolete.

When a finding is version-dependent, identify the relevant version or range whenever possible.

When new evidence establishes that a previous finding no longer applies:

1. Preserve the original finding.
2. Add a correction or superseding finding.
3. Identify the affected versions or conditions.
4. Record the new evidence and link to its source.
5. Explain which behavior should be used under current conditions.

Do not assume that an older finding has become worthless. Older information may still matter to agents working with older versions or investigating historical behavior.

Do not use a generic `last_updated` field as a substitute for version and evidence context.

Never rely on a heading alone to indicate that a finding is obsolete or incorrect. Every statement describing superseded behavior must explicitly state its applicability or that it is no longer valid under the specified conditions.

### 9. Reference lifecycle

A reference may be created, extended, corrected, or marked obsolete.

**Creation**

Create a reference when a significant external discovery is likely to save future agents from repeating research.

**Extension**

Append a new finding when additional external information belongs to an existing subject.

**Correction**

Append a correction or make the smallest necessary surgical edit. Preserve the original finding and its supporting evidence.

**Obsolescence**

Mark a finding or reference as obsolete when new evidence establishes that the information no longer applies. Preserve the historical finding and explain its applicability in every affected statement.

**Deletion**

Do not routinely delete references. Remove a reference only when its continued existence is harmful or when removal is explicitly authorized. Remove its AGENTS.md entry at the same time.

## Requirements

When a reference file or its AGENTS.md entry is changed, perform the following actions.

### 1. Inspect existing material

- Read the affected reference in full before editing.
- Inspect its YAML front matter.
- Inspect the relevant AGENTS.md entry.
- Check for other references covering the same subject.
- Identify existing findings, examples, evidence, source links, and version conditions that must be preserved.

Do not make changes based only on a filename or a partial excerpt.

### 2. Establish whether the information belongs in references

Before creating or extending a reference, establish that the information is external.

Identify the external tool, library, framework, service, standard, or other source.

Determine whether the finding is reusable and sufficiently supported to preserve.

Exclude specific local constants and current repository configuration values. If an example requires local configuration, use placeholders or describe the required configuration generically.

If the information belongs in an internal specification, issue, or project document instead, do not put it in references.

If the finding is uncertain, preserve the uncertainty rather than presenting it as established fact.

### 3. Choose the correct reference

Search existing references before creating a new file.

If an existing reference covers the subject, append the new finding there unless the new subject has a meaningful independent discovery identity.

Do not create a new file merely because the existing reference already contains related material.

Do not merge unrelated references merely to reduce the number of files.

### 4. Preserve existing content

Before editing:

- Identify the exact passage or section requiring change.
- Determine whether an appended finding is sufficient.
- Preserve unrelated text byte-for-byte wherever practical.
- Preserve existing examples, source links, and reproduction details.
- Do not perform formatting or stylistic cleanup.

After editing, inspect the diff.

Reject changes that rewrite or reformat unrelated existing content.

If an agent cannot confidently make a surgical change, do not regenerate the document. Stop and request clarification.

### 5. Maintain metadata

Verify that every reference contains valid YAML front matter with non-empty `name` and `description` fields.

Ensure the name and description accurately describe the contents.

Descriptions must contain useful retrieval vocabulary and must not promise information absent from the reference.

Avoid unnecessary metadata changes. New findings do not automatically require a new description.

### 6. Synchronize AGENTS.md

Verify that the reference has exactly one corresponding AGENTS.md discovery entry.

Ensure the entry has the correct link, name, and description.

Update the entry when the reference is created, renamed, removed, or when its metadata changes.

Do not independently rewrite descriptions in AGENTS.md.

### 7. Verify the result

Before finishing:

- Check YAML front matter syntax.
- Check that all reference links point to the correct files.
- Check that all reference files have corresponding AGENTS.md entries.
- Check that names and descriptions match exactly.
- Check that no existing finding was silently removed.
- Check that corrections preserve earlier findings.
- Check that every externally sourced finding retains a link to its source.
- Check that claims distinguish documented behavior from observed behavior and interpretation.
- Check that version and environment conditions are included where relevant.
- Check that no specific local constants or current repository configuration values have been introduced.
- Check that every false or superseded statement explicitly identifies its invalidity or limited applicability, without relying on a heading.
- Check that material is ordered by descending confidence and authority.
- Check that examples are labeled honestly.
- Test new commands or code examples when practical.
- Inspect the complete diff for unintended rewriting.

Report any unverified examples, unresolved questions, or external sources that could not be accessed.

## Considerations

### Preserve specificity over consistency

References do not need identical section structures.

A CLI reference, a protocol reference, a dependency bug report, and a working code pattern have different information needs.

Do not impose a universal document template that encourages unnecessary prose or empty sections.

The consistent requirements are evidence, useful specifics, discoverability, preservation, and confidence ordering.

### Avoid reference proliferation

Not every search result deserves a reference.

A reference should preserve a meaningful discovery that is likely to be reused. Trivial facts that are easy to find in official documentation do not automatically justify a new file.

Prefer extending an existing reference when the new finding belongs to the same subject.

### Do not confuse research with project decisions

An external standard may describe a requirement that the project has not adopted.

A library may make an existing implementation difficult.

A tool limitation may require a change to an implementation plan.

References preserve the evidence. They do not make the product decision.

Follow the repository's specification and issue processes for changes to product direction or implementation work.

### Protect the reference collection from automated rewriting

Do not run formatting, documentation-generation, or cleanup operations over the reference directory if those operations rewrite existing content.

Do not replace files with generated summaries.

Do not perform broad search-and-replace operations without inspecting every affected passage.

Use narrowly scoped patches and inspect the resulting diff.

## Alternatives and Trade-offs

### Append-only findings versus rewriting current knowledge

Append-preserving references can contain historical findings and corrections. A reader may need to consider more than one finding to determine current applicability.

This is intentional.

Rewriting a reference into a clean description of current knowledge can destroy the original evidence, conditions, and details that explain how the conclusion was reached.

Preserving historical findings is more valuable than maintaining a continuously polished narrative.

### Flexible structure versus a universal template

A universal template might improve visual consistency but can add empty sections and unnecessary prose.

Allow references to use structures suited to their subject while enforcing the shared metadata and preservation requirements.

### Exact metadata duplication versus independent summaries

Keeping the AGENTS.md entry identical to the reference front matter duplicates text.

The duplication is intentional because AGENTS.md provides discovery without requiring agents to open every reference. The skill enforces consistency, making the duplication manageable.

## Open Questions

None are required to apply this skill. Repository-specific naming, grouping, and presentation conventions may be adopted as the reference collection grows, provided the requirements above remain intact.