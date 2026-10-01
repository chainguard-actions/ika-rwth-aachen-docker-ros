<!-- markdownlint-disable -->

# Hardening Report: ika-rwth-aachen--docker-ros/v1.8.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ika-rwth-aachen--docker-ros/v1.8.1** was hardened automatically. 6 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 9 `uses:` references in action.yml use mutable tags or branch names instead of pinned 40-character commit SHAs. This exposes the action to supply-chain attacks if any of the referenced actions are compromised or their tags are moved. Unpinned references: `actions/checkout@v3`, `docker/setup-qemu-action@v2`, `docker/login-action@v3`, `docker/setup-buildx-action@v3`, `ASzc/change-string-case-action@v6` (×3), `ros-industrial/industrial_ci@master`, `gacts/github-slug@v1`.

Locations:

- `action.yml:118`
- `action.yml:130`
- `action.yml:133`
- `action.yml:139`
- `action.yml:143`
- `action.yml:148`
- `action.yml:153`
- `action.yml:196`
- `action.yml:210`

### script-injection (severity: high)

Sub-rule (a): The 'Set up industrial_ci' run: block directly interpolates `${{ inputs.build-context }}` into the shell command string: `run: test -f ${{ inputs.build-context }}/.repos || echo "repositories:" > ${{ inputs.build-context }}/.repos`. An attacker-controlled value for `inputs.build-context` is expanded by the YAML template engine before the shell ever sees it, enabling command injection via shell metacharacters.

Locations:

- `action.yml:192`

### script-injection (severity: high)

Sub-rule (b): In scripts/ci.sh, the variable `${SLIM_BUILD_ARGS}` is expanded unquoted in the slim build invocation: `./slim build --target "${image}" --tag "${slim_image}" ${SLIM_BUILD_ARGS}`. `SLIM_BUILD_ARGS` is set from `inputs.slim-build-args` (a caller-controlled input) via the env: block in action.yml. Without double-quoting, shell metacharacters in the value (`;`, `|`, `&`, `$(...)`, etc.) are interpreted by the shell, enabling command injection.

Locations:

- `scripts/ci.sh:83`

### github-env-injection (severity: high)

In scripts/ci.sh, the value `INDUSTRIAL_CI_IMAGE=${industrial_ci_image}` is written to `$GITHUB_OUTPUT` without sanitization. The variable `industrial_ci_image` is derived from `IMAGE` / `DEV_IMAGE` (sourced from caller-controlled inputs `IMAGE_NAME`, `DEV_IMAGE_NAME`) and `_IMAGE_POSTFIX` (also caller-controlled). A newline character embedded in any of these inputs could inject arbitrary key=value pairs into GITHUB_OUTPUT. The required sanitization step (`printf '%s' ... | tr -d '\n\r'`) is absent before the write.

Locations:

- `scripts/ci.sh:72`

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
1. Pinned all 9 `uses:` references to full 40-char commit SHAs (actions/checkout, docker/setup-qemu-action, docker/login-action, docker/setup-buildx-action, ASzc/change-string-case-action ×3, ros-industrial/industrial_ci, gacts/github-slug).
2. Fixed script-injection in 'Set up industrial_ci' step: moved `${{ inputs.build-context }}` to env: block as BUILD_CONTEXT, referenced as "${BUILD_CONTEXT}" in shell.
3. Fixed unquoted SLIM_BUILD_ARGS expansion in scripts/ci.sh: tokenized via xargs into a bash array and expanded as "${slim_args[@]}" to prevent shell metacharacter injection.
4. Fixed github-env-injection in scripts/ci.sh: sanitized industrial_ci_image with `printf '%s' ... | tr -d '\n\r'` before writing to GITHUB_OUTPUT.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in the 'Set up industrial_ci' step (action.yml ~line 222). The BUILD_CONTEXT env var (set from attacker-controllable ${{ inputs.build-context }}) was used as ${BUILD_CONTEXT} in the shell run command. Changed the run: scalar to a block scalar and used properly double-quoted "$BUILD_CONTEXT" variable expansion. The ${{ }} expression remains in the env: block (not directly in the shell script), following the correct pattern to prevent script injection.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed script-injection in the 'Run industrial_ci' step of action.yml. The AFTER_INIT_EMBED env var was directly interpolating ${{ inputs.git-https-server }}, ${{ inputs.git-https-user }}, and ${{ inputs.git-https-password }} into a shell command string that industrial_ci evaluates via eval. Fixed by: (1) adding GIT_HTTPS_SERVER, GIT_HTTPS_USER, and GIT_HTTPS_PASSWORD as separate env vars in the step's env: block with the ${{ inputs.* }} expressions, and (2) rewriting AFTER_INIT_EMBED to reference these env vars using plain shell variable syntax ($GIT_HTTPS_SERVER, $GIT_HTTPS_USER, $GIT_HTTPS_PASSWORD). The values are now passed as environment variables and treated as data by the shell, preventing injection of shell metacharacters.

