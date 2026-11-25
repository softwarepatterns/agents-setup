# Comments and body files

## PR comments are issue comments

```bash
forgejo repos issues comments create \
  --owner O --repo R --issue N --body-file reply.json

forgejo repos issues comments list --owner O --repo R --issue N
forgejo repos issues comments delete --owner O --repo R --issue N --comment <id>
```

`reply.json`:

```json
{"body": "Plain prose response to the review."}
```

`N` is the pull request number (PRs share the issues comment API).

## Body-file law

Prefer `--body-file <path>` over inline flags for any non-trivial JSON.
Write the file, pass the path, delete the temp file after success if it
contained nothing secret (PR bodies are not secrets; tokens never go in body
files).

## Body scanner gotcha

Some comment bodies are rejected with:

```json
{"type":"unknown","Portable shell scanner cannot analyze command: invalid-structure"}
```

Trigger: markdown that leans on **inline code backticks** and similar structures
the portable shell scanner misreads.

**Replacement:** plain prose, numbered lists, minimal code spans. Put long code
in the PR diff or a gist-like follow-up, not in the comment body. *verified*
against the estate instance with forgejo-cli 0.3.0.

WHEN a create comment fails with that payload, MUST rewrite the body without
heavy backticks and retry — do not switch to raw curl to "bypass" the scanner
unless the user asks.

## PR create / merge bodies

Same file pattern: JSON on disk, `--body-file`. Keep titles short; keep bodies
readable without a fence forest.
