<!-- markdownlint-disable -->

# Hardening Report: actions--create-github-app-token/v3.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--create-github-app-token/v3.0.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tags or version strings instead of pinned 40-character SHA digests, making them vulnerable to supply-chain attacks if the referenced tag is moved. Unpinned references found:
- publish-immutable-action.yml: `actions/checkout@v6`, `actions/publish-immutable-action@v0.0.4`
- release.yml: `actions/checkout@v6`, `actions/setup-node@v6`
- stale.yml: `actions/stale@v10`
- test.yml: `actions/checkout@v6`, `actions/setup-node@v6`, `octokit/request-action@v2.x`
- update-permission-inputs.yml: `actions/checkout@v6`, `actions/setup-node@v6`

Locations:

- `.github/workflows/publish-immutable-action.yml:14`
- `.github/workflows/publish-immutable-action.yml:16`
- `.github/workflows/release.yml:18`
- `.github/workflows/release.yml:21`
- `.github/workflows/stale.yml:22`
- `.github/workflows/test.yml:24`
- `.github/workflows/test.yml:26`
- `.github/workflows/test.yml:37`
- `.github/workflows/test.yml:38`
- `.github/workflows/test.yml:44`
- `.github/workflows/test.yml:51`
- `.github/workflows/test.yml:52`
- `.github/workflows/update-permission-inputs.yml:23`
- `.github/workflows/update-permission-inputs.yml:24`

### script-injection (severity: high)

GitHub Actions expression values are interpolated directly inside run: shell commands, violating rule (a). This allows an attacker to inject arbitrary shell commands if they can control the expression value.

- test.yml: `run: echo '${{ steps.get-repository.outputs.data }}'` — `steps.*.outputs.*` is a workflow-controllable context interpolated directly into a shell command.
- test.yml: `run: test "${{ steps.test.outcome }}" = "failure"` — `steps.*.outputs.*` context interpolated directly into a shell command.
- update-permission-inputs.yml: `gh pr edit ${{ github.event.pull_request.number }} --title "${{ env.COMMIT_MESSAGE }}"` — `github.*` and `env.*` contexts interpolated directly into a shell command. The `github.event.pull_request.number` value is attacker-controllable via pull request events.

Locations:

- `.github/workflows/test.yml:43`
- `.github/workflows/test.yml:60`
- `.github/workflows/update-permission-inputs.yml:38`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all unpinned action references by resolving them to full 40-character SHA digests with tag comments preserved: actions/checkout@v6→d23441a4, actions/publish-immutable-action@v0.0.4→4bc8754f, actions/setup-node@v6→24997072, actions/stale@v10→1e223db2, octokit/request-action@v2.x→02f5e7c6. Fixed three script injection vulnerabilities by moving ${{ }} expressions into step env: blocks and referencing them as plain shell variables: steps.get-repository.outputs.data→$REPOSITORY_DATA, steps.test.outcome→$TEST_OUTCOME, github.event.pull_request.number→$PR_NUMBER, env.COMMIT_MESSAGE→$PR_TITLE.

