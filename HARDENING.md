<!-- markdownlint-disable -->

# Hardening Report: lowply--build-hugo/v0.162.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **lowply--build-hugo/v0.162.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files reference `actions/checkout@v4` using a mutable tag instead of a pinned 40-character commit SHA. This exposes the action to supply-chain attacks if the tag is moved. Failing references: `actions/checkout@v4` in catchup.yml, release.yml, and test.yml.

Locations:

- `.github/workflows/catchup.yml:13`
- `.github/workflows/release.yml:9`
- `.github/workflows/test.yml:11`

### script-injection (severity: high)

Sub-rule (a): A `${{ ... }}` expression is interpolated directly inside a `run:` shell command string in release.yml. The offending line is: `echo ${{ secrets.GITHUB_TOKEN }} | gh auth login --with-token`. The expression is expanded by the YAML template engine before the shell sees it, bypassing shell quoting and making the value injectable. The token should be passed via an environment variable instead (e.g. `env: GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}` and then `echo "$GH_TOKEN" | gh auth login --with-token`).

Locations:

- `.github/workflows/release.yml:12`

### missing-permissions (severity: medium)

None of the three workflow files define a `permissions:` block at the top level or at the job level. Without explicit permissions, workflows inherit the repository's default token permissions, which may be overly broad (e.g. write access to contents, pull-requests, etc.). Each workflow should declare the minimal set of permissions required.

Locations:

- `.github/workflows/catchup.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three workflow files: (1) Pinned actions/checkout@v4 to full SHA 11d5960a326750d5838078e36cf38b85af677262 in catchup.yml, release.yml, and test.yml. (2) Fixed script injection in release.yml by moving ${{ secrets.GITHUB_TOKEN }} into a step-level env var (GH_TOKEN) and referencing it as "$GH_TOKEN" in the shell script. (3) Added top-level permissions blocks: contents:write for catchup.yml and release.yml (which push/create releases), and contents:read for test.yml (read-only).

