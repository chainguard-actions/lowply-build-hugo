<!-- markdownlint-disable -->

# Hardening Report: lowply--build-hugo/v0.163.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **lowply--build-hugo/v0.163.3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files reference `actions/checkout@v4`, which is a mutable tag rather than a pinned 40-character commit SHA. If the tag is moved (e.g. by a supply-chain compromise of the actions/checkout repository), the action will silently execute different code. Each file should pin to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/catchup.yml:13`
- `.github/workflows/release.yml:9`
- `.github/workflows/test.yml:10`

### permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` block, and none of the individual jobs define job-level `permissions:` blocks either. Without explicit permissions, workflows run with the default (often broad) repository token permissions. Each workflow should declare the minimal required permissions (e.g. `permissions: contents: read`).

Locations:

- `.github/workflows/catchup.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/test.yml:1`

### script-injection (severity: high)

Sub-rule (a): In `release.yml`, the `run:` block directly interpolates a `${{ secrets.GITHUB_TOKEN }}` expression into the shell command string: `echo ${{ secrets.GITHUB_TOKEN }} | gh auth login --with-token`. Any `${{ ... }}` expression interpolated directly inside a `run:` block is a script-injection risk because the value is substituted into the shell command before the shell parses it. The safe pattern is to pass the token via an environment variable and reference it as `$ENV_VAR` in the shell, e.g.:
```yaml
env:
  GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
run: gh auth login --with-token <<< "$GH_TOKEN"
```

Locations:

- `.github/workflows/release.yml:13`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

1. Pinned actions/checkout@v4 to full SHA 34e114876b0b11c390a56381ad16ebd13914f8d5 in all three workflow files (catchup.yml, release.yml, test.yml). 2. Added top-level permissions blocks: catchup.yml gets 'contents: write' (needs to push updates), release.yml gets 'contents: write' (needs to create releases), test.yml gets 'contents: read' (read-only). 3. Fixed script injection in release.yml: removed the 'echo ${{ secrets.GITHUB_TOKEN }} | gh auth login --with-token' line and instead set GH_TOKEN as an env var on the step — the gh CLI automatically uses the GH_TOKEN environment variable, so no explicit auth login is needed.

