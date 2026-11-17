# Git remotes, paths, Forgejo, and CI

Estate operating facts for clone/push, the `forgejo` CLI, org layout, and
Actions on the build box. Load this when touching git remotes, Forgejo, PRs,
CI waits, local clone paths, or workflow files.

## Mental model (read first)

| Plane | Role |
| --- | --- |
| **Forgejo** `forgejo.softwarepatterns.com` | **Primary** git host and source of truth. Repo storage is JuiceFS-backed on the builder estate. |
| **GitHub** | **Mirror of `main`** for the public face where a mirror exists — not the day-to-day origin for estate work. |
| **`softwarepatterns`** | Public / business org (shared face of the company). |
| **`ironspecs`** | Private org for the individual developer — not public distribution. |
| **Local disk** | Always `…/<org-or-user>/<repo>/` — same shape as the host path `owner/repo`. |
| **`git`** | Porcelain: clone, commit, branch, push, fetch. |
| **`forgejo` CLI** | Agent-first **HTTP API** client (issues, PRs, Actions metadata, etc.). Not a git replacement. |

Agents are often surprised by this layout the first time. Prefer this file over
re-deriving remotes, inventing `ubuntu-latest`, or treating GitHub as canonical.

---

## Local paths (every machine)

Clones MUST live under a directory named for the Forgejo/GitHub **owner**, then
the **repo**:

```text
~/Projects/softwarepatterns/agents-setup
~/Projects/softwarepatterns/forgejo-actions
~/Projects/ironspecs/clis-rs
~/Projects/ironspecs/services-rs
```

| Rule | Detail |
| --- | --- |
| Shape | `<org-or-user>/<repo>/` under the machine’s projects root (commonly `~/Projects/`) |
| Match remote | Directory names match `owner` and `repo` in `owner/repo` and in SSH URLs |
| No flat dumps | Do not clone everything into one undifferentiated folder |
| Worktrees | Stay related to that repo tree; do not break the `owner/repo` identity |

Rationale: scripts, sibling `file:` dependencies, muscle memory, and CI
`owner/repo` cache keys all assume this shape.

Examples:

- Good: `~/Projects/softwarepatterns/agents-setup`
- Bad: `~/Projects/agents-setup` with no org segment
- Bad: `~/src/random-clone-name` for `ironspecs/clis-rs`

---

## Remotes and hosts

### Primary: Forgejo

```text
ssh://git@forgejo.softwarepatterns.com:2222/<owner>/<repo>.git
# HTTPS (when used):
https://forgejo.softwarepatterns.com/<owner>/<repo>.git
```

- SSH is common on estate machines (port **2222**).
- **`origin` should point at Forgejo** for normal develop/push/PR work.
- Web UI and API: `https://forgejo.softwarepatterns.com/`.

### Mirror: GitHub

- Where configured, **GitHub tracks `main`** as a public mirror of the Forgejo
  primary — especially for **`softwarepatterns/*`** public repos.
- Do not treat GitHub as the place to open day-to-day PRs or as the only remote
  unless the task is explicitly about the mirror.
- **`ironspecs/*`** is private developer work — not the public GitHub face.

### Org cheat sheet

| Owner | Visibility | Use |
| --- | --- | --- |
| `softwarepatterns` | Public / business | Shared products, actions, taps, this catalog |
| `ironspecs` | Private | Personal/private monorepos and experiments |

When creating remotes or documenting clone URLs, keep owner and visibility
straight so private material does not land on the public org by habit.

---

## `git` vs `forgejo` CLI

| Tool | Use for |
| --- | --- |
| **`git`** | clone, fetch, commit, branch, rebase, push, worktrees |
| **`forgejo`** | Forgejo HTTP API: repos, issues, pulls, comments, commit statuses, packages, org/user admin surfaces exposed by swagger, etc. |

The binary is **`forgejo`** (from the estate forgejo-cli product). It is
**agent-first**:

- **JSON on stdout** by default (`--format json`)
- Structured errors on **stderr**
- Stable exit codes; **no TUI** for normal API use
- Commands follow noun/verb grammar from Forgejo swagger (e.g.
  `forgejo repos pulls get --owner O --repo R --pull N`)

**Out of scope for `forgejo`:** git porcelain (`clone` / `commit` / `push`).
Use `git` for those.

Login once per server (PAT or password→token); API calls use the stored
credential. Prefer non-interactive flags in agent/CI contexts.

```bash
forgejo --help
forgejo repos pulls get --help
# typical shape:
forgejo repos pulls get --owner softwarepatterns --repo agents-setup --pull 1
```

Globals worth knowing: `--server`, `--profile`, `--format` (`json` default),
`--api-version`, `--cacert`, `--insecure`.

### `--wait-until` (important)

On **wait-eligible read** leaves (not mutations), `forgejo` accepts:

```text
--wait-until '<JMESPath>'
```

Behavior (estate law from forgejo-cli):

