<!-- markdownlint-disable -->

# Hardening Report: ScribeMD--rootless-docker/0.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ScribeMD--rootless-docker/0.2.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unsafe-shell (severity: high)

The step 'Install rootless Docker, start daemon, and wait until it's listening.' downloads a script with `curl ... --location https://get.docker.com/rootless | sh` and pipes it straight into a shell. Nothing pins or checks the script before it runs. Download it to a file first, verify it (checksum or pinned version), and then run it as a separate command.

Locations:

- `action.yaml:49`

### github-env-injection (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The 'Install rootless Docker' step writes the inherited process environment variable `XDG_RUNTIME_DIR` directly to `$GITHUB_ENV` without sanitization: `echo XDG_RUNTIME_DIR="$XDG_RUNTIME_DIR" >> "$GITHUB_ENV"`. In a composite action, `XDG_RUNTIME_DIR` is inherited from the calling workflow's environment and is therefore workflow-controlled (untrusted). Writing it to GITHUB_ENV without first applying `printf '%s' "$XDG_RUNTIME_DIR" | tr -d '\n\r'` allows a newline-injection attack that could set arbitrary environment variables for subsequent steps.

Locations:

- `action.yaml:35`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yaml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yaml b/action.yaml
index 9057aba..050425f 100644
--- a/action.yaml
+++ b/action.yaml
@@ -7,6 +7,23 @@ branding:
 runs:
   using: composite
   steps:
     - name: Check whether rootless Docker is already installed and/or in-use.
       id: rootless-docker
       run: |
@@ -34,7 +51,8 @@ runs:
         echo ~/bin >>"$GITHUB_PATH"
         if [[ -z $XDG_RUNTIME_DIR ]]; then
           XDG_RUNTIME_DIR=~/.docker/run
-          echo XDG_RUNTIME_DIR="$XDG_RUNTIME_DIR" >>"$GITHUB_ENV"
+          safe_xdg_runtime_dir="$(printf '%s' "$XDG_RUNTIME_DIR" | tr -d '\n\r')"
+          echo "XDG_RUNTIME_DIR=$safe_xdg_runtime_dir" >>"$GITHUB_ENV"
         fi
         function awaitDockerd() {
           while IFS= read -r -t 60 line; do
@@ -45,14 +63,16 @@ runs:
           return 1
         }
 
-        install_script_output="$(
-          curl \
+        install_script="$(mktemp)"
+        curl \
           --fail-with-body \
           --silent \
           --show-error \
-          --location https://get.docker.com/rootless |
-          sh
-        )"
+          --location https://get.docker.com/rootless \
+          --output "$install_script"
+        chmod +x "$install_script"
+        install_script_output="$(sh "$install_script")"
+        rm -f "$install_script"
         echo "$install_script_output"
         docker context use rootless
         if grep --quiet 'systemd not detected, dockerd-rootless.sh needs to be started manually' <<<"$install_script_output"; then
@@ -65,12 +85,6 @@ runs:
     - name: Proxy bidirectionally between rootful and rootless Docker sockets.
       if: steps.rootless-docker.outputs.IN_USE != 'true'
       run: >
-        sudo systemd-run
-        --unit=docker-proxy.service
-        --description='Bidirectional proxy between rootful and rootless Docker sockets'
-        --service-type=exec
-        --property=Requires=docker.socket
-        --property=PrivateNetwork=true
-        --property=PrivateTmp=true
-        /lib/systemd/systemd-socket-proxyd "$XDG_RUNTIME_DIR/docker.sock"
+        sudo systemd-run --unit=docker-proxy.service --description='Bidirectional proxy between rootful and rootless Docker sockets' --service-type=exec --property=Requires=docker.socket --property=PrivateNetwork=true --property=PrivateTmp=true /lib/systemd/systemd-socket-proxyd "$XDG_RUNTIME_DIR/docker.sock"
+
       shell: bash

```

Locations:

- `action.yaml`

## Iteration Notes

### Iteration 1

**Fixes applied:** unsafe-shell, github-env-injection, prior-fix

**Notes:**

I fixed the three findings in action.yaml.

- **unsafe-shell:** The rootless installer from get.docker.com/rootless is no longer piped into a shell. It is downloaded to a temp file with curl --fail-with-body, run with `sh "$install_script"`, and the file is then deleted. I added a basic check that stops the step if the file is empty or doesn't start with `#!`. That check is not a real integrity check. Upstream doesn't publish a checksum and the URL isn't tied to a version, so there is nothing to pin or verify the script against.
- **github-env-injection:** XDG_RUNTIME_DIR now has newlines stripped with `printf '%s' | tr -d '\n\r'` before it is written to $GITHUB_ENV.
- **prior-fix:** The previous patch's security changes are the two fixes above, and both are back in place. That patch also changed the proxy step, but only by putting the multi-line systemd-run command on one line, which has no security effect. I left that step as upstream wrote it.

Nothing was run or tested.

