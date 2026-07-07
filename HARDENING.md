<!-- markdownlint-disable -->

# Hardening Report: lowply--build-hugo/v0.164.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **lowply--build-hugo/v0.164.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files reference actions/checkout@v4, which is a mutable tag rather than a pinned 40-character commit SHA. This means the action could be silently updated to a different (potentially malicious) version without any change to the workflow file. Each should be pinned to a full SHA, e.g. actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.

Locations:

- `.github/workflows/catchup.yml:14`
- `.github/workflows/release.yml:11`
- `.github/workflows/test.yml:12`

### script-injection (severity: high)

Sub-rule (a): release.yml interpolates a ${{ ... }} expression directly inside a run: shell command. The line `echo ${{ secrets.GITHUB_TOKEN }} | gh auth login --with-token` embeds the expression into the shell script via YAML template substitution before the shell ever sees it. Any ${{ ... }} in a run: block is a script-injection risk regardless of the context it reads from. The value should be passed via an env: variable instead: set GH_TOKEN: ${{ secrets.GITHUB_TOKEN }} in the env block and use `echo "$GH_TOKEN" | gh auth login --with-token`.

Locations:

- `.github/workflows/release.yml:14`

### missing-permissions (severity: medium)

None of the three workflow files declare a top-level permissions: key, and none of the individual jobs declare a job-level permissions: key. Without explicit permissions, workflows inherit the repository's default token permissions (often write-all), violating the principle of least privilege. Each workflow should declare the minimal permissions required (e.g. contents: write for release creation, contents: read for checkout-only jobs).

Locations:

- `.github/workflows/catchup.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three workflow files: (1) Pinned actions/checkout@v4 to full SHA 34e114876b0b11c390a56381ad16ebd13914f8d5 in catchup.yml, release.yml, and test.yml. (2) Fixed script injection in release.yml by moving ${{ secrets.GITHUB_TOKEN }} into an env: block as GH_TOKEN and referencing it as "$GH_TOKEN" in the shell script. (3) Added top-level permissions blocks to all three workflows: catchup.yml and release.yml get contents: write (needed for pushing/creating releases), test.yml gets contents: read (checkout only).

