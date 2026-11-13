<!-- markdownlint-disable -->

# Hardening Report: ika-rwth-aachen--docker-ros/v1.9.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ika-rwth-aachen--docker-ros/v1.9.0** was hardened automatically. 7 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Composite action steps in action.yml point to mutable tags or branches instead of 40-character commit SHAs: actions/checkout@v3, docker/setup-qemu-action@v2, docker/login-action@v3, docker/setup-buildx-action@v3, ASzc/change-string-case-action@v6 (3 times), ros-industrial/industrial_ci@master and gacts/github-slug@v1.

Locations:

- `action.yml:161`
- `action.yml:183`
- `action.yml:186`
- `action.yml:193`
- `action.yml:197`
- `action.yml:203`
- `action.yml:209`
- `action.yml:264`
- `action.yml:279`

### script-injection (severity: high)

Rule (a): the 'Set up industrial_ci' step puts the expression ${{ inputs.build-context }} directly into the run script: `run: test -f ${{ inputs.build-context }}/.repos || echo "repositories:" > ${{ inputs.build-context }}/.repos`. Pass the value through an env var such as BUILD_CONTEXT and reference it as "$BUILD_CONTEXT".

Locations:

- `action.yml:260`

### github-env-injection (severity: high)

The 'Build images' step runs scripts/ci.sh, which writes `echo "INDUSTRIAL_CI_IMAGE=${industrial_ci_image}" >> "${GITHUB_OUTPUT}"`. The value is built from IMAGE_NAME/IMAGE_TAG/DEV_IMAGE_TAG/PLATFORM/_IMAGE_POSTFIX, which come from inputs.* and steps.*.outputs. Newlines are not stripped (no printf '%s' ... | tr -d '\n\r') before the write, so a newline in an input could inject extra output entries.

Locations:

- `scripts/ci.sh:85`
- `action.yml:217`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.build-context }}" appears directly in run: block of step "Set up industrial_ci"; move to env: map

Locations:

- `action.yml:261`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.build-context }}" appears directly in run: block of step "Set up industrial_ci"; move to env: map

Locations:

- `action.yml:261`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index d832496..fda4251 100644
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
@@ -124,25 +123,7 @@ inputs:
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
@@ -151,65 +132,70 @@ inputs:
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
index 6c91786..9a1bdce 100755
--- a/scripts/ci.sh
+++ b/scripts/ci.sh
@@ -81,7 +81,8 @@ if [[ -n "${GITHUB_ACTIONS}" ]]; then
     else
         industrial_ci_image="${industrial_ci_image}-$(dpkg --print-architecture)"
     fi
-    echo "INDUSTRIAL_CI_IMAGE=${industrial_ci_image}" >> "${GITHUB_OUTPUT}"
+    safe_industrial_ci_image="$(printf '%s' "${industrial_ci_image}" | tr -d '\n\r')"
+    echo "INDUSTRIAL_CI_IMAGE=${safe_industrial_ci_image}" >> "${GITHUB_OUTPUT}"
 fi
 
 # parse (potentially) comma-separated lists to arrays
@@ -126,7 +127,16 @@ for PLATFORM in "${PLATFORMS[@]}"; do
         cd dist_linux*
         export DOCKER_API_VERSION="${DOCKER_API_VERSION:-$(docker version --format '{{.Server.APIVersion}}')}"
         docker pull "${image}"
-        ./mint slim --target "${image}" --tag "${slim_image}" ${SLIM_BUILD_ARGS} ${ADDITIONAL_SLIM_BUILD_ARGS}
+        slim_args=()
+        if [[ -n "${SLIM_BUILD_ARGS}" ]]; then
+            while IFS= read -r -d '' t; do slim_args+=("$t"); done \
+                < <(printf '%s' "${SLIM_BUILD_ARGS}" | xargs printf '%s\0')
+        fi
+        if [[ -n "${ADDITIONAL_SLIM_BUILD_ARGS}" ]]; then
+            while IFS= read -r -d '' t; do slim_args+=("$t"); done \
+                < <(printf '%s' "${ADDITIONAL_SLIM_BUILD_ARGS}" | xargs printf '%s\0')
+        fi
+        ./mint slim --target "${image}" --tag "${slim_image}" "${slim_args[@]}"
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

- All 9 uses: refs (checkout v3, setup-qemu v2, login v3, setup-buildx v3, change-string-case v6 x3, industrial_ci master, github-slug v1) are pinned to the commit SHAs the lookup returned, with the tag kept as a comment.
- In the 'Set up industrial_ci' step, inputs.build-context now goes through env BUILD_CONTEXT and is used as "$BUILD_CONTEXT/.repos" in double quotes.
- scripts/ci.sh: newlines are now stripped from the image name before it is written to GITHUB_OUTPUT.
- Left out from the previous patch: it rewrote the mint slim call to split the flags with 'xargs printf'. Upstream expands $SLIM_BUILD_ARGS/$ADDITIONAL_SLIM_BUILD_ARGS unquoted from env. That only splits words and expands globs; it runs nothing, so it is not injection. xargs would also change how the arguments are split, and the script's built-in default is wrapped in quote characters. Upstream's form is kept.
- Also left out: the previous patch flattened the slim-build-args default into one line and removed blank lines. Neither is a security change: the folded '>-' default already joins the lines with spaces.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Quoted ${GITHUB_ACTION_PATH} in the 'Set up docker-ros' step's cp command (action.yml).

