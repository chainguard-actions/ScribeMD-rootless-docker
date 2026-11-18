<!-- markdownlint-disable -->

# Hardening Report: ScribeMD--rootless-docker/0.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ScribeMD--rootless-docker/0.2.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unsafe-shell (severity: high)

The 'Install rootless Docker' step downloads a remote script and pipes it straight into a shell: `curl --fail --silent --show-error --location https://get.docker.com/rootless | sh`. The script is never saved to a file or checked against a pinned checksum before it runs.

Locations:

- `action.yaml:48`

### script-injection (severity: high)

Rule (b): the 'Install rootless Docker' step expands the inherited env var XDG_RUNTIME_DIR without quotes: `if [[ -z $XDG_RUNTIME_DIR ]]; then`. The calling workflow controls this variable. It should be quoted as "$XDG_RUNTIME_DIR".

Locations:

- `action.yaml:35`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yaml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yaml b/action.yaml
index 0bc2e03..59c626a 100644
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
+          --output /tmp/rootless-docker-install.sh \
+          https://get.docker.com/rootless
+        install_script_output="$(sh /tmp/rootless-docker-install.sh)"
         echo "$install_script_output"
         docker context use rootless
         if grep --quiet 'systemd not detected, dockerd-rootless.sh needs to be started manually' <<<"$install_script_output"; then
@@ -63,12 +82,6 @@ runs:
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

**Fixes applied:** unsafe-shell, script-injection, prior-fix

**Notes:**

I changed action.yaml in two places.

- **unsafe-shell:** the install step no longer pipes `curl https://get.docker.com/rootless` into `sh`. It now saves the script to a `mktemp` file, stops if the file is empty, runs it with `sh "$install_script"`, and deletes it afterwards. `install_script_output` still captures the script's output, so the check for 'systemd not detected' works as before. This is not checked against a pinned checksum, which the finding mentions: get.docker.com/rootless changes over time, so a fixed hash would break the action whenever Docker updates the script. If you want that guarantee, the alternative is to bundle a reviewed copy of the script with the action.
- **script-injection:** `[[ -z $XDG_RUNTIME_DIR ]]` is now `[[ -z "$XDG_RUNTIME_DIR" ]]`. Inside bash's `[[ ]]` the variable isn't word-split, so the unquoted form wasn't actually exploitable; the change just satisfies the rule.
- **prior-fix:** the security part of the earlier patch was this same download-then-run change, so it's covered above. That patch also squashed the proxy step's `systemd-run` command onto one line. That was formatting only, so I didn't repeat it. Its diff also showed 17 lines added near the top of the file without their content, so I couldn't redo those.

I haven't run the action to test these changes.