| Rule | Detail |
| --- | --- |
| Predicate | JMESPath (legacy 2015-09-05) over the JSON response |
| Success | expression evaluates to JSON **boolean `true`** → print the satisfying document once on stdout and exit 0 |
| Keep polling | `false`, `null`, or **HTTP 404** on get/list (“not there yet”) |
| Invalid predicate | non-bool result or bad expression → fail **before** relying on a good wait; structured error, no false success |
| Mutations | **`--wait-until` is rejected** on merge/create-style leaves (usage error, no HTTP) |
| Timing | **Not CLI flags.** Intervals/timeouts come from baked **wait profiles** (defaults on the order of **30s interval / 10m timeout**; some ops tighter/looser, e.g. commit status vs reviews) |
| Timeout | exit **28**, Problem Details `wait_timeout`; **stdout empty** (only a satisfying body is ever printed) |

Use this instead of sleep-loops or scraping the web UI when waiting for PR
fields, commit status, reviews, or other API-visible state.

Illustrative pattern (adjust owner/repo/pull and the expression to the real
response shape):

```bash
forgejo repos pulls get \
  --owner ironspecs --repo services-rs --pull 42 \
  --wait-until 'mergeable == `true`'
```

For commit CI status, prefer the status get operation with a predicate on the
combined state once you confirm the JSON shape for that endpoint—still
`--wait-until`, still no ad-hoc sleep.

**Legacy note:** some older repo docs still mention `tea`. For new agent work,
prefer **`forgejo`**. `tea` does not replace `--wait-until` and is weaker for
Actions log/API workflows.

---

## Forgejo Actions (CI)

### Workflow home

- Prefer **`.forgejo/workflows/`**.
- If `.forgejo/` exists, Forgejo **ignores** `.github/workflows/` for Actions.
- A repo with only `.github/workflows/` and `ubuntu-latest` will **mis-schedule
  or hang** on this estate: the runner label is **`bookworm`**, not GitHub-hosted
  `ubuntu-latest`.

### Job environment (build-box)

| Fact | Detail |
| --- | --- |
| Runner | Self-hosted on the builder box |
| Isolation | Each job runs in a **disposable LXC** (amd64 Debian bookworm template) |
| Label | `bookworm` (LXC + nested docker capability) |
| Docker | **dockerd inside the job LXC** — not the host Docker socket |
| Image contents (typical) | curl, git, node, docker+compose, sops, age — **no bun/jq guarantee**; use `python3` for JSON when unsure |
| Checkout | Often `uses: https://forgejo.softwarepatterns.com/actions/checkout@…` (estate-hosted action mirror) |

### `/cache` (per repo, shared across that repo’s workflows)

| Fact | Detail |
| --- | --- |
| Path in job | **`/cache`** on every `bookworm` job (no special label) |
| Sharing | **All workflows/jobs for the same `owner/repo` share one cache tree** |
| Size | **~8–10 GiB useful/hard capacity** per repo (loop-backed; treat as finite) |
| Concurrency | Concurrent jobs **share** the mount — use workflow `concurrency` if you need exclusive writers |
| Not JuiceFS | Job rootfs, `/cache`, and nested dockerd are **box-local disk**. Forgejo **repo** data is JuiceFS; **CI scratch/cache is not** |

Common consumer pattern (Rust monorepos):

```text
CARGO_HOME=/cache/cargo
SCCACHE_DIR=/cache/sccache
```

Do not assume infinite disk or a shared host-global cache across unrelated
repos. Repo rename ⇒ new empty cache key (`owner/repo`).

### Tokens in Actions

Steps typically see **`FORGEJO_TOKEN` and `GITHUB_TOKEN` as the same token under
two names** (Forgejo compatibility). Use them for API/git as the action docs
describe; do not invent GitHub-only auth schemes on Forgejo.

---

## Practical agent checklist

1. **Path:** work under `…/<owner>/<repo>/` matching the remote.  
2. **`origin`:** Forgejo SSH/HTTPS; push/PR there.  
3. **GitHub:** mirror of `main` when present — not default `origin`.  
4. **Org:** `softwarepatterns` public face; `ironspecs` private.  
5. **History:** `git` for commits; **`forgejo`** for API; **`--wait-until`** to
   block on API state.  
6. **CI:** write/fix **`.forgejo/workflows`**, `runs-on: bookworm`, own
   dockerd, `/cache` ~8–10 GiB per repo.  
7. **Do not:** invent `ubuntu-latest` as the estate runner, put job cache on
   JuiceFS, or sleep-poll the UI when `forgejo … --wait-until` applies.

---

## Related deeper docs (optional)

When a consumer repo already vendors longer Forgejo research, prefer linking
rather than forking:

- build-box `persist/README.md` — `/cache` contract  
- `softwarepatterns/forgejo-actions` `.agents/references/forgejo.md` — events,
  tokens, API deltas, merge gates  
- clis-rs `apps/forgejo-cli/README.md` — full wait law and credential model  

This file is the **short shared injection** so agents discover the estate shape
before they are “delighted/surprised” mid-task.
