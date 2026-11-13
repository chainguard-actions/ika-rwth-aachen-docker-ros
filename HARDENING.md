<!-- markdownlint-disable -->

# Hardening Report: ika-rwth-aachen--docker-ros/v1.11.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ika-rwth-aachen--docker-ros/v1.11.0** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml references actions by tag or branch instead of a full 40-character commit SHA: actions/checkout@v3, docker/setup-qemu-action@v2, docker/login-action@v3, docker/setup-buildx-action@v3, ASzc/change-string-case-action@v6 (3 times), ros-industrial/industrial_ci@master, gacts/github-slug@v1.

Locations:

- `action.yml:192`
- `action.yml:214`
- `action.yml:217`
- `action.yml:224`
- `action.yml:229`
- `action.yml:235`
- `action.yml:241`
- `action.yml:305`
- `action.yml:327`

### script-injection (severity: high)

Rule (b): the 'Set up docker-ros' step expands an inherited env var without quotes: `cp -r ${GITHUB_ACTION_PATH} docker/docker-ros`. In a composite action, env vars count as workflow-controlled, and this path also contains the workspace path. The shell splits it on whitespace and expands globs. Fix: `cp -r "${GITHUB_ACTION_PATH}" docker/docker-ros`.

Locations:

- `action.yml:204`

### github-env-injection (severity: high)

The 'Build images' and 'Push images' steps run scripts/ci.sh. That script writes values built from inputs to $GITHUB_OUTPUT without first removing newlines: `echo "RUN_IMAGE=${IMAGE}${_IMAGE_POSTFIX}"`, `echo "DEV_IMAGE=${DEV_IMAGE}${_IMAGE_POSTFIX}"`, `echo "SLIM_IMAGE=${SLIM_IMAGE}${_IMAGE_POSTFIX}"` and `echo "INDUSTRIAL_CI_IMAGE=${industrial_ci_image}"`. The values come from IMAGE_NAME/IMAGE_TAG, DEV_IMAGE_*, SLIM_IMAGE_*, PLATFORM and _IMAGE_POSTFIX, which are set from inputs.* and github.ref_name / head_ref. A value containing a newline can add extra output entries. Fix: strip newlines from each value with printf '%s' ... | tr -d '\n\r' before writing it.

Locations:

- `scripts/ci.sh:86`
- `scripts/ci.sh:87`
- `scripts/ci.sh:88`
- `scripts/ci.sh:90`

### script-injection (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. Rule (b) violation in scripts/ci.sh: The variables `${SLIM_BUILD_ARGS}` and `${ADDITIONAL_SLIM_BUILD_ARGS}` are expanded unquoted in the `mint slim` shell command. Both variables hold values sourced from `inputs.slim-build-args` and `inputs.additional-slim-build-args` respectively (set via `env:` in action.yml). Unquoted expansion allows the shell to parse metacharacters (`;`, `|`, `&`, `$(...)`, glob chars, whitespace word-splitting) from attacker-controlled input, enabling command injection. Offending line:
  `./mint slim --target "${image}" --tag "${slim_image}" ${SLIM_BUILD_ARGS} ${ADDITIONAL_SLIM_BUILD_ARGS}`

Locations:

- `scripts/ci.sh:115`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index d0dd8e7..a555d83 100644
--- a/action.yml
+++ b/action.yml
@@ -188,9 +188,26 @@ inputs:
 runs:
   using: "composite"
   steps:
 
     - name: Checkout repository
       uses: actions/checkout@v3
       if: ${{ inputs.enable-checkout == 'true' }}
       with:
         submodules: ${{ inputs.enable-checkout-submodules }}
@@ -212,33 +229,33 @@ runs:
 
     - name: Set up QEMU
       if: ${{ steps.prepare-setup-of-qemu.outputs.RUNNER_ARCH != inputs.platform }}
       uses: docker/setup-qemu-action@v2
 
     - name: Login to Docker registry
       uses: docker/login-action@v3
       with:
         registry: ${{ inputs.registry }}
         username: ${{ inputs.registry-user }}
         password: ${{ inputs.registry-password }}
 
     - name: Set up Docker buildx
       uses: docker/setup-buildx-action@v3
 
     - name: Enforce lower-case image name 
       id: image-name
       uses: ASzc/change-string-case-action@v6
       with:
         string: ${{ inputs.image-name }}
 
     - name: Enforce lower-case dev image name 
       id: dev-image-name
       uses: ASzc/change-string-case-action@v6
       with:
         string: ${{ inputs.dev-image-name }}
 
     - name: Enforce lower-case slim image name 
       id: slim-image-name
       uses: ASzc/change-string-case-action@v6
       with:
         string: ${{ inputs.slim-image-name }}
 
@@ -295,7 +312,7 @@ runs:
 
     - name: Run industrial_ci
       if: ${{ inputs.enable-industrial-ci == 'true' }}
-      uses: ros-industrial/industrial_ci@master
+      uses: ros-industrial/industrial_ci@125164b9f1883cdf1858897a7146d1bebf2be5c6 # master
       env:
         ADDITIONAL_DEBS: git
         AFTER_SETUP_UPSTREAM_WORKSPACE: 'rosenv bash "${{ inputs.build-context }}/docker/docker-ros/scripts/industrial-ci-upstream.sh"'
