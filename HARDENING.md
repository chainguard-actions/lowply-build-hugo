<!-- markdownlint-disable -->

# Hardening Report: lowply--build-hugo/v0.160.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **lowply--build-hugo/v0.160.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is directly interpolated inside a run: shell command string. In release.yml line 13, `echo ${{ secrets.GITHUB_TOKEN }} | gh auth login --with-token` embeds the expression directly in the shell command. Any ${{ ... }} in a run: block is a script-injection risk because the value is substituted into the shell command string before the shell parses it.

Locations:

- `.github/workflows/release.yml:13`

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable version tags (@v4) instead of full 40-character commit SHA digests. This exposes the workflow to supply-chain attacks if the tag is moved to a malicious commit. Affected references: actions/checkout@v4 in catchup.yml, release.yml, and test.yml.

Locations:

- `.github/workflows/catchup.yml:13`
- `.github/workflows/release.yml:11`
- `.github/workflows/test.yml:10`

### missing-permissions (severity: medium)

None of the workflow files define a top-level permissions: key, and none of the individual jobs define a job-level permissions: key. Without explicit permissions, workflows run with the default repository token permissions, which may be broader than necessary (e.g., write access to contents). Each workflow should declare minimal required permissions.

Locations:

- `.github/workflows/catchup.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across catchup.yml, release.yml, and test.yml:
1. script-injection (release.yml line 13): Moved `${{ secrets.GITHUB_TOKEN }}` out of the run: shell string into an env: block (GITHUB_TOKEN), then referenced it as "$GITHUB_TOKEN" in the shell command to prevent shell injection.
2. unpinned-uses: Pinned all three `actions/checkout@v4` references to the full commit SHA `actions/checkout@11d5960a326750d5838078e36cf38b85af677262 # v4`.
3. missing-permissions: Added top-level `permissions: {}` to all three workflows, plus job-level minimal permissions: `contents: write` for catchup.yml (pushes updates) and release.yml (creates releases), and `contents: read` for test.yml (read-only checkout).

