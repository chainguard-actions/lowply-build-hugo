<!-- markdownlint-disable -->

# Hardening Report: lowply--build-hugo/v0.164.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **lowply--build-hugo/v0.164.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: A ${{ secrets.GITHUB_TOKEN }} expression is interpolated directly inside a run: shell command string in release.yml. Any ${{ ... }} expression inside a run: block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it. The offending line is: `echo ${{ secrets.GITHUB_TOKEN }} | gh auth login --with-token`. The token should be passed via an env: variable and referenced as $ENV_VAR instead.

Locations:

- `.github/workflows/release.yml:14`

### unpinned-uses (severity: high)

All three workflow files reference actions/checkout@v4, which is a mutable version tag rather than a full 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, creating a supply-chain risk. Each uses: reference should be pinned to a full SHA, e.g. actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.

Locations:

- `.github/workflows/catchup.yml:13`
- `.github/workflows/release.yml:12`
- `.github/workflows/test.yml:10`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level permissions: key, and none of the individual jobs define a job-level permissions: key. Without explicit permissions, workflows inherit the repository's default token permissions, which may be overly broad (e.g. write access to contents, packages, etc.). Each workflow should declare the minimal permissions required.

Locations:

- `.github/workflows/catchup.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across .github/workflows/release.yml, catchup.yml, and test.yml:
1. script-injection (release.yml line 14): Moved `${{ secrets.GITHUB_TOKEN }}` from the run: shell string into an env: block as GITHUB_TOKEN, referenced as `$GITHUB_TOKEN` in the shell script.
2. unpinned-uses (all three files): Pinned `actions/checkout@v4` to full SHA `actions/checkout@34e114876b0b11c390a56381ad16ebd13914f8d5 # v4`.
3. missing-permissions (all three files): Added top-level `permissions: contents: write` to release.yml and catchup.yml (both need write access), and `permissions: contents: read` to test.yml (read-only).

