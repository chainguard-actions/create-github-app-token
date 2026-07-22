<!-- markdownlint-disable -->

# Hardening Report: actions--create-github-app-token/v1.12.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--create-github-app-token/v1.12.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags instead of full 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the tag is moved.

.github/workflows/test.yml:
  - uses: actions/checkout@v4
  - uses: actions/setup-node@v4
  - uses: octokit/request-action@v2.x

.github/workflows/release.yml:
  - uses: actions/checkout@v4
  - uses: actions/setup-node@v4

.github/workflows/publish-immutable-action.yml:
  - uses: actions/checkout@v4
  - uses: actions/publish-immutable-action@v0.0.4

Locations:

- `.github/workflows/test.yml:19`
- `.github/workflows/test.yml:22`
- `.github/workflows/test.yml:33`
- `.github/workflows/test.yml:35`
- `.github/workflows/test.yml:42`
- `.github/workflows/release.yml:17`
- `.github/workflows/release.yml:22`
- `.github/workflows/publish-immutable-action.yml:12`
- `.github/workflows/publish-immutable-action.yml:14`

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is interpolated directly inside a run: shell command. The step `run: echo '${{ steps.get-repository.outputs.data }}'` injects the value of steps.get-repository.outputs.data (an API response from octokit/request-action) directly into the shell command string before the shell ever sees it. If the API response contains shell metacharacters or newlines, this can lead to command injection.

Locations:

- `.github/workflows/test.yml:51`

### missing-permissions (severity: medium)

The workflow file test.yml has no top-level `permissions:` key and neither of its jobs (`integration`, `end-to-end`) defines a job-level `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (write-all by default for many repositories).

Locations:

- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across three workflow files:

1. **unpinned-uses**: Pinned all 4 action references to full 40-char SHAs with tag comments preserved: actions/checkout@11d5960a..., actions/setup-node@49933ea5..., octokit/request-action@02f5e7c6..., actions/publish-immutable-action@4bc8754f...

2. **script-injection** (test.yml line 51): Moved `${{ steps.get-repository.outputs.data }}` out of the `run:` shell string into an `env:` block as `REPOSITORY_DATA`, then referenced it as `echo "$REPOSITORY_DATA"` in the shell script.

3. **missing-permissions** (test.yml): Added `permissions: {}` at the workflow top level and `permissions: contents: read` at the job level for both `integration` and `end-to-end` jobs, which need to check out code.

