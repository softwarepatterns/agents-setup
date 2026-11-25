# Pull requests

## Create

```bash
forgejo repos pulls create --owner O --repo R --body-file pr.json
```

`pr.json` MUST be JSON:

```json
{
  "base": "main",
  "head": "<branch>",
  "title": "short imperative title",
  "body": "plain prose; minimize inline code backticks"
}
```

Stdout: full PR JSON. Read `number`, `html_url`, `mergeable`. *verified*
forgejo-cli 0.3.0 against forgejo.softwarepatterns.com.

Git: branch must exist on the remote (`git push -u origin <branch>`) before
create. Estate SSH git remote uses port **2222**
(`ssh://git@forgejo.softwarepatterns.com:2222/OWNER/REPO.git`).

## Get vs number

| Goal | Command |
|---|---|
| PR by base+head | `forgejo repos pulls get --owner O --repo R --base main --head <branch>` |
| PR number from create | use create response `number` |
| List / filter | `forgejo repos pulls list --owner O --repo R` (`--state`, `--head`, `--limit`, `--page`; wait-eligible) |

NEVER pass a PR index to `get` as if it were GitHub `gh pr view N` — this
CLI's `get` is base/head path bindings.

Reviews and issue comments use `--pull N` or `--issue N` (PR number).

## Reviews

```bash
forgejo repos pulls reviews list --owner O --repo R --pull N --format json
```

Wait: see [wait-until.md](wait-until.md).

Estate dogfood bot: user `forgejo-actions`. States include `APPROVED`,
`REQUEST_CHANGES`. Lineage footers from opencode-forgejo look like HTML
comments pinning head SHA, model, reviewer name — useful to tell lenses apart
when multiple jobs post.

## Merge

```bash
forgejo repos pulls merge create --owner O --repo R --pull N --body-file merge.json
```

Inspect body shape with:

```bash
forgejo repos pulls merge create --help
```

Also: `repos pulls merge list` (merged?), `repos pulls merge delete` (cancel
auto-merge).

**NEVER** run merge create unless the user explicitly asked to merge. APPROVED
reviews are not permission to merge.

## Related verbs

Under `forgejo repos pulls`: `commits`, `files`, `requested_reviewers`,
`reviews`, `update`, `pinned`. Always `--help` before first use of a verb.
