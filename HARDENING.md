<!-- markdownlint-disable -->

# Hardening Report: ScribeMD--rootless-docker/0.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ScribeMD--rootless-docker/0.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unsafe-shell (severity: high)

The 'Install rootless Docker' step downloads a remote script and pipes it straight into a shell: `curl --fail --silent --show-error --location https://get.docker.com/rootless | sh`. The script is not saved to a file or checked before it runs.

Locations:

- `action.yaml:48`

### script-injection (severity: high)

Rule (b): an inherited, workflow-controlled environment variable is expanded without quotes in the 'Install rootless Docker' step: `if [[ -z $XDG_RUNTIME_DIR ]]; then`. Inside bash `[[ ]]` the risk is minimal, but the check requires double quotes (`"$XDG_RUNTIME_DIR"`).

Locations:

- `action.yaml:35`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yaml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yaml b/action.yaml
index e78e8f9..75de4a2 100644
--- a/action.yaml
+++ b/action.yaml
@@ -7,6 +7,23 @@ branding:
 runs:
   using: composite
   steps:
     - name: Check whether rootless Docker is already installed and/or in-use.
       id: rootless-docker
       run: |
@@ -45,12 +62,14 @@ runs:
           return 1
         }
 
-        install_script_output="$(curl \
+        curl \
           --fail \
           --silent \
           --show-error \
-          --location https://get.docker.com/rootless |
-          sh)"
+          --location \
+          --output /tmp/get-docker-rootless.sh \
+          https://get.docker.com/rootless
+        install_script_output="$(sh /tmp/get-docker-rootless.sh)"
         echo "$install_script_output"
         docker context use rootless
         if grep --quiet "systemd not detected, dockerd-rootless.sh needs to be started manually" <<<"$install_script_output"; then
@@ -63,12 +82,6 @@ runs:
     - name: Proxy bidirectionally between rootful and rootless Docker sockets.
       if: steps.rootless-docker.outputs.in-use != 'true'
       run: >
-        sudo systemd-run
-        --unit=docker-proxy.service
-        --description="Bidirectional proxy between rootful and rootless Docker sockets"
-        --service-type=exec
-        --property=Requires=docker.socket
-        --property=PrivateNetwork=true
-        --property=PrivateTmp=true
-        /lib/systemd/systemd-socket-proxyd "$XDG_RUNTIME_DIR/docker.sock"
+        sudo systemd-run --unit=docker-proxy.service --description="Bidirectional proxy between rootful and rootless Docker sockets" --service-type=exec --property=Requires=docker.socket --property=PrivateNetwork=true --property=PrivateTmp=true /lib/systemd/systemd-socket-proxyd "$XDG_RUNTIME_DIR/docker.sock"
+
       shell: bash

```

Locations:

- `action.yaml`

## Iteration Notes

### Iteration 1

**Fixes applied:** unsafe-shell, script-injection, prior-fix

**Notes:**

I fixed all three findings in action.yaml.

- **unsafe-shell:** the get.docker.com/rootless installer is no longer piped into sh. It is now downloaded to a mktemp file with curl --fail and checked to be non-empty, then run with `sh "$file"`. Its output is still captured for the existing systemd grep. The step's composite bash runs with errexit and pipefail, so a failed curl still stops the step.
- **script-injection:** `$XDG_RUNTIME_DIR` inside `[[ -z ]]` is now double-quoted.
- **prior-fix:** the earlier harden's real security change was the same download-then-run rewrite, and it is re-applied by the unsafe-shell fix above. That patch also squashed the folded (`>`) systemd-run command onto one line. I didn't redo that: a folded block already joins its lines with spaces, so the command is the same either way and nothing security-related changes.

