<!-- markdownlint-disable -->

# Hardening Report: ika-rwth-aachen--docker-ros/v1.10.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ika-rwth-aachen--docker-ros/v1.10.0** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 9 `uses:` references in action.yml use mutable tags or branch names instead of pinned 40-character SHA commit hashes. This exposes the action to supply-chain attacks if any of the referenced actions are compromised or their tags are moved. Failing references:
- `actions/checkout@v3`
- `docker/setup-qemu-action@v2`
- `docker/login-action@v3`
- `docker/setup-buildx-action@v3`
- `ASzc/change-string-case-action@v6` (used 3 times)
- `ros-industrial/industrial_ci@master` (branch ref — highest risk)
- `gacts/github-slug@v1`

Locations:

- `action.yml:161`
- `action.yml:176`
- `action.yml:181`
- `action.yml:187`
- `action.yml:192`
- `action.yml:198`
- `action.yml:204`
- `action.yml:261`
- `action.yml:278`

### script-injection (severity: high)

Rule (a): The 'Set up industrial_ci' step's `run:` block directly interpolates `${{ inputs.build-context }}` into a shell command string without routing through an env var. An attacker controlling the `build-context` input can inject arbitrary shell commands. Offending line: `run: test -f ${{ inputs.build-context }}/.repos || echo "repositories:" > ${{ inputs.build-context }}/.repos`

Rule (b): In scripts/ci.sh, the variables `${SLIM_BUILD_ARGS}` and `${ADDITIONAL_SLIM_BUILD_ARGS}` are expanded unquoted in the `mint slim` invocation: `./mint slim --target "${image}" --tag "${slim_image}" ${SLIM_BUILD_ARGS} ${ADDITIONAL_SLIM_BUILD_ARGS}`. These variables are populated from `inputs.slim-build-args` and `inputs.additional-slim-build-args` (set via `env:` in action.yml). Unquoted expansion allows shell metacharacter injection from attacker-controlled input values.

Locations:

- `action.yml:257`
- `scripts/ci.sh:113`

### github-env-injection (severity: high)

In scripts/ci.sh, the value `INDUSTRIAL_CI_IMAGE` is written to `$GITHUB_OUTPUT` without sanitization. The value is composed from `${IMAGE}`, `${DEV_IMAGE}`, `${_IMAGE_POSTFIX}`, and `${PLATFORM}` — all of which are inherited from the calling workflow's environment (set via `env:` in action.yml from `inputs.*` and `github.*` values such as `inputs.image-name`, `inputs.image-tag`, `inputs.platform`, and `github.ref_name`). A newline injected into any of these values could allow an attacker to write arbitrary key=value pairs to `$GITHUB_OUTPUT`, potentially overwriting subsequent step outputs. The write `echo "INDUSTRIAL_CI_IMAGE=${industrial_ci_image}" >> "${GITHUB_OUTPUT}"` is not preceded by the required `printf '%s' ... | tr -d '\n\r'` sanitization.

Locations:

- `scripts/ci.sh:76`

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

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all 9 unpinned uses: references in action.yml by resolving each to its full 40-char SHA (actions/checkout@v3, docker/setup-qemu-action@v2, docker/login-action@v3, docker/setup-buildx-action@v3, ASzc/change-string-case-action@v6 ×3, ros-industrial/industrial_ci@master, gacts/github-slug@v1). Fixed script injection in 'Set up industrial_ci' step by moving ${{ inputs.build-context }} to an env var BUILD_CONTEXT. Fixed github-env-injection in scripts/ci.sh by sanitizing industrial_ci_image with tr -d '\n\r' before writing to GITHUB_OUTPUT. Fixed unquoted ${SLIM_BUILD_ARGS} and ${ADDITIONAL_SLIM_BUILD_ARGS} in scripts/ci.sh by tokenizing them into bash arrays via xargs with proper guards.

