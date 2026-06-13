<!-- markdownlint-disable -->

# Hardening Report: ika-rwth-aachen--docker-ros/v1.8.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **ika-rwth-aachen--docker-ros/v1.8.1** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 9 `uses:` references in action.yml use mutable tags or branch names instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved.

Failing references:
- `uses: actions/checkout@v3`
- `uses: docker/setup-qemu-action@v2`
- `uses: docker/login-action@v3`
- `uses: docker/setup-buildx-action@v3`
- `uses: ASzc/change-string-case-action@v6` (×3)
- `uses: ros-industrial/industrial_ci@master`
- `uses: gacts/github-slug@v1`

Locations:

- `action.yml:113`
- `action.yml:125`
- `action.yml:128`
- `action.yml:133`
- `action.yml:137`
- `action.yml:142`
- `action.yml:147`
- `action.yml:179`
- `action.yml:194`

### script-injection (severity: high)

Sub-rule (a): The 'Set up industrial_ci' step directly interpolates `${{ inputs.build-context }}` inside a `run:` shell command string. An attacker-controlled value for `inputs.build-context` is expanded by the YAML template engine before the shell sees it, enabling arbitrary command injection.

Offending line: `run: test -f ${{ inputs.build-context }}/.repos || echo "repositories:" > ${{ inputs.build-context }}/.repos`

Locations:

- `action.yml:175`

### script-injection (severity: high)

Sub-rule (b): In scripts/ci.sh, the variable `${SLIM_BUILD_ARGS}` is expanded unquoted in the `./slim build` command. This variable is set from `inputs.slim-build-args` (a workflow-controllable input) via the `env:` block. An unquoted expansion allows the shell to parse metacharacters (`;`, `|`, `&`, `$(...)`, etc.) from the value, enabling command injection.

Offending line: `./slim build --target "${image}" --tag "${slim_image}" ${SLIM_BUILD_ARGS}`

Locations:

- `scripts/ci.sh:80`

### github-env-injection (severity: high)

In scripts/ci.sh, the variable `industrial_ci_image` is derived from workflow-controlled inputs (`IMAGE_NAME`, `IMAGE_TAG`, `DEV_IMAGE_NAME`, `DEV_IMAGE_TAG`, `_IMAGE_POSTFIX`, `PLATFORM` — all set from `inputs.*` via the `env:` block in action.yml) and written directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline embedded in any of these values could inject additional key=value pairs into the GitHub output file.

Offending line: `echo "INDUSTRIAL_CI_IMAGE=${industrial_ci_image}" >> "${GITHUB_OUTPUT}"`

Locations:

- `scripts/ci.sh:62`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.build-context }}" appears directly in run: block of step "Set up industrial_ci"; move to env: map

Locations:

- `action.yml:235`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.build-context }}" appears directly in run: block of step "Set up industrial_ci"; move to env: map

Locations:

- `action.yml:235`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all 6 findings:
1. Pinned all 9 unpinned `uses:` references to full 40-char SHA digests with tag comments preserved.
2. Fixed script injection in action.yml 'Set up industrial_ci' step: moved `${{ inputs.build-context }}` to env: block as BUILD_CONTEXT, referenced as "${BUILD_CONTEXT}" in shell.
3. Fixed unquoted SLIM_BUILD_ARGS expansion in scripts/ci.sh by splitting into an array with `IFS=' ' read -ra SLIM_BUILD_ARGS_ARRAY` and using `"${SLIM_BUILD_ARGS_ARRAY[@]}"` to prevent shell metacharacter injection.
4. Fixed github-env-injection in scripts/ci.sh by sanitizing industrial_ci_image with `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_OUTPUT.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in the 'Run industrial_ci' step's AFTER_INIT_EMBED env variable. Previously, ${{ inputs.git-https-server }}, ${{ inputs.git-https-user }}, and ${{ inputs.git-https-password }} were directly interpolated into the shell command string, allowing attacker-controlled values with shell metacharacters to be executed. The fix moves these three values into separate environment variables (GIT_HTTPS_SERVER, GIT_HTTPS_USER, GIT_HTTPS_PASSWORD) and updates AFTER_INIT_EMBED to reference them as plain shell variables ($GIT_HTTPS_SERVER, $GIT_HTTPS_USER, $GIT_HTTPS_PASSWORD), so the values are passed safely as environment variables rather than being embedded in the command string at template-expansion time.

