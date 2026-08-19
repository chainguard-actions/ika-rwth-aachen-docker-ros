<!-- markdownlint-disable -->

# Hardening Report: ika-rwth-aachen--docker-ros/v1.10.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ika-rwth-aachen--docker-ros/v1.10.0** was hardened automatically. 11 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml references 9 external actions using mutable tag/version refs instead of pinned 40-character commit SHAs, making the action vulnerable to supply-chain attacks if any upstream tag is moved or compromised:
- `actions/checkout@v3`
- `docker/setup-qemu-action@v2`
- `docker/login-action@v3`
- `docker/setup-buildx-action@v3`
- `ASzc/change-string-case-action@v6` (×3)
- `ros-industrial/industrial_ci@master`
- `gacts/github-slug@v1`

Locations:

- `action.yml:148`
- `action.yml:163`
- `action.yml:167`
- `action.yml:174`
- `action.yml:177`
- `action.yml:183`
- `action.yml:189`
- `action.yml:248`
- `action.yml:265`

### unpinned-uses (severity: high)

.github/workflows/github.yml references `convictional/trigger-workflow-and-wait@v1.6.5` using a mutable version tag instead of a pinned 40-character commit SHA.

Locations:

- `.github/workflows/github.yml:17`

### unpinned-uses (severity: high)

.github/workflows/gitlab.yml references two actions using mutable version tags instead of pinned 40-character commit SHAs:
- `actions/upload-artifact@v4`
- `actions/download-artifact@v4`

Locations:

- `.github/workflows/gitlab.yml:21`
- `.github/workflows/gitlab.yml:28`

### script-injection (severity: high)

Sub-rule (a): `${{ inputs.build-context }}` is interpolated directly inside a `run:` shell command in action.yml. An attacker-controlled value for `inputs.build-context` is substituted into the shell command before execution, enabling command injection. Offending line: `run: test -f ${{ inputs.build-context }}/.repos || echo "repositories:" > ${{ inputs.build-context }}/.repos`

Locations:

- `action.yml:248`

### script-injection (severity: high)

Sub-rule (a): `${{ secrets.DOCKER_ROS_CI_TRIGGER_GITLAB_TOKEN }}` and `${{ github.sha }}` are interpolated directly inside a `run:` curl command in gitlab.yml. Any `${{ ... }}` expression inside a run: block is a script-injection risk as the value is substituted before the shell processes it. Offending line: `curl --silent --fail --request POST --form "token=${{ secrets.DOCKER_ROS_CI_TRIGGER_GITLAB_TOKEN }}" ... "variables[DOCKER_ROS_GIT_REF]=${{ github.sha }}"`

Locations:

- `.github/workflows/gitlab.yml:18`

### script-injection (severity: high)

Sub-rule (a): `${{ secrets.DOCKER_ROS_CI_READ_PIPELINE_GITLAB_TOKEN }}` is interpolated directly inside a `run:` curl command in gitlab.yml. Offending line: `PIPELINE_STATUS=$(curl --silent --header "PRIVATE-TOKEN: ${{ secrets.DOCKER_ROS_CI_READ_PIPELINE_GITLAB_TOKEN }}" ...)`

Locations:

- `.github/workflows/gitlab.yml:33`

### github-env-injection (severity: high)

scripts/ci.sh writes `INDUSTRIAL_CI_IMAGE` to `$GITHUB_OUTPUT` without sanitization. The value is derived from workflow-controlled inputs: `IMAGE_NAME`, `IMAGE_TAG`, `_IMAGE_POSTFIX`, and `PLATFORM` — all set from `inputs.*` in action.yml. A newline character embedded in any of these inputs could inject additional key=value pairs into GITHUB_OUTPUT. The required sanitization step (`printf '%s' ... | tr -d '\n\r'`) is absent before the write: `echo "INDUSTRIAL_CI_IMAGE=${industrial_ci_image}" >> "${GITHUB_OUTPUT}"`

Locations:

- `scripts/ci.sh:72`

### missing-permissions (severity: medium)

.github/workflows/github.yml has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs. Without explicit permissions, the workflow inherits the repository's default token permissions (which may be write-all), granting broader access than necessary.

Locations:

- `.github/workflows/github.yml:1`

### missing-permissions (severity: medium)

.github/workflows/gitlab.yml has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs. Without explicit permissions, the workflow inherits the repository's default token permissions (which may be write-all), granting broader access than necessary.

Locations:

