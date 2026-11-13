<!-- markdownlint-disable -->

# Hardening Report: ika-rwth-aachen--docker-ros/v1.10.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ika-rwth-aachen--docker-ros/v1.10.0** was hardened automatically. 7 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml has composite steps whose uses: refs are tags or branches, not full commit SHAs: actions/checkout@v3, docker/setup-qemu-action@v2, docker/login-action@v3, docker/setup-buildx-action@v3, ASzc/change-string-case-action@v6 (x3), ros-industrial/industrial_ci@master, gacts/github-slug@v1.

Locations:

- `action.yml:173`
- `action.yml:196`
- `action.yml:199`
- `action.yml:206`
- `action.yml:210`
- `action.yml:216`
- `action.yml:222`
- `action.yml:283`
- `action.yml:298`

### script-injection (severity: high)

Rule (a) is violated in the 'Set up industrial_ci' step: the run: script puts a GitHub Actions expression directly into the shell command: `run: test -f ${{ inputs.build-context }}/.repos || echo "repositories:" > ${{ inputs.build-context }}/.repos`. The inputs.build-context value is substituted into the script without quoting, so shell metacharacters in it would run. Fix: pass it through env (e.g. BUILD_CONTEXT) and use "$BUILD_CONTEXT/.repos" with double quotes.

Locations:

- `action.yml:279`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.build-context }}" appears directly in run: block of step "Set up industrial_ci"; move to env: map

Locations:

- `action.yml:279`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.build-context }}" appears directly in run: block of step "Set up industrial_ci"; move to env: map

Locations:

- `action.yml:279`

### github-env-injection (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. In scripts/ci.sh, the value `INDUSTRIAL_CI_IMAGE` is written to `$GITHUB_OUTPUT` without sanitization. The value is composed from `${IMAGE}`, `${DEV_IMAGE}`, `${_IMAGE_POSTFIX}`, and `${PLATFORM}` — all of which are inherited from the calling workflow's environment (set via `env:` in action.yml from `inputs.*` and `github.*` values such as `inputs.image-name`, `inputs.image-tag`, `inputs.platform`, and `github.ref_name`). A newline injected into any of these values could allow an attacker to write arbitrary key=value pairs to `$GITHUB_OUTPUT`, potentially overwriting subsequent step outputs. The write `echo "INDUSTRIAL_CI_IMAGE=${industrial_ci_image}" >> "${GITHUB_OUTPUT}"` is not preceded by the required `printf '%s' ... | tr -d '\n\r'` sanitization.

Locations:

- `scripts/ci.sh:76`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 1b0e3e7..3c7a3da 100644
--- a/action.yml
+++ b/action.yml
@@ -3,7 +3,6 @@ description: "docker-ros automatically builds development and deployment Docker
 branding:
   color: blue
   icon: package
-
 inputs:
   additional-debs-file:
     description: "Relative filepath to file containing additional apt deb packages to install"
@@ -136,25 +135,7 @@ inputs:
   slim-build-args:
     description: "Arguments to `mint slim` (except for `--target` and `--tag`)"
     default: >-
