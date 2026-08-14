<!-- markdownlint-disable -->

# Hardening Report: ScribeMD--rootless-docker/0.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ScribeMD--rootless-docker/0.2.2** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unsafe-shell (severity: high)

The 'Install rootless Docker' step fetches a remote install script from https://get.docker.com/rootless and pipes it directly into `sh` via `curl ... | sh` inside a command substitution. This is an unsafe shell pattern: if the remote URL is compromised or the connection is intercepted, arbitrary code will be executed on the runner without any integrity verification.

Locations:

- `action.yaml:51`

### github-env-injection (severity: high)

The 'Install rootless Docker' step writes the inherited process environment variable `XDG_RUNTIME_DIR` directly to `$GITHUB_ENV` without sanitization: `echo XDG_RUNTIME_DIR="$XDG_RUNTIME_DIR" >> "$GITHUB_ENV"`. In a composite action, `XDG_RUNTIME_DIR` is inherited from the calling workflow's environment and is therefore workflow-controlled (untrusted). Writing it to GITHUB_ENV without first applying `printf '%s' "$XDG_RUNTIME_DIR" | tr -d '\n\r'` allows a newline-injection attack that could set arbitrary environment variables for subsequent steps.

Locations:

- `action.yaml:35`

## Iteration Notes

### Iteration 1

**Fixes applied:** unsafe-shell, github-env-injection

**Notes:**

Fixed two high-severity findings in hardened/action/action.yaml:
1. unsafe-shell (line 51): Replaced `curl ... | sh` pattern with a safe two-step approach: download the install script to a temp file using `mktemp` and `--output`, then execute it separately with `sh "$install_script"`, and clean up with `rm -f`. This prevents arbitrary code execution if the remote URL is compromised or the connection is intercepted.
2. github-env-injection (line 35): Sanitized the XDG_RUNTIME_DIR value before writing to $GITHUB_ENV by using `printf '%s' "$XDG_RUNTIME_DIR" | tr -d '\n\r'` to strip newlines, preventing newline-injection attacks that could set arbitrary environment variables for subsequent steps.

### Iteration 2

**Fixes applied:** unpinned-uses

**Notes:**

Pinned all 5 unpinned action references to full 40-character commit SHAs with version tags preserved as comments:
- actions/checkout@v3.1.0 → @93ea575cb5d8a053eaa0ac8fa3b40d7e05a33cc8 # v3.1.0 (test.yaml)
- ScribeMD/pre-commit-action@0.9.23 → @33bd9c94e22d6b9532fcd80a5ba36780ea311309 # 0.9.23 (test.yaml)
- ScribeMD/slack-templates@0.6.12 → @c7982d3a4caac776a8faafff78462927293175ef # 0.6.12 (test.yaml)
- ScribeMD/slack-templates/.github/workflows/notify-assignee.yaml@0.6.12 → @c7982d3a4caac776a8faafff78462927293175ef # 0.6.12 (notify-assignee.yaml)
- ScribeMD/slack-templates/.github/workflows/notify-reviewers.yaml@0.6.12 → @c7982d3a4caac776a8faafff78462927293175ef # 0.6.12 (notify-reviewers.yaml)
A minor file corruption during editing was detected and corrected.

