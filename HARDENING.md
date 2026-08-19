<!-- markdownlint-disable -->

# Hardening Report: lowply--build-hugo/v0.162.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **lowply--build-hugo/v0.162.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files reference actions/checkout@v4, which is a mutable tag rather than a pinned 40-character commit SHA. If the tag is moved (e.g. by a supply-chain compromise), the action will silently execute different code. Each file should pin to a full SHA, e.g. actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.

Locations:

- `.github/workflows/catchup.yml:14`
- `.github/workflows/release.yml:12`
- `.github/workflows/test.yml:11`

### script-injection (severity: high)

Rule (a) violation: release.yml interpolates a ${{ secrets.GITHUB_TOKEN }} expression directly inside a run: shell command string: `echo ${{ secrets.GITHUB_TOKEN }} | gh auth login --with-token`. Any ${{ ... }} expression inside a run: block is substituted by the Actions template engine before the shell ever sees the string, bypassing shell quoting. The safe alternative is to pass the token via an env: variable and reference it as $GH_TOKEN, or use the built-in `gh auth login` with the GITHUB_TOKEN environment variable directly.

Locations:

- `.github/workflows/release.yml:15`

### missing-permissions (severity: medium)

None of the three workflow files declare a top-level permissions: block, and no job within them declares job-level permissions. Without explicit permissions, workflows run with the repository's default token permissions, which may be overly broad (e.g. write access to contents, packages, etc.). Each workflow should declare the minimal permissions required, e.g. `permissions: contents: read` at the top level or per job.

Locations:

- `.github/workflows/catchup.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three workflow files: (1) Pinned actions/checkout@v4 to full SHA 11d5960a326750d5838078e36cf38b85af677262 in all three files. (2) Fixed script injection in release.yml by removing the `echo ${{ secrets.GITHUB_TOKEN }} | gh auth login --with-token` line and instead setting GH_TOKEN as an env var — the gh CLI automatically uses GH_TOKEN for authentication, making the explicit login step unnecessary. (3) Added top-level permissions blocks: `contents: write` for catchup.yml (needs to push updates) and release.yml (needs to create releases), and `contents: read` for test.yml (only needs to read code).