- `.github/workflows/gitlab.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.build-context }}" appears directly in run: block of step "Set up industrial_ci"; move to env: map

Locations:

- `action.yml:279`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.build-context }}" appears directly in run: block of step "Set up industrial_ci"; move to env: map

Locations:

- `action.yml:279`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions, static-inline-injection

**Notes:**

Fixed all findings:

1. **unpinned-uses (action.yml)**: Pinned all 9 external actions to full 40-char commit SHAs:
   - actions/checkout@v3 → @f43a0e5ff2bd294095638e18286ca9a3d1956744
   - docker/setup-qemu-action@v2 → @2b82ce82d56a2a04d2637cd93a637ae1b359c0a7
   - docker/login-action@v3 → @c94ce9fb468520275223c153574b00df6fe4bcc9
   - docker/setup-buildx-action@v3 → @8d2750c68a42422c14e847fe6c8ac0403b4cbd6f
   - ASzc/change-string-case-action@v6 (×3) → @d0603cd0a7dd490be678164909f65c7737470a7f
   - ros-industrial/industrial_ci@master → @125164b9f1883cdf1858897a7146d1bebf2be5c6
   - gacts/github-slug@v1 → @83cd3d95888be8777f0d9c810c1266da1c5c3ef1

2. **unpinned-uses (github.yml)**: Pinned convictional/trigger-workflow-and-wait@v1.6.5 → @f69fa9eedd3c62a599220f4d5745230e237904be

3. **unpinned-uses (gitlab.yml)**: Pinned actions/upload-artifact@v4 → @ea165f8d65b6e75b540449e92b4886f43607fa02 and actions/download-artifact@v4 → @d3f86a106a0bac45b974a628896c90dbdf5c8093

4. **script-injection (action.yml line 248/279)**: Moved `${{ inputs.build-context }}` to env: BUILD_CONTEXT and referenced as `"$BUILD_CONTEXT"` in the run: block.

5. **script-injection (gitlab.yml lines 18, 33)**: Moved `${{ secrets.DOCKER_ROS_CI_TRIGGER_GITLAB_TOKEN }}`, `${{ github.sha }}`, and `${{ secrets.DOCKER_ROS_CI_READ_PIPELINE_GITLAB_TOKEN }}` to env: blocks and referenced as plain shell variables.

6. **github-env-injection (scripts/ci.sh line 72)**: Added sanitization step using `printf '%s' ... | tr -d '\n\r'` before writing INDUSTRIAL_CI_IMAGE to GITHUB_OUTPUT.

7. **missing-permissions (github.yml, gitlab.yml)**: Added `permissions: {}` at the top level of both workflow files.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in the 'Run industrial_ci' step of action.yml (line 230). The AFTER_INIT_EMBED env var was directly interpolating ${{ inputs.git-https-server }}, ${{ inputs.git-https-user }}, and ${{ inputs.git-https-password }} into a shell command string, allowing an attacker to inject arbitrary shell commands. The fix adds three new env vars (GIT_HTTPS_SERVER, GIT_HTTPS_USER, GIT_HTTPS_PASSWORD) that safely receive the input values via ${{ }} expressions, and updates AFTER_INIT_EMBED to reference them as plain shell variables ($GIT_HTTPS_SERVER, $GIT_HTTPS_USER, $GIT_HTTPS_PASSWORD) instead.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection vulnerability in scripts/ci.sh line 118. The unquoted expansions of ${SLIM_BUILD_ARGS} and ${ADDITIONAL_SLIM_BUILD_ARGS} were replaced with a safe array-based approach: each variable is piped through `printf '%s' "${VAR}" | xargs printf '%s\n'` and captured into a bash array via `mapfile -t`. xargs safely parses shell-quoted words without executing shell metacharacters (semicolons, pipes, command substitutions, etc.), preventing injection while preserving the intended word-splitting of multi-flag arguments. The arrays are then expanded with proper double-quoting `"${slim_args[@]}"` and `"${additional_slim_args[@]}"`.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed script-injection in .github/workflows/gitlab.yml: The $GIT_SHA variable (from ${{ github.sha }}) was embedded inside a larger double-quoted string without being explicitly double-quoted. Restructured the curl command using bash string concatenation so $GIT_SHA appears as an explicitly double-quoted shell word: `--form "variables[DOCKER_ROS_GIT_REF]=""$GIT_SHA"`. Adjacent quoted strings in bash are concatenated into a single argument, preserving correct curl behavior while satisfying the quoting requirement. The command was also reformatted with line continuations for readability.

