<!-- markdownlint-disable -->

# Hardening Report: actions--create-github-app-token/v3.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--create-github-app-token/v3.2.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable version tags instead of pinned 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved.

In release.yml: `actions/checkout@v6` (×2), `actions/setup-node@v6`.
In test.yml: `actions/checkout@v6` (×3), `actions/setup-node@v6` (×3), `octokit/request-action@v2.x`.
In stale.yml: `actions/stale@v10`.
In update-permission-inputs.yml: `actions/checkout@v6`, `actions/setup-node@v6`.

All of these should be replaced with their full 40-character SHA digest, e.g. `actions/checkout@<sha> # v6`.

Locations:

- `.github/workflows/release.yml:20`
- `.github/workflows/release.yml:33`
- `.github/workflows/release.yml:37`
- `.github/workflows/test.yml:24`
- `.github/workflows/test.yml:25`
- `.github/workflows/test.yml:39`
- `.github/workflows/test.yml:40`
- `.github/workflows/test.yml:43`
- `.github/workflows/test.yml:55`
- `.github/workflows/test.yml:56`
- `.github/workflows/stale.yml:24`
- `.github/workflows/update-permission-inputs.yml:23`
- `.github/workflows/update-permission-inputs.yml:24`

### script-injection (severity: high)

Several `run:` blocks interpolate `${{ ... }}` expressions directly into shell commands (sub-rule a), allowing an attacker to inject arbitrary shell commands.

1. `test.yml` line 46: `run: echo '${{ steps.get-repository.outputs.data }}'` — the step output value is interpolated directly into the shell command. Even inside single quotes in YAML, the expression is substituted by the Actions runner before the shell sees it.

2. `test.yml` line 67: `run: test "${{ steps.test.outcome }}" = "failure"` — a step output is interpolated directly into the shell command.

3. `update-permission-inputs.yml` line 38: `run: gh pr edit ${{ github.event.pull_request.number }} --title "${{ env.COMMIT_MESSAGE }}"` — `github.event.pull_request.number` is attacker-controlled (PR title/number from a pull_request event) and is interpolated directly into the shell command without quoting or sanitization. `env.COMMIT_MESSAGE` is also interpolated directly.

Fix: move values into `env:` variables and reference them as quoted shell variables, e.g. `"$VAR"`.

Locations:

- `.github/workflows/test.yml:46`
- `.github/workflows/test.yml:67`
- `.github/workflows/update-permission-inputs.yml:38`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all unpinned action references by resolving real commit SHAs: actions/checkout@v6→df4cb1c, actions/setup-node@v6→249970729, octokit/request-action@v2.x→02f5e7c, actions/stale@v10→1e223db. Fixed three script injection vulnerabilities in test.yml (lines 46 and 67) and update-permission-inputs.yml (line 38) by moving ${{ }} expressions into step env: blocks and referencing them as quoted shell variables.

