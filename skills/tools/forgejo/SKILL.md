---
name: forgejo
description: >
  Drive the estate forgejo-cli: PR create/comment/review/merge, --wait-until
  JMESPath (exit 28), dismiss-safe multi-reviewer waits, body-file and body
  scanner, discovery before claiming missing. Use for Forgejo PRs, reviews,
  comments, wait loops, or forgejo binary questions. Not git porcelain.
---

# Forgejo CLI

The `forgejo` binary is the estate **forgejo-cli** (HTTP API client). It covers
issues, pull requests, comments, reviews, actions runs, packages. It is **not**
git porcelain — clone, commit, push stay with `git`.

## Load order

1. This file (laws + default loop).
2. WHEN waiting on reviews or CI → [references/wait-until.md](references/wait-until.md)
3. WHEN creating/listing/merging PRs → [references/pull-requests.md](references/pull-requests.md)
4. WHEN commenting or hitting scanner errors → [references/comments.md](references/comments.md)
5. Consumer repo platform facts (tokens, runner, packages) stay in that repo's
   `.agents/references/` — do not re-derive them here.

## Discovery law

BEFORE claiming the binary or a flag is missing, MUST run:

```bash
command -v forgejo
forgejo --version
forgejo status
forgejo <noun> <verb> --help
```

NEVER invent curl-only or git-only Forgejo API workflows as a substitute without
being asked. NEVER bury `forgejo` in a shell that discards stdout, stderr, or
exit codes.

## I/O law

- JSON on **stdout** by default (`--format json`; also `plain`, `ids`).
- Structured errors on **stderr**.
- Grammar: `forgejo <noun> <verb>` from server swagger.
- Global flags: `--server`, `--profile`, `--format`, `--api-version`, `--cacert`,
  `--insecure`.
- Pass `--owner` and `--repo` explicitly when profile defaults are null
  (`forgejo config get`).

## Wait law

`--wait-until '<JMESPath>'` exists on wait-eligible **reads** only. Mutations
reject it. Timeout exits **28** with empty stdout. Interval/timeout are
baked-in (~30s / ~10 min), not CLI flags.

MUST attach `--wait-until` to the forgejo process itself. NEVER wrap wait so
errors disappear, then claim waiting is impossible.

Default review wait uses **non-dismissed** rows (parallel dogfood + push make
raw `length(@)` lie). Details: [references/wait-until.md](references/wait-until.md).

## Merge law

NEVER merge a pull request unless the user explicitly asks to merge. Open the
PR, wait, fix, comment — stop before `repos pulls merge create`.

## Body law

Prefer `--body-file` with a JSON file. Heavy markdown inline code backticks in
comment bodies trip the portable shell scanner. Plain prose passes. Details:
[references/comments.md](references/comments.md).

## Default agent loop

```bash
git checkout -b <branch>
# commit…
git push -u origin <branch>

# pr.json: {"base":"main","head":"<branch>","title":"...","body":"..."}
forgejo repos pulls create --owner O --repo R --body-file pr.json
# read number from JSON stdout

forgejo repos pulls reviews list --owner O --repo R --pull N --format json \
  --wait-until 'length([?dismissed==`false`]) >= `1`'

# fix, push, comment (see references/comments.md)
# wait again; for two parallel reviewers use >= `2`
# merge ONLY if the user said merge — see references/pull-requests.md
```

Estate auto-reviewer account is often `forgejo-actions`. Reviews carry
`APPROVED` or `REQUEST_CHANGES`. New commits mark prior bot reviews
`dismissed: true`, `stale: true`.

## Anti-patterns (ban → replacement)

| NEVER | Instead |
|---|---|
| Claim `forgejo` missing without `command -v` | Run discovery law |
| Claim `--wait-until` impossible | `forgejo <cmd> --help`; use on reads; exit 28 = timeout |
| `length(@) > N` after dismiss/parallel jobs | `length([?dismissed==\`false\`]) >= \`K\`` |
| `repos pulls get --pull N` | `get` is `--base`+`--head`; number via create/list/reviews |
| Merge without user ask | Stop after APPROVED; wait for "merge" |
| Hand-roll curl for routine PR/review | `forgejo` noun/verb |
| Hide wait in `$(...)` / pipes that drop exit | Run forgejo as the waited process |

## Verified context

Exercised against `https://forgejo.softwarepatterns.com` with forgejo-cli
**0.3.0** (2026-11-22/23, 2026-11-25). Re-check `--help` after CLI upgrades.