-      --sensor-ipc-mode proxy
-      --continue-after=30
-      --show-clogs
-      --http-probe=false
-      --env DOCKER_UID=23456
-      --env DOCKER_GID=23456
-      --env DOCKER_USER=dockerslimprobe
-      --include-path /opt/ros
-      --include-path /docker-ros/ws/install
-      --include-path /etc/ld.so.cache
-      --include-path /etc/ld.so.conf
-      --include-path /etc/ld.so.conf.d
-      --preserve-path /etc/passwd
-      --preserve-path /etc/group
-      --preserve-path /etc/shadow
-      --preserve-path /etc/gshadow
-      --exclude-pattern /home/dockerslimprobe
-      --exclude-pattern /home/dockerslimprobe/**
-      --exclude-pattern /var/mail/dockerslimprobe
+      --sensor-ipc-mode proxy --continue-after=30 --show-clogs --http-probe=false --env DOCKER_UID=23456 --env DOCKER_GID=23456 --env DOCKER_USER=dockerslimprobe --include-path /opt/ros --include-path /docker-ros/ws/install --include-path /etc/ld.so.cache --include-path /etc/ld.so.conf --include-path /etc/ld.so.conf.d --preserve-path /etc/passwd --preserve-path /etc/group --preserve-path /etc/shadow --preserve-path /etc/gshadow --exclude-pattern /home/dockerslimprobe --exclude-pattern /home/dockerslimprobe/** --exclude-pattern /var/mail/dockerslimprobe
   slim-image-name:
     description: "Image name of slim run image"
     default: ghcr.io/${{ github.repository }}
@@ -163,65 +144,70 @@ inputs:
   target:
     description: "Target stage of Dockerfile (comma-separated list) [dev|run]"
     default: run
-
-
 runs:
   using: "composite"
   steps:
-
     - name: Checkout repository
       uses: actions/checkout@v3
       if: ${{ inputs.enable-checkout == 'true' }}
       with:
         submodules: ${{ inputs.enable-checkout-submodules }}
         lfs: ${{ inputs.enable-checkout-lfs }}
-
     - name: Set up docker-ros
       shell: bash
       working-directory: ${{ inputs.build-context }}
       run: |
-            if ! [[ -d "docker/docker-ros" ]]; then
-              mkdir -p docker
-              cp -r ${GITHUB_ACTION_PATH} docker/docker-ros
-            fi
-
+        if ! [[ -d "docker/docker-ros" ]]; then
+          mkdir -p docker
+          cp -r ${GITHUB_ACTION_PATH} docker/docker-ros
+        fi
     - name: Prepare setup of QEMU
       id: prepare-setup-of-qemu
       shell: bash
       run: echo "RUNNER_ARCH=$(dpkg --print-architecture)" >> $GITHUB_OUTPUT
-
     - name: Set up QEMU
       if: ${{ steps.prepare-setup-of-qemu.outputs.RUNNER_ARCH != inputs.platform }}
       uses: docker/setup-qemu-action@v2
-
     - name: Login to Docker registry
       uses: docker/login-action@v3
       with:
         registry: ${{ inputs.registry }}
         username: ${{ inputs.registry-user }}
         password: ${{ inputs.registry-password }}
-
     - name: Set up Docker buildx
       uses: docker/setup-buildx-action@v3
-
-    - name: Enforce lower-case image name 
+    - name: Enforce lower-case image name
       id: image-name
       uses: ASzc/change-string-case-action@v6
       with:
         string: ${{ inputs.image-name }}
-
-    - name: Enforce lower-case dev image name 
+    - name: Enforce lower-case dev image name
       id: dev-image-name
       uses: ASzc/change-string-case-action@v6
       with:
         string: ${{ inputs.dev-image-name }}
-
-    - name: Enforce lower-case slim image name 
+    - name: Enforce lower-case slim image name
       id: slim-image-name
       uses: ASzc/change-string-case-action@v6
       with:
         string: ${{ inputs.slim-image-name }}
-
     - name: Build images
       id: build-images
       she
... (truncated)
```

Locations:

- `action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed scripts/ci.sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/scripts/ci.sh b/scripts/ci.sh
index 6c91786..b0abbea 100755
--- a/scripts/ci.sh
+++ b/scripts/ci.sh
@@ -81,7 +81,8 @@ if [[ -n "${GITHUB_ACTIONS}" ]]; then
     else
         industrial_ci_image="${industrial_ci_image}-$(dpkg --print-architecture)"
     fi
-    echo "INDUSTRIAL_CI_IMAGE=${industrial_ci_image}" >> "${GITHUB_OUTPUT}"
+    safe_industrial_ci_image=$(printf '%s' "${industrial_ci_image}" | tr -d '\n\r')
+    echo "INDUSTRIAL_CI_IMAGE=${safe_industrial_ci_image}" >> "${GITHUB_OUTPUT}"
 fi
 
 # parse (potentially) comma-separated lists to arrays
@@ -126,7 +127,17 @@ for PLATFORM in "${PLATFORMS[@]}"; do
         cd dist_linux*
         export DOCKER_API_VERSION="${DOCKER_API_VERSION:-$(docker version --format '{{.Server.APIVersion}}')}"
         docker pull "${image}"
-        ./mint slim --target "${image}" --tag "${slim_image}" ${SLIM_BUILD_ARGS} ${ADDITIONAL_SLIM_BUILD_ARGS}
+        slim_args=()
+        if [ -n "${SLIM_BUILD_ARGS}" ]; then
+            while IFS= read -r -d '' t; do slim_args+=("$t"); done \
+                < <(printf '%s' "${SLIM_BUILD_ARGS}" | xargs printf '%s\0')
+        fi
+        additional_slim_args=()
+        if [ -n "${ADDITIONAL_SLIM_BUILD_ARGS}" ]; then
+            while IFS= read -r -d '' t; do additional_slim_args+=("$t"); done \
+                < <(printf '%s' "${ADDITIONAL_SLIM_BUILD_ARGS}" | xargs printf '%s\0')
+        fi
+        ./mint slim --target "${image}" --tag "${slim_image}" "${slim_args[@]}" "${additional_slim_args[@]}"
         docker push "${slim_image}"
         cd -
         rm -rf dist_linux* ds.tar.gz

```

Locations:

- `scripts/ci.sh`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection, github-env-injection, prior-fix

**Notes:**

Pinned every composite uses: to a full commit SHA, found with lookup_action_sha, and kept the tag as a trailing comment. Moved inputs.build-context in the 'Set up industrial_ci' step into env BUILD_CONTEXT and quoted it as "$BUILD_CONTEXT/.repos". In scripts/ci.sh, INDUSTRIAL_CI_IMAGE now has newlines stripped (printf | tr -d '\n\r') before it is written to GITHUB_OUTPUT. These are the security parts of the earlier harden, re-applied. Two things from that harden were deliberately left out. First, rewriting the mint slim args with xargs: upstream expands ${SLIM_BUILD_ARGS} and ${ADDITIONAL_SLIM_BUILD_ARGS} unquoted. That only splits on whitespace and expands globs, and runs nothing in the value, so it is not injectable. An xargs split would also change behavior: it would treat quotes as syntax and turn off glob expansion. Upstream's unquoted expansion stays. Second, collapsing the slim-build-args default onto one line: with the unquoted expansion kept, splitting on whitespace gives the same arguments either way, so the change wasn't needed. The other differences in that patch were blank lines, trailing spaces and indentation.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Quoted ${GITHUB_ACTION_PATH} in the 'Set up docker-ros' step of action.yml: cp -r "${GITHUB_ACTION_PATH}" docker/docker-ros.

