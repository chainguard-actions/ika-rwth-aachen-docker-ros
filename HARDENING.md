<!-- markdownlint-disable -->

# Hardening Report: ika-rwth-aachen--docker-ros/v1.9.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **ika-rwth-aachen--docker-ros/v1.9.0** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 9 `uses:` references in action.yml are pinned to mutable version tags or branch names instead of immutable 40-character commit SHAs. This exposes the action to supply-chain attacks if any upstream action is compromised or its tag is moved. Failing references: `actions/checkout@v3`, `docker/setup-qemu-action@v2`, `docker/login-action@v3`, `docker/setup-buildx-action@v3`, `ASzc/change-string-case-action@v6` (×3), `ros-industrial/industrial_ci@master`, `gacts/github-slug@v1`.

Locations:

- `action.yml:143`
- `action.yml:155`
- `action.yml:160`
- `action.yml:166`
- `action.yml:172`
- `action.yml:178`
- `action.yml:184`
- `action.yml:222`
- `action.yml:237`

### script-injection (severity: high)

Rule (a): The 'Set up industrial_ci' run: block directly interpolates the expression `${{ inputs.build-context }}` into a shell command string: `run: test -f ${{ inputs.build-context }}/.repos || echo "repositories:" > ${{ inputs.build-context }}/.repos`. An attacker-controlled value for `inputs.build-context` (e.g. containing shell metacharacters) is expanded by the YAML template engine before the shell ever sees it, enabling command injection.

Locations:

- `action.yml:218`

### script-injection (severity: high)

Rule (b): In scripts/ci.sh (invoked from multiple `run:` steps in action.yml), the variables `${SLIM_BUILD_ARGS}` and `${ADDITIONAL_SLIM_BUILD_ARGS}` are expanded unquoted in the shell command: `./mint slim --target "${image}" --tag "${slim_image}" ${SLIM_BUILD_ARGS} ${ADDITIONAL_SLIM_BUILD_ARGS}`. Both variables are sourced from `inputs.slim-build-args` and `inputs.additional-slim-build-args` (workflow-controllable). Unquoted expansion allows shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) in those inputs to be interpreted by the shell, enabling command injection.

Locations:

- `scripts/ci.sh:90`

### github-env-injection (severity: high)

In scripts/ci.sh, the variable `industrial_ci_image` is constructed from `IMAGE` (= `IMAGE_NAME:IMAGE_TAG`), `DEV_IMAGE`, `_IMAGE_POSTFIX`, and `PLATFORM` — all of which are set from caller-controlled inputs (`inputs.image-name`, `inputs.image-tag`, `inputs.dev-image-name`, `inputs.dev-image-tag`, `inputs.platform`, and `github.ref`/`steps.slugify-ref-name.outputs.slug` for `_IMAGE_POSTFIX`). This value is written to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`): `echo "INDUSTRIAL_CI_IMAGE=${industrial_ci_image}" >> "${GITHUB_OUTPUT}"`. A newline embedded in any of these inputs could inject arbitrary key-value pairs into the GitHub output context.

Locations:

- `scripts/ci.sh:72`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.build-context }}" appears directly in run: block of step "Set up industrial_ci"; move to env: map

Locations:

- `action.yml:261`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.build-context }}" appears directly in run: block of step "Set up industrial_ci"; move to env: map

Locations:

- `action.yml:261`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all 6 findings: (1) Pinned all 9 uses: references in action.yml to full 40-char commit SHAs using lookup_action_sha. (2) Fixed script-injection in action.yml 'Set up industrial_ci' step by moving ${{ inputs.build-context }} to an env: block as BUILD_CONTEXT and referencing it as "${BUILD_CONTEXT}" in the shell. (3) Fixed script-injection in scripts/ci.sh by converting unquoted ${SLIM_BUILD_ARGS} and ${ADDITIONAL_SLIM_BUILD_ARGS} to arrays via IFS=' ' read -ra and passing them as "${_slim_args[@]}" and "${_additional_slim_args[@]}". (4) Fixed github-env-injection in scripts/ci.sh by sanitizing industrial_ci_image with printf '%s' ... | tr -d '\n\r' before writing to $GITHUB_OUTPUT.

