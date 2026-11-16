---
name: release
description: Cut and publish a forgejo release. Use when opening a release (tag), building/signing/uploading a leg, or capping a release.
---

# Release

## The model

Five steps, three kinds of actor:

1. **Open.** Push a `forgejo/v{semver}` tag. The tag means exactly one
   thing: the `release-draft` workflow creates the **draft** release —
   always `draft: true`; a human never opens a release by hand. Idempotent:
   a re-tag finds the existing release and does nothing.
2–4. **Build, sign, upload.** Each actor builds their OS's artifacts on
   their own machine, signs with their OS's own machinery, and uploads to
   the draft. **The cap never signs a binary — every OS signs its own.**
5. **Cap.** One dispatch, `release.yml`: completeness, verify the
   actors' signatures, `SHA256SUMS` (signed — the one artifact the cap
   creates), stage the public channels, push the formula, publish the
   Forgejo release. One run, no mid-cap checkpoint — the capper's smoke
   test happens *before* the dispatch, and a failed cap re-runs from the
   top (both stages are idempotent).

## Critical

- **Every OS signs its own artifacts.** Apple signs + notarizes on the Mac;
  Windows signs on Windows; the linux actors sign their debs with the
  business GPG subkey at build time. No cross-signing, ever.
- **No linux cross-compilation, ever** — each linux target builds natively
  on a machine of its own architecture (amd64 on the server, arm64 on the
  Mac's Docker). **Sanctioned exception:** both darwin targets build on the
  Mac — the darwin toolchain and SDK are arch-agnostic there, and macOS
  binaries must come from Apple hardware.
- **The draft is machine-guaranteed.** Only `release-draft` creates
  releases, always as drafts; only the cap's last step publishes.
- **Publish never rebuilds.** The cap downloads the draft's assets.
- **Per-CLI versioning.** `forgejo-cli` carries its own `version` in its
  `Cargo.toml`. Bump it on `main` before cutting the tag — and cut the tag
  when the pipeline is ready, because the legs build the tag.
- **Secrets are sops-only.** Release secrets live in `releases.enc.yaml`
  (age-encrypted, committed); the only Forgejo workflow secret is
  `SOPS_AGE_KEY`. Never put a plaintext secret in the repo or a workflow,
  and never decrypt more than your own section.
- **Uploaders verify before they upload** — each actor runs their own smoke
  test; the cap cannot re-verify Apple or Windows signatures (it runs on
  Linux), so the actors' own `spctl`/`codesign` (mac) and Windows tooling
  checks are the contract.

## Channels

| Channel | Content | Visibility |
|---|---|---|
| private Forgejo `ironspecs/clis-rs` | source, workflows, the published release — the canonical record (JuiceFS → S3 backup) | private org |
| `softwarepatterns/homebrew-tap` (Forgejo) | the **tap of record**: formulae, `install-deb.sh`, `public.gpg` — the cap pushes the formula here | public |
| `softwarepatterns/homebrew-tap` (GitHub) | the public copy of the tap (synced) + the public release assets: binaries, checksums, signatures | public |
| `softwarepatterns/scoop-bucket` (Forgejo) | the **bucket of record**: Scoop manifests — the cap pushes the manifest here | public |
| `softwarepatterns/scoop-bucket` (GitHub) | the public copy of the bucket (synced) | public |

Public channels carry binaries, checksums, signatures, and packaging
metadata only — never source (closed source, public binaries).

## Build targets

| Target | Host | Signing |
|---|---|---|
| `x86_64-unknown-linux-gnu` | the server via `release-leg-amd64.yml` (on-demand — the server is the CI runner) | the actor: business GPG subkey, in the container |
| `aarch64-unknown-linux-gnu` | the Mac's Docker (native arm64) | the actor: business GPG subkey, in the container |
| `x86_64-apple-darwin` | the Mac (Apple hardware) | the actor: Developer ID + notarytool |
| `aarch64-apple-darwin` | the Mac (Apple hardware) | the actor: Developer ID + notarytool |
| `x86_64-pc-windows-msvc` | Windows | unsigned — the GPG integrity chain (the cap-signed `SHA256SUMS`) covers it; Authenticode deferred |

The naming contract (the `assets` list in `releases.enc.yaml`) defines
every expected name, including the linux `.asc` signatures and the Windows
msi row (`releases.yaml.example` carries it — the encrypted contract must
include it before the first cap run; the cap fails loudly listing what is
missing). The Windows leg is implemented (`release-leg.sh` msvc branch,
`release-env.sh` windows section, `wix/main.wxs` authored) and ships
unsigned for now — integrity comes from the cap-signed `SHA256SUMS` (Scoop
manifests carry the same hash). Authenticode is deferred until a signing
path is chosen.

## Secrets and settings

Release secrets live in **`releases.enc.yaml`** — sops age-encrypted, committed,
one file for the whole release. Its gitignored plaintext twin `releases.yaml`
is the local working copy (estate convention: plaintext local, encrypted
committed); `releases.yaml.example` (committed) shows the shape for readers
without a key. Edit the twin with `sops edit releases.enc.yaml` (the
catch-all creation rule in `.sops.yaml` targets the estate's four age
recipients: Personal / Production / Backup / Windows). Every consumer
decrypts **only its own section** (`sops -d --extract`; types are always
explicit — `--input-type yaml`):

| Section | Key | Consumer | Role |
|---|---|---|---|
| `server` | `gpg_signing_key_base64` | the linux actors + the cap | the business signing subkey (identity Software Patterns LLC, subkey `D073F8087305E5D2C4431376AB91371C6069F19F`) — actors sign their debs with it, the cap signs `SHA256SUMS` with it |
| `server` | `tap_github_pat` | the cap | write access to `softwarepatterns/homebrew-tap` on GitHub — stages the public release (binaries, checksums, signatures) |
| `server` | `tap_forgejo_pat` | the cap | write access to the tap and bucket of record on Forgejo — `public.gpg` fetch, the formula and Scoop manifest pushes, and the draft-asset downloads (its scope covers all three: `softwarepatterns/homebrew-tap`, `softwarepatterns/scoop-bucket`, `ironspecs/clis-rs`) |
| `apple` | `signing_identity` · `cert_base64` · `cert_password` · `keychain_password` · `api_key_base64` · `api_key_id` · `api_key_issuer` | the mac actor (`just release-env`) | Developer ID cert + App Store Connect notarytool credentials. `api_key_issuer` is a UUID for **Team** keys and **empty for Individual** keys (notarytool omits `--issuer` — this slot legitimately holds `""`) |
| `windows` | — (empty while unsigned) | the windows actor (`just release-env`) | deferred: no Authenticode material until a signing path is chosen |

- **The only Forgejo workflow secret is `SOPS_AGE_KEY`** (the estate CI age
  key). Server actions decrypt just the `server` section; actors decrypt
  with their machine's own age key at `~/.config/sops/age/keys.txt` via
  `eval "$(just release-env)"` (needs sops on PATH).
- The `server` section is **one decrypt unit**: a linux actor decrypting it
  for `gpg_signing_key_base64` also receives the tap PATs. Accepted — the
  actors are the estate's machine owners, and the PATs never leave their
  machines.
- Unfilled slots hold the `REPLACE_ME` placeholder — sops encrypts only
  non-empty values, so a placeholder keeps an empty slot encrypted and
  visibly unfilled. A leftover `REPLACE_ME` means the slot was never
  filled.
- `releases.yaml` is the gitignored plaintext working copy; the committed
  twin holds the same structure encrypted, and the cap reads the naming
  contract from the twin. Data, not code; the config is not a secret.

The public GPG key lives at `softwarepatterns/homebrew-tap/public.gpg` on
the tap of record (Forgejo), synced to the public GitHub copy.

## The trigger surface

| Trigger | What runs |
|---|---|
| `forgejo/v*` tag push | `release-draft` — creates the draft, always draft |
| actor commands | `release-leg.sh` (build + sign) + `release-upload.sh`; the amd64 actor dispatches `release-leg-amd64.yml` |
| capper dispatch | `release.yml` — assemble + publish in one run |

Nothing else is wired. No environment gates (this Forgejo has no
environments feature); the human gates are the capper's smoke test and the
dispatch itself.

## Artifact naming

- macOS: `forgejo-{version}-{target}.tar.gz` (signed binary) and
  `forgejo-{version}-{target}.dmg` (notarized + stapled, direct downloads)
- Linux: `forgejo_{version}_{arch}.deb` + `forgejo_{version}_{arch}.deb.asc`
  (the actor's GPG signature)
- `SHA256SUMS` + `SHA256SUMS.asc` — created and signed by the cap

## End-user installation

- macOS: `brew tap softwarepatterns/tap && brew trust softwarepatterns/tap && brew install forgejo-cli`
  (the formula is named `forgejo-cli` — the bare name `forgejo` is the
  Forgejo *server* in homebrew/core, and third-party taps need trust)
- Linux: `curl -sSL https://github.com/softwarepatterns/homebrew-tap/raw/main/install-deb.sh | bash -s forgejo`
- Windows: `scoop bucket add softwarepatterns https://github.com/softwarepatterns/scoop-bucket && scoop install forgejo` — Scoop extracts the msi (an administrative layout, no installer run) into its dir; the manifest's `extract_dir` + `bin` point at the laid-out binary

## Known limitations (current)

- **The Forgejo and package-manager behaviors that shaped the pipeline**
  (asset metadata vs bytes, token realms, tag mechanics, the formula
  namespace) are catalogued in
  `.agents/references/forgejo-behaviors.md`. The one release-critical law,
  restated: the public release tag must live in the tap of record — the
  sync that copies refs to the public copy prunes refs absent from source,
  and a pruned tag turns the public release into a zombie.
- Direct downloads ship a **stapled, notarized dmg** — zips cannot be
  stapled (`stapler` refuses them), and an unstapled ticket leaves
  Gatekeeper's offline check empty. The tarball carries the signed binary
  for brew; its ticket does not survive tar extraction, so direct-download
  users should take the dmg. A brand-new Developer ID still shows the
  standard first-run confirmation prompt until it builds reputation.
- No `.rpm`, no APT repo.
- `SHA256SUMS` is GPG-signed; provenance attestation and reproducible builds
  are roadmap items after signing is proven.

## Instructions

To cut a release:

1. **Open:** bump `version` in `apps/forgejo-cli/Cargo.toml`, commit on
   `main`, confirm `just gate`, push the tag `forgejo/v{semver}`. The draft
   appears automatically (`release-draft`) — verify it exists before the
   actors start.
2. **Build, sign, upload (each actor):**
   - **linux (both actors):** decrypt the signing key once, then build:
     `export GPG_SIGNING_KEY_BASE64="$(sops -d --input-type yaml --output-type json --extract '["server"]' releases.enc.yaml | python3 -c 'import json,sys; print(json.load(sys.stdin)["gpg_signing_key_base64"])')"`.
     The arm64 actor builds in the Mac's Docker (`just release-build
     forgejo-cli aarch64-unknown-linux-gnu`); the amd64 actor dispatches
     `release-leg-amd64.yml` (it decrypts the same key itself). Upload the
     deb **and** its `.asc`.
   - **mac:** `eval "$(just release-env)"` → `just release-build
     forgejo-cli x86_64-apple-darwin` and `aarch64-apple-darwin` →
     `stapler validate` + `spctl -a -t open --context context:primary-signature`
     on each dmg, `codesign --verify --strict` on the mounted binary → upload
     the four artifacts. (Apple signing gotchas live in
     `.agents/references/apple-signing.md`.)
   - **windows:** check out the **tag** and build from it — actors build
     the tag's source, never main (a stale local tag makes `git fetch`
     refuse to clobber; that refusal is the tell). Run the recipes from
     Git Bash (the recipes invoke `bash`; the WSL shim on a default
     Windows PATH is not one). After `release-build`, install the msi
     (UAC — the msi is per-machine by design) and smoke-test it against
     the live private instance, and check its sha256 against the name it
     will carry in the `SHA256SUMS` — the owner's smoke test is the
     contract. Then upload.3. **Cap:** download your platform's artifact from the draft and run it
   (the capper's smoke test), then dispatch `release.yml` with the tag.
   It verifies completeness and the actors' signatures, signs
   `SHA256SUMS`, stages the public channels, and publishes the Forgejo
   release.
4. **Verify:** the tap release + formula land, and
   `brew install forgejo-cli` / the `install-deb.sh` path install the published
   version.
