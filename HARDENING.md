<!-- markdownlint-disable -->

# Hardening Report: ScribeMD--rootless-docker/0.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **ScribeMD--rootless-docker/0.2.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unsafe-shell (severity: high)

The 'Install rootless Docker' step downloads a remote shell script from https://get.docker.com/rootless and pipes it directly to `sh` without first saving it to a file for inspection. This pattern (`curl ... | sh`) allows arbitrary code execution if the remote URL is compromised or tampered with in transit (e.g., via a MITM attack or supply-chain compromise of get.docker.com). The offending line is: `--location https://get.docker.com/rootless | sh`

Locations:

- `action.yaml:47`

## Iteration Notes

### Iteration 1

**Fixes applied:** unsafe-shell

**Notes:**

Fixed the unsafe curl | sh pattern in the 'Install rootless Docker' step of action.yaml. The script at https://get.docker.com/rootless is now downloaded to a temporary file via curl --output before being executed with sh, rather than being piped directly to sh. The temporary file is cleaned up after execution. This prevents arbitrary code execution if the remote URL is compromised or tampered with in transit.

