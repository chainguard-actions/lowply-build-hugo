<!-- markdownlint-disable -->

# Hardening Report: lowply--build-hugo/v0.163.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **lowply--build-hugo/v0.163.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files reference `actions/checkout@v4` using a mutable version tag instead of a full 40-character commit SHA. This exposes the action to supply-chain attacks if the tag is moved to a malicious commit. Failing references: `uses: actions/checkout@v4` in catchup.yml, release.yml, and test.yml.

Locations:

- `.github/workflows/catchup.yml:13`
- `.github/workflows/release.yml:10`
- `.github/workflows/test.yml:11`

### permissions (severity: medium)

missing-permissions: None of the workflow files define a top-level `permissions:` key, and no job within them defines job-level `permissions:` either. Without explicit permissions, workflows run with the default (potentially broad) token permissions.

Locations:

- `.github/workflows/catchup.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/test.yml:1`

### script-injection (severity: high)

Sub-rule (a): A `${{ ... }}` expression is interpolated directly inside a `run:` shell command string in release.yml. The offending line is: `echo ${{ secrets.GITHUB_TOKEN }} | gh auth login --with-token`. Even though this is a secrets context, any `${{ }}` expression directly in a run: block is a script-injection risk because the value is substituted into the shell command string before the shell parses it. The safe pattern is to pass the token via an environment variable instead.

Locations:

- `.github/workflows/release.yml:13`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

1. Pinned `actions/checkout@v4` to full SHA `11d5960a326750d5838078e36cf38b85af677262` (with `# v4` comment) in all three workflow files: catchup.yml, release.yml, and test.yml.
2. Added `permissions: {}` at the top level of all three workflow files to enforce least-privilege by default. For release.yml, added `permissions: contents: write` at the job level since the job needs to create GitHub releases.
3. Fixed script injection in release.yml: moved `${{ secrets.GITHUB_TOKEN }}` out of the `run:` shell string into the step's `env:` block as `GH_TOKEN`, then referenced it safely as `"$GH_TOKEN"` in the shell script.

