<!-- markdownlint-disable -->

# Hardening Report: actions--create-github-app-token/v1.12.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **actions--create-github-app-token/v1.12.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags instead of pinned SHA digests, making them vulnerable to supply-chain attacks. Failing references: publish-immutable-action.yml: actions/checkout@v4, actions/publish-immutable-action@v0.0.4; release.yml: actions/checkout@v4, actions/setup-node@v4; test.yml: actions/checkout@v4, actions/setup-node@v4 (twice), octokit/request-action@v2.x.

Locations:

- `.github/workflows/publish-immutable-action.yml:12`
- `.github/workflows/publish-immutable-action.yml:14`
- `.github/workflows/release.yml:18`
- `.github/workflows/release.yml:21`
- `.github/workflows/test.yml:19`
- `.github/workflows/test.yml:21`
- `.github/workflows/test.yml:33`
- `.github/workflows/test.yml:35`
- `.github/workflows/test.yml:44`

### missing-permissions (severity: medium)

test.yml has no top-level permissions: key and neither of its jobs (integration, end-to-end) defines a job-level permissions: block. Without explicit permissions, the workflow inherits the default repository token permissions, which may be overly broad.

Locations:

- `.github/workflows/test.yml:1`

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is interpolated directly inside a run: shell command. In test.yml, the step `run: echo '${{ steps.get-repository.outputs.data }}'` injects the output of an API call directly into the shell command string before the shell ever sees it. An attacker who can influence the API response could inject arbitrary shell commands. The offending line is: run: echo '${{ steps.get-repository.outputs.data }}'

Locations:

- `.github/workflows/test.yml:50`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across three workflow files:

1. unpinned-uses: Pinned all action references to full commit SHAs:
   - actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5 # v4 (in all 3 files)
   - actions/publish-immutable-action@v0.0.4 → @4bc8754ffc40f27910afb20287dbbbb675a4e978 # v0.0.4
   - actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020 # v4 (in release.yml and test.yml)
   - octokit/request-action@v2.x → @02f5e7c637a73a3b12ed81015fa7fb5f11cc5d7d # v2.x

2. missing-permissions: Added top-level `permissions: contents: read` block to test.yml.

3. script-injection: In test.yml, moved `${{ steps.get-repository.outputs.data }}` from the run: shell string into an env: block as REPOSITORY_DATA, then referenced it as `$REPOSITORY_DATA` in the shell command to prevent shell injection.

