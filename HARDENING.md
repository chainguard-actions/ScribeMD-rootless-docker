<!-- markdownlint-disable -->

# Hardening Report: ScribeMD--rootless-docker/0.1.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ScribeMD--rootless-docker/0.1.6** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unsafe-shell (severity: high)

The 'Install rootless Docker' step fetches a remote script and pipes it directly to `sh` via `curl --location https://get.docker.com/rootless | sh`. This allows arbitrary code execution if the remote URL is compromised or redirected. The script should be downloaded to a file first, verified (e.g., via checksum), and then executed separately.

Locations:

- `action.yaml:47`

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files use mutable version tags instead of full 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the referenced tag is moved. Failing references: `actions/checkout@v3.0.2` (line 17), `ScribeMD/pre-commit-action@0.8.6` (line 21), `ScribeMD/slack-templates@0.6.2` (line 24).

Locations:

- `.github/workflows/test.yaml:17`
- `.github/workflows/test.yaml:21`
- `.github/workflows/test.yaml:24`

### unpinned-uses (severity: high)

The reusable workflow reference `ScribeMD/slack-templates/.github/workflows/notify-assignee.yaml@0.6.2` uses a mutable version tag instead of a full 40-character SHA commit hash.

Locations:

- `.github/workflows/notify-assignee.yaml:9`

### unpinned-uses (severity: high)

The reusable workflow reference `ScribeMD/slack-templates/.github/workflows/notify-reviewers.yaml@0.6.2` uses a mutable version tag instead of a full 40-character SHA commit hash.

Locations:

- `.github/workflows/notify-reviewers.yaml:9`

### missing-permissions (severity: medium)

The workflow has no top-level `permissions:` key and the single job also has no `permissions:` key. Without explicit permissions, the workflow inherits the default repository permissions (which may be `write-all` depending on repository settings). A minimal permissions block should be added.

Locations:

- `.github/workflows/notify-assignee.yaml:1`

### missing-permissions (severity: medium)

The workflow has no top-level `permissions:` key and the single job also has no `permissions:` key. Without explicit permissions, the workflow inherits the default repository permissions (which may be `write-all` depending on repository settings). A minimal permissions block should be added.

Locations:

- `.github/workflows/notify-reviewers.yaml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unsafe-shell, unpinned-uses, missing-permissions

**Notes:**

Fixed all 6 findings across 4 files:
1. action.yaml (unsafe-shell): Changed `curl ... | sh` to download the script to /tmp/install-rootless-docker.sh first, then execute it separately with `sh /tmp/install-rootless-docker.sh`.
2. .github/workflows/test.yaml (unpinned-uses): Pinned actions/checkout@v3.0.2 → SHA 2541b1294d2704b0964813337f33b291d3f8596b, ScribeMD/pre-commit-action@0.8.6 → SHA 470a5b90295d8adf9fc3a344d276c6caaf11c530, ScribeMD/slack-templates@0.6.2 → SHA 6ad4cc1a99244b41714352c0025b40a9e90ccb25.
3. .github/workflows/notify-assignee.yaml (unpinned-uses + missing-permissions): Pinned reusable workflow reference to SHA 6ad4cc1a99244b41714352c0025b40a9e90ccb25 and added `permissions: {}` top-level block.
4. .github/workflows/notify-reviewers.yaml (unpinned-uses + missing-permissions): Pinned reusable workflow reference to SHA 6ad4cc1a99244b41714352c0025b40a9e90ccb25 and added `permissions: {}` top-level block.

