# `--wait-until` deep reference

## Eligibility

`--wait-until <JMESPath>` appears on wait-eligible **list/get reads**. Mutations
reject it. Discover per command:

```bash
forgejo repos pulls reviews list --help
forgejo repos pulls list --help
forgejo actions run list --help
```

If the help text lacks `--wait-until`, that command is not wait-eligible.

## Semantics

- JMESPath legacy spec 2015-09-05.
- Poll until the expression evaluates to boolean **`true`**.
- On success: print the satisfying document **once** on stdout, exit **0**.
- Keep polling on `false`, `null`, and HTTP **404**.
- Timing: baked-in profiles (order of ~30s interval / ~10 min timeout) — **no**
  CLI flags for interval/timeout.
- Timeout: exit **28**, **empty** stdout. Treat 28 as "condition not met in
  time", not as "flag broken".

## Visibility law

Run wait on the forgejo process the agent observes:

```bash
forgejo repos pulls reviews list \
  --owner O --repo R --pull N --format json \
  --wait-until 'length([?dismissed==`false`]) >= `1`'
```

NEVER:

```bash
# bad: hides exit 28 and stderr
out=$(forgejo ... --wait-until '...' 2>/dev/null) || true
```

## Review waits that survive dogfood

### Why raw length lies

- Parallel jobs post two reviews; order is undefined.
- Push after REQUEST_CHANGES marks old reviews `dismissed: true` / `stale: true`
  but they **remain in the list**.
- `length(@) > 1` can be true from one live + one dismissed review.

### Prefer non-dismissed counts

```bash
# first live review (any lens)
--wait-until 'length([?dismissed==`false`]) >= `1`'

# two live reviews (e.g. parallel code + docs jobs)
--wait-until 'length([?dismissed==`false`]) >= `2`'
```

### Pin to head commit when needed

After synchronize, wait for live reviews whose `commit_id` matches the new
head (full 40-char SHA from the PR). String literals are JSON inside
backticks — quote them, or the predicate fails `wait_predicate_invalid`:

```bash
--wait-until 'length([?dismissed==`false` && commit_id==`"e26c7e4a1b2c3d4e5f60718293a4b5c6d7e8f901"`]) >= `1`'
```

### APPROVED-only

When the goal is approval, not merely "a review existed":

```bash
--wait-until 'length([?dismissed==`false` && state==`"APPROVED"`]) >= `1`'
```

Combine with count ≥ 2 when two lenses must both approve.

### After you already hold a list

If you fetched reviews without wait, set the next threshold from **current**
non-dismissed length + 1 (or required total), then call list again with
`--wait-until`. Do not hard-code `> 0`, `> 1` across pushes without re-reading
state.

## Actions runs

Debugging dogfood CI: `forgejo actions run list --help` (wait-eligible). Use
JMESPath over the run list shape the command returns — inspect one JSON sample
without wait before writing the predicate.
