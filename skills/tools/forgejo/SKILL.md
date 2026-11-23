---
name: forgejo
description: >
  Drive the estate's agent-first Forgejo API client - PR/issue/review
  commands, --wait-until blocking reads, and the body-scanner gotcha. Use
  when creating PRs, commenting, waiting on reviews, or reading commit
  statuses from the forgejo CLI.
---

# Forgejo CLI

The `forgejo` binary is the estate's agent-first HTTP API client for Forgejo
(the estate forgejo-cli product). It covers issues, pull requests, comments,
reviews, and commit statuses. It is **not** git porcelain - clone, commit, and
push stay with `git`.

## Verified (ran against forgejo.softwarepatterns.com, 2026-11-22/23)

- **Output is JSON on stdout by default** (`--format json`; also `plain`,
  `ids`). Structured errors go to stderr; exit codes are stable (see timeout
  below). No TUI for normal API use.
- **Create a PR:** `forgejo repos pulls create --owner O --repo R --body-file <path>`.
  The body file is JSON: `{"base": "main", "head": "<branch>", "title": "...",
  "body": "..."}`. The response is the full PR JSON, including `number`,
  `html_url`, and `mergeable`.
- **Comment on a PR:** PRs are issues for comment purposes.
  `forgejo repos issues comments create --owner O --repo R --issue N --body-file <path>`
  with body `{"body": "..."}`. List with `repos issues comments list`; delete
  with `repos issues comments delete --issue N --comment <id>`.
- **Block until a review lands:** `--wait-until '<JMESPath>'` works on
  wait-eligible reads (not mutations - it is rejected there). It polls until
  the expression evaluates to boolean `true`, prints the satisfying document
  once on stdout, and exits 0. It keeps polling on `false`, `null`, and 404.
  Timing comes from baked-in profiles (on the order of 30s interval / 10 min
  timeout), not CLI flags. Timeout exits **28** with empty stdout.
  Proven form for "next review on this PR":
  `forgejo repos pulls reviews list --owner O --repo R --pull N --format json --wait-until 'length(@) > \`N\`'`
- **The reviewer is the `forgejo-actions` account** (Forgejo Actions). Reviews
  arrive as `REQUEST_CHANGES` or `APPROVED`, and the body ends with an HTML
  comment pinning the reviewed head commit - new commits invalidate the old
  review (`dismissed: true`, `stale: true`).
- **Gotcha - the body scanner:** comment bodies whose markdown leans on inline
  code backticks are rejected with
  `{"type":"unknown","Portable shell scanner cannot analyze command: invalid-structure"}`.
  Plain prose (numbered lists, no code spans) passes. Keep code out of comment
  bodies or quote it minimally.

## Documented (estate reference, not re-verified here)

- Server web UI and API: `https://forgejo.softwarepatterns.com`. Git over SSH
  is port **2222**. PRs and day-to-day work target Forgejo; GitHub (where a
  mirror exists) is a mirror of `main`, not the work remote.
- Command grammar is noun/verb from the Forgejo swagger
  (`forgejo <noun> <verb>`, e.g. `repos pulls get`). `--help` per command lists
  exact flags.

## Typical agent workflow

1. `git checkout -b <branch>`, commit, `git push -u origin <branch>`.
2. `forgejo repos pulls create --body-file pr.json` -> PR number.
3. `forgejo repos pulls reviews list --wait-until 'length(@) > \`0\`'` -> wait
   for the first review.
4. Read findings, fix, push, reply with
   `forgejo repos issues comments create --body-file reply.json`.
5. Repeat the wait with the count bumped (`> \`1\``, `> \`2\``, ...) until
   APPROVED.
