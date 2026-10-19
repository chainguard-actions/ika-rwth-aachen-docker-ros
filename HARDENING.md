<!-- markdownlint-disable -->

# Hardening Report: ika-rwth-aachen--docker-ros/v1.11.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ika-rwth-aachen--docker-ros/v1.11.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 9 `uses:` references in action.yml pin to mutable tags or branch names instead of immutable 40-character commit SHAs. This exposes the action to supply-chain attacks if any upstream action is compromised or its tag is moved. Failing references:
- `actions/checkout@v3`
- `docker/setup-qemu-action@v2`
- `docker/login-action@v3`
- `docker/setup-buildx-action@v3`
- `ASzc/change-string-case-action@v6` (×3)
- `ros-industrial/industrial_ci@master`
- `gacts/github-slug@v1`

Locations:

- `action.yml:149`
- `action.yml:161`
- `action.yml:164`
- `action.yml:169`
- `action.yml:173`
- `action.yml:179`
- `action.yml:185`
- `action.yml:222`
- `action.yml:251`

### github-env-injection (severity: high)

In scripts/ci.sh, user-controlled values derived from action inputs (IMAGE_NAME, IMAGE_TAG, DEV_IMAGE_NAME, DEV_IMAGE_TAG, SLIM_IMAGE_NAME, SLIM_IMAGE_TAG, _IMAGE_POSTFIX) are written directly to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). These variables are set from `inputs.*` passed via `env:` blocks in action.yml. An attacker-controlled input containing newlines could inject arbitrary key=value pairs into the GitHub output context. Offending lines:
  `echo "RUN_IMAGE=${IMAGE}${_IMAGE_POSTFIX}" >> "${GITHUB_OUTPUT}"`
  `echo "DEV_IMAGE=${DEV_IMAGE}${_IMAGE_POSTFIX}" >> "${GITHUB_OUTPUT}"`
  `echo "SLIM_IMAGE=${SLIM_IMAGE}${_IMAGE_POSTFIX}" >> "${GITHUB_OUTPUT}"`
  `echo "INDUSTRIAL_CI_IMAGE=${industrial_ci_image}" >> "${GITHUB_OUTPUT}"`

Locations:

- `scripts/ci.sh:85`
- `scripts/ci.sh:86`
- `scripts/ci.sh:87`
- `scripts/ci.sh:89`

### script-injection (severity: high)

Rule (b) violation in scripts/ci.sh: The variables `${SLIM_BUILD_ARGS}` and `${ADDITIONAL_SLIM_BUILD_ARGS}` are expanded unquoted in the `mint slim` shell command. Both variables hold values sourced from `inputs.slim-build-args` and `inputs.additional-slim-build-args` respectively (set via `env:` in action.yml). Unquoted expansion allows the shell to parse metacharacters (`;`, `|`, `&`, `$(...)`, glob chars, whitespace word-splitting) from attacker-controlled input, enabling command injection. Offending line:
  `./mint slim --target "${image}" --tag "${slim_image}" ${SLIM_BUILD_ARGS} ${ADDITIONAL_SLIM_BUILD_ARGS}`

Locations:

- `scripts/ci.sh:115`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection, script-injection

**Notes:**

Fixed all 3 findings:

1. unpinned-uses: Pinned all 9 `uses:` references in action.yml to full 40-character commit SHAs using lookup_action_sha. Tags preserved as comments.

2. github-env-injection: In scripts/ci.sh, sanitized all 4 values written to $GITHUB_OUTPUT (RUN_IMAGE, DEV_IMAGE, SLIM_IMAGE, INDUSTRIAL_CI_IMAGE) by capturing them into temp variables via `printf '%s' ... | tr -d '\n\r'` before writing, preserving exit-status semantics under set -e.

3. script-injection: In scripts/ci.sh, replaced unquoted `${SLIM_BUILD_ARGS}` and `${ADDITIONAL_SLIM_BUILD_ARGS}` expansions with bash array tokenization using `read -r -d '' -a` (whitespace splitting, matching upstream's env-var expansion behavior), then expanded with `"${_slim_build_args[@]}"` and `"${_additional_slim_build_args[@]}"` to prevent shell metacharacter injection while preserving argument boundaries.

