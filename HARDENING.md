<!-- markdownlint-disable -->

# Hardening Report: ScribeMD--rootless-docker/0.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ScribeMD--rootless-docker/0.2.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unsafe-shell (severity: high)

In action.yaml, the 'Install rootless Docker' step downloads and executes a remote shell script by piping curl output directly to sh: `curl --fail --silent --show-error --location https://get.docker.com/rootless | sh`. This is unsafe because the remote content is executed without first being inspected or verified, allowing a compromised or malicious server to execute arbitrary code on the runner.

Locations:

- `action.yaml:46`

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable version tags instead of immutable 40-character commit SHAs. This exposes the workflow to supply-chain attacks if a tag is moved or a repository is compromised. Failing references:
- test.yaml: `actions/checkout@v3.0.2`, `ScribeMD/pre-commit-action@0.9.7`, `ScribeMD/slack-templates@0.6.8`
- notify-assignee.yaml: `ScribeMD/slack-templates/.github/workflows/notify-assignee.yaml@0.6.7`
- notify-reviewers.yaml: `ScribeMD/slack-templates/.github/workflows/notify-reviewers.yaml@0.6.7`

Locations:

- `.github/workflows/test.yaml:14`
- `.github/workflows/test.yaml:18`
- `.github/workflows/test.yaml:21`
- `.github/workflows/notify-assignee.yaml:9`
- `.github/workflows/notify-reviewers.yaml:9`

## Iteration Notes

### Iteration 1

**Fixes applied:** unsafe-shell, unpinned-uses

**Notes:**

Fixed two findings: (1) unsafe-shell in action.yaml: replaced `curl ... | sh` with a two-step approach that downloads the script to /tmp/get-docker-rootless.sh first, then executes it separately with `sh /tmp/get-docker-rootless.sh`. (2) unpinned-uses: pinned all five mutable tag references to full 40-character commit SHAs — actions/checkout@v3.0.2→2541b1294d2704b0964813337f33b291d3f8596b, ScribeMD/pre-commit-action@0.9.7→9deb7b526d077f1be37d9b0e345189343311b045, ScribeMD/slack-templates@0.6.8→0c6d199238616660e80fc05b138a6b5b1d3ec92b, and both notify-assignee/notify-reviewers ScribeMD/slack-templates@0.6.7→501ba2b32c9337d251cb4f486fc95122fa4832dc. Original tags preserved as inline comments.

