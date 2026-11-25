---
name: external-references
description: >
  Maintain repository-local external references in .agents/references/.
  Trigger when creating, editing, renaming, or removing reference files, or
  changing their entries in AGENTS.md. Not for merely reading a reference.
---

# External References

## Purpose

Maintain repository-local external discoveries so future agents never re-research the same external fact.

References preserve findings, evidence, conditions, working examples, and discrepancies between docs and observed behavior. They are not general project docs, not a second specification hierarchy, not a product knowledge base.

This skill maintains references. Trigger it when creating, modifying, renaming, or removing `.agents/references/` files, or when changing the matching `AGENTS.md` entries. Never trigger it merely because an agent reads an existing reference.

## Location and identity

Store references under:

```text
.agents/references/<reference-name>.md
```

Filenames: descriptive, stable, lowercase kebab-case. Filename, front matter `name`, and subject stay consistent. Never rename only to polish wording.

Every reference file begins with YAML front matter: non-empty `name` and `description`.

```yaml
---
name: Tokio Runtime Patterns
description: >
  Tokio runtime configuration, task scheduling, blocking operations,
  spawn_blocking, runtime starvation, and working examples for
  asynchronous Rust services.
---
```

**name** identifies the subject. **description** is retrieval ammo: technical terms, tool names, problem areas the agent holds at temptation—and an accurate promise of contents. Never pad keywords absent from the file. Never promise examples or coverage the body lacks. Bad: `description: Information about Tokio.`

Before creating a file, inspect existing references. Extend the same subject in place when it fits without becoming confusing or excessively broad. Create a separate file only when the discovery needs independent discovery identity. Never create competing references for the same subject without a clear reason. Never merge unrelated references merely to reduce file count.

## What belongs

**Include** (external provenance—originates outside the repo or from investigating an external system):

- External docs for tools, libraries, frameworks, services, dependencies
- Standards, protocols, specifications
- Working CLI commands and configuration examples from research
- Working code patterns against external APIs and libraries
- External bugs, limitations, compatibility problems
- Docs vs actual behavior
- Maintainer explanations, issues, release notes, authoritative external findings
- Reproducible observations about external software
- Workarounds and constraints forced by external behavior

**Never** put in references:

- Internal architecture or implementation found only by reading this repo
- Product requirements, principles, pillars, capabilities, features, intended behavior
- Internal coding conventions or repo-only rules
- Issues, implementation plans, PR history
- General prose that duplicates project docs or belongs in `./specs`
- Specific local constants, env values, or current project configuration that can change independently of the external finding

Never record a value merely because it appeared in a working local command. Use placeholders or describe the configuration requirement generically. Test: the finding stays useful if local configuration changes.

An external discovery may inform a spec or implementation decision. The reference holds the evidence; the project artifact holds the decision. References never establish product requirements or authorize specification changes.

## AGENTS.md discovery

Every reference has exactly one corresponding discovery entry in the repository `AGENTS.md`. Front matter is authoritative for `name` and `description`. The entry is an index, never an independent summary.

**Entry must include:** link to the file; exact front matter `name`; exact front matter `description` (or the same description text used as a trigger line that still matches front matter vocabulary exactly when the repo uses a trigger table).

Example (link list form):

```markdown
## External References

- [Tokio Runtime Patterns](.agents/references/tokio-runtime-patterns.md)
  — Tokio runtime configuration, task scheduling, blocking operations,
  spawn_blocking, runtime starvation, and working examples for
  asynchronous Rust services.
```

On create, metadata change, rename, or remove: update `AGENTS.md` in the same change. On any `AGENTS.md` reference-entry edit: verify the file and front matter.

Consistency law:

- One discovery entry per reference; every entry points at an existing file
- Entry name = front matter name; entry description = front matter description
- Link is the correct relative path
- Never add entries for missing files
- Never remove an entry merely because a reference feels obsolete—mark obsolete inside the reference; keep discoverability while the file remains useful
- Never independently rewrite descriptions in `AGENTS.md`

## Preserve existing content

**Existing reference body is never rewritten after initial write.**

Never regenerate, paraphrase, summarize, reorganize, or stylistically rewrite existing content. Only surgical edits. Specifics beat prose quality.

Preserve: exact commands and arguments; working code; configuration intrinsic to the external tool or API; error messages; version numbers; URLs and sources; reproduction steps; conditions and environment; ordering and sequencing; why a workaround is necessary; details that look redundant but are independent evidence.

Never replace a detailed example with a shorter “equivalent.” Never consolidate findings into a summary. Never remove detail to sound concise. Never restyle section organization.

### Append before replace

Append new findings in new sections or subsections. When a discovery extends an earlier finding, add alongside—never replace the original text. Correction changes only the smallest relevant passage; never rewrite surrounding prose under cover of a fix.

### Findings later proven false

Never silently delete or rewrite a finding because new evidence contradicts it. Append a correction or superseding finding that states:

- Which earlier finding is affected
- What new evidence appeared
- Versions or conditions where the earlier finding still applies
- Behavior under the new conditions
- Sources or reproduction evidence

**Never use a heading alone to mark information false, incorrect, erroneous, or obsolete.** Partial readers miss headings. Every statement that presents a false or superseded finding must itself say the information is false, incorrect, erroneous, or no longer applicable. A correction never depends on having read another section or a heading.

Bad:

```markdown
## Incorrect: Runtime Configuration

Tool X requires the `--legacy` option.
```

Good:

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

Original finding and evidence stay available. Current applicability is clear inside the statements.

### Exceptional removal

Never delete references or findings as routine maintenance or cleanup.

Remove or redact only for:

- Exposed credentials, tokens, passwords, or secrets
- Material that never belonged
- Accidental duplication or unrelated content
- Explicit authorization to remove