@@ -314,7 +331,7 @@ runs:
 
     - name: Slugify ref name
       id: slugify-ref-name
       uses: gacts/github-slug@v1
       with:
         to-slug: ${{ github.head_ref || github.ref_name }}
 

```

Locations:

- `action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed scripts/ci.sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/scripts/ci.sh b/scripts/ci.sh
index 3d7f366..25c8809 100755
--- a/scripts/ci.sh
+++ b/scripts/ci.sh
@@ -83,11 +83,15 @@ else
 fi
 if [[ -n "${GITHUB_OUTPUT:-}" ]]; then
     if [[ "${_ENABLE_IMAGE_PUSH}" == "true" ]]; then
-        echo "RUN_IMAGE=${IMAGE}${_IMAGE_POSTFIX}" >> "${GITHUB_OUTPUT}"
-        echo "DEV_IMAGE=${DEV_IMAGE}${_IMAGE_POSTFIX}" >> "${GITHUB_OUTPUT}"
-        echo "SLIM_IMAGE=${SLIM_IMAGE}${_IMAGE_POSTFIX}" >> "${GITHUB_OUTPUT}"
+        _safe_run_image="$(printf '%s' "${IMAGE}${_IMAGE_POSTFIX}" | tr -d '\n\r')"
+        _safe_dev_image="$(printf '%s' "${DEV_IMAGE}${_IMAGE_POSTFIX}" | tr -d '\n\r')"
+        _safe_slim_image="$(printf '%s' "${SLIM_IMAGE}${_IMAGE_POSTFIX}" | tr -d '\n\r')"
+        echo "RUN_IMAGE=${_safe_run_image}" >> "${GITHUB_OUTPUT}"
+        echo "DEV_IMAGE=${_safe_dev_image}" >> "${GITHUB_OUTPUT}"
+        echo "SLIM_IMAGE=${_safe_slim_image}" >> "${GITHUB_OUTPUT}"
     fi
-    echo "INDUSTRIAL_CI_IMAGE=${industrial_ci_image}" >> "${GITHUB_OUTPUT}"
+    _safe_industrial_ci_image="$(printf '%s' "${industrial_ci_image}" | tr -d '\n\r')"
+    echo "INDUSTRIAL_CI_IMAGE=${_safe_industrial_ci_image}" >> "${GITHUB_OUTPUT}"
 fi
 
 # parse (potentially) comma-separated lists to arrays
@@ -132,7 +136,15 @@ for PLATFORM in "${PLATFORMS[@]}"; do
         cd dist_linux*
         export DOCKER_API_VERSION="${DOCKER_API_VERSION:-$(docker version --format '{{.Server.APIVersion}}')}"
         docker pull "${image}"
-        ./mint slim --target "${image}" --tag "${slim_image}" ${SLIM_BUILD_ARGS} ${ADDITIONAL_SLIM_BUILD_ARGS}
+        _slim_build_args=()
+        if [[ -n "${SLIM_BUILD_ARGS}" ]]; then
+            read -r -d '' -a _slim_build_args <<< "${SLIM_BUILD_ARGS}" || true
+        fi
+        _additional_slim_build_args=()
+        if [[ -n "${ADDITIONAL_SLIM_BUILD_ARGS}" ]]; then
+            read -r -d '' -a _additional_slim_build_args <<< "${ADDITIONAL_SLIM_BUILD_ARGS}" || true
+        fi
+        ./mint slim --target "${image}" --tag "${slim_image}" "${_slim_build_args[@]}" "${_additional_slim_build_args[@]}"
         docker push "${slim_image}"
         cd -
         rm -rf dist_linux* ds.tar.gz

```

Locations:

- `scripts/ci.sh`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, prior-fix

**Notes:**

- **Pinned actions (action.yml):** every `uses:` now points to a full commit SHA, with the tag kept as a comment. That covers checkout v3, setup-qemu v2, login v3, setup-buildx v3, change-string-case v6 (3 times), industrial_ci master and github-slug v1.
- **Quoted path (action.yml):** `cp -r` now quotes `${GITHUB_ACTION_PATH}`.
- **Clean outputs (scripts/ci.sh):** newlines and carriage returns are stripped with `printf '%s' ... | tr -d '\n\r'` before RUN_IMAGE, DEV_IMAGE, SLIM_IMAGE and INDUSTRIAL_CI_IMAGE are written to $GITHUB_OUTPUT.
- **`mint slim` arguments (scripts/ci.sh):** SLIM_BUILD_ARGS and ADDITIONAL_SLIM_BUILD_ARGS are now each loaded into an array with an unquoted assignment, then passed as quoted arrays. The value is never run as code, and the split and glob behaviour is the same as upstream's unquoted expansion. An empty value gives no argument.
- **Differs from the previous harden:** the previous harden used `read -r -d '' -a`, which never globs, so a pattern would reach `mint` as literal text instead of matching files the way upstream did. I didn't re-apply that part.

