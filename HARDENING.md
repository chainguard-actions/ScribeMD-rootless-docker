<!-- markdownlint-disable -->

# Hardening Report: ScribeMD--rootless-docker/0.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ScribeMD--rootless-docker/0.2.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unsafe-shell (severity: high)

In action.yaml, the 'Install rootless Docker' step fetches a remote install script from https://get.docker.com/rootless and pipes it directly to `sh` via `curl ... | sh`. This allows a compromised or malicious remote server to execute arbitrary code on the runner without any integrity verification.

Locations:

- `action.yaml:52`

### unpinned-uses (severity: high)

All `uses:` references in the workflow files use mutable version tags instead of immutable full 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the referenced tags are moved or the upstream repositories are compromised. Unpinned references: `actions/checkout@v3.1.0` (test.yaml line 17), `ScribeMD/pre-commit-action@0.9.16` (test.yaml line 23), `ScribeMD/slack-templates@0.6.11` (test.yaml line 26), `ScribeMD/slack-templates/.github/workflows/notify-assignee.yaml@0.6.7` (notify-assignee.yaml line 9), `ScribeMD/slack-templates/.github/workflows/notify-reviewers.yaml@0.6.7` (notify-reviewers.yaml line 9).

Locations:

- `.github/workflows/test.yaml:17`
- `.github/workflows/test.yaml:23`
- `.github/workflows/test.yaml:26`
- `.github/workflows/notify-assignee.yaml:9`
- `.github/workflows/notify-reviewers.yaml:9`

## Iteration Notes

### Iteration 1

**Fixes applied:** unsafe-shell, unpinned-uses

**Notes:**

Fixed 2 findings: (1) unsafe-shell in action.yaml line 52: replaced `curl ... | sh` pipe with a two-step approach that downloads the script to /tmp/rootless-docker-install.sh first, then executes it separately. (2) unpinned-uses: pinned all 5 mutable tag references to full 40-character commit SHAs in test.yaml (actions/checkout@v3.1.0→93ea575..., ScribeMD/pre-commit-action@0.9.16→2577ac8..., ScribeMD/slack-templates@0.6.11→3c4515e...) and in notify-assignee.yaml and notify-reviewers.yaml (ScribeMD/slack-templates@0.6.7→501ba2b...). Original tags preserved as inline comments for readability.