When removing, keep useful historical context without retaining harmful content. Never preserve secrets to satisfy append-preservation. If the removal reason is unclear, stop and ask—never silently delete.

## Evidence and sources

Distinguish what an external source says from what an agent observed or concluded. Never present interpretation as established external fact.

**Maintain source links.** Every externally sourced finding retains a direct link to the supporting source—specific docs page, standard section, issue, discussion, release note, or upstream example—not only a project homepage. Preserve links when editing. Corrections link new evidence without removing the original source.

Sources: official docs; standards; maintainer discussions; upstream trackers; release notes; upstream source and examples; reproducible experiments.

Evidence types—name them when the distinction matters:

- **Documented** — external source states it
- **Observed** — agent reproduced or tested it; record environment and conditions
- **Maintainer-confirmed** — maintainer or project representative confirmed it
- **Inferred** — conclusion from evidence, not independently established

Never describe unverified behavior as tested or confirmed. Record version, date, configuration requirements, and reproduction conditions on observed and version-bound findings.

Never copy large slabs of external documentation. Keep the specific reusable finding plus links.

## Examples and local values

Use concise working code and CLI over long prose. A future agent must apply the discovery without repeating the research.

Preserve in examples: required imports and dependencies; types and signatures; required configuration; env vars when they are external tool requirements (not current local values); CLI flags and argument order; version constraints; setup and execution order; error handling that affects the demonstrated behavior.

Never simplify by dropping details whose importance is unproven.

**Never include specific local constants or current repository configuration values in examples.** Stale values erode trust and waste verification time. Use descriptive placeholders. Exact values only when intrinsic to the external tool, API, standard, or behavior—not copied from this repo’s current config.

Label examples honestly when status matters:

- **Verified** — tested against the named version or environment
- **Documented** — from an external source, not independently tested
- **Observed** — behavior an agent reproduced
- **Untested** — plausible, not verified

Never label verified unless actually tested. CLI: exact working invocations. Code: complete usable patterns over abstract pseudocode. Prose explains constraints and consequences—never lengthy intros that do not help use the finding.

## Confidence order

Arrange every reference in descending confidence and authority. Proven, actively used material first; speculative, inferred, uncertain, unknown last.

- Highest: authoritative external source **and** verified in use
- Next: authoritative documentation, not independently verified
- Reproduced observations name conditions and evidence
- Inferences stay labeled inferences
- Speculation and open unknowns at the end

Authority ≠ verification. Official docs can be high authority while testing establishes whether they apply under these conditions. Preserve both. New higher-confidence findings go in the right place without rewriting or silently removing earlier material. Never let document order alone imply a low-confidence statement is fact—state uncertainty in the statement.

## Versions and obsolescence

Version-dependent findings name the version or range.

When new evidence shows a previous finding no longer applies:

1. Preserve the original finding
2. Append correction or superseding finding
3. Name affected versions or conditions
4. Record new evidence and link its source
5. State which behavior to use under current conditions

Older findings still matter for older versions and historical investigation. Never use a generic `last_updated` field instead of version and evidence context. Never rely on a heading alone for obsolete or incorrect—every superseded statement states applicability or invalidity itself.

## Lifecycle

- **Create** when a significant external discovery will save future agents from repeating research. Trivial facts already trivial in official docs do not automatically deserve a file.
- **Extend** by appending when new external information belongs on an existing subject.
- **Correct** with append or the smallest surgical edit; preserve original finding and evidence.
- **Obsolete** by marking in-statement applicability; preserve the historical finding.
- **Delete** only when continued existence is harmful or removal is explicitly authorized; remove the `AGENTS.md` entry in the same change.

## Agent checklist

When a reference file or its `AGENTS.md` entry changes:

1. Read the affected reference **in full**. Inspect front matter, `AGENTS.md` entry, sibling references on the same subject, and every finding/example/link/version condition that must be preserved. Never edit from filename or partial excerpt alone.
2. Confirm external provenance, reusable support, and no local constants or current repo config (placeholders if needed). If it belongs in specs, issues, or project docs—keep it out of references. Preserve uncertainty; never upgrade inference to fact.
3. Append on the existing subject file; new file only for independent discovery identity.
4. Surgical edit only: identify the exact passage; append when adding; preserve unrelated text; preserve examples, source links, reproduction detail; never formatting or style cleanup. Inspect the diff. Reject rewrites or reformats of unrelated content. If a surgical change is not possible, never regenerate the document—stop and ask.
5. Valid front matter `name` and `description`; accurate; retrieval vocabulary; no false promises. New findings do not automatically require a new description.
6. Exactly one matching `AGENTS.md` entry; link, name, description exact; update on create/rename/remove/metadata change.
7. Before finish: YAML syntax; links resolve; every reference file has an entry and every entry has a file; names/descriptions match; no silent removal of findings; corrections preserve earlier findings; external findings keep source links; documented vs observed vs inferred stay distinct; version/env where relevant; no local constants introduced; every false or superseded statement self-labels without relying on a heading; confidence order holds; example labels honest; never mark verified without a real test; complete diff shows no unintended rewrite. Report unverified examples, unresolved questions, and sources that could not be fetched.

## Protect the collection

Never run formatters, doc generators, or cleanup over `.agents/references/` if they rewrite bodies. Never replace files with generated summaries. Never broad search-and-replace without inspecting every affected passage. Narrow patches; inspect the diff.

References need no universal template. CLI notes, protocol notes, dependency bugs, and code patterns differ. Shared requirements only: evidence, useful specifics, discoverability, preservation, confidence order. Never empty boilerplate sections.

References preserve evidence. They never make the product decision—follow specification and issue process for product direction and implementation work.
