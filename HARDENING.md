<!-- markdownlint-disable -->

# Hardening Report: ScribeMD--rootless-docker/0.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **ScribeMD--rootless-docker/0.2.2** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unsafe-shell (severity: high)

The 'Install rootless Docker' step downloads a remote shell script from https://get.docker.com/rootless and pipes it directly to `sh` without first saving it to a file for inspection. This pattern (`curl ... | sh`) is unsafe because it executes arbitrary remote content immediately, with no opportunity to verify integrity or content before execution. If the remote URL is compromised or subject to a MITM attack, malicious code would execute on the runner.

Locations:

- `action.yaml:51`

## Iteration Notes

### Iteration 1

**Fixes applied:** unsafe-shell

**Notes:**

Fixed the unsafe `curl | sh` pattern in the 'Install rootless Docker' step of action.yaml. The remote script from https://get.docker.com/rootless is now downloaded to a temporary file via `mktemp` and `curl --output`, made executable with `chmod +x`, then executed separately as `sh "$install_script"`, and cleaned up with `rm -f`. This prevents immediate execution of arbitrary remote content and gives an opportunity to inspect the script before running it, mitigating MITM and supply-chain attack risks.

