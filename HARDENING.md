<!-- markdownlint-disable -->

# Hardening Report: ika-rwth-aachen--docker-ros/v1.8.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ika-rwth-aachen--docker-ros/v1.8.1** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 9 `uses:` references in action.yml are pinned to mutable tags or branches rather than immutable 40-character commit SHAs. This exposes the action to supply-chain attacks if any upstream action is compromised or its tag is moved. Failing references: `actions/checkout@v3`, `docker/setup-qemu-action@v2`, `docker/login-action@v3`, `docker/setup-buildx-action@v3`, `ASzc/change-string-case-action@v6` (×3), `ros-industrial/industrial_ci@master`, `gacts/github-slug@v1`.

Locations:

- `action.yml:113`
- `action.yml:127`
- `action.yml:130`
- `action.yml:135`
- `action.yml:140`
- `action.yml:145`
- `action.yml:150`
- `action.yml:178`
- `action.yml:192`

### script-injection (severity: high)

Sub-rule (a): The 'Set up industrial_ci' step directly interpolates `${{ inputs.build-context }}` inside a `run:` shell command string: `run: test -f ${{ inputs.build-context }}/.repos || echo "repositories:" > ${{ inputs.build-context }}/.repos`. An attacker-controlled value for `inputs.build-context` is expanded by the GitHub Actions template engine before the shell ever sees it, enabling arbitrary command injection (e.g. a value containing `;`, `$(...)`, or `&&` sequences).

Locations:

- `action.yml:175`

### github-env-injection (severity: high)

In scripts/ci.sh, the variable `industrial_ci_image` is written to `$GITHUB_OUTPUT` without sanitization. Its value is derived from `IMAGE_NAME`, `IMAGE_TAG`, `DEV_IMAGE_NAME`, and `DEV_IMAGE_TAG` environment variables, all of which are set from `inputs.*` values (e.g. `IMAGE_NAME: ${{ inputs.image-name }}`, `IMAGE_TAG: ${{ inputs.image-tag }}`). A newline embedded in any of these inputs could inject additional key=value pairs into `$GITHUB_OUTPUT`. The required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`) is absent before the write: `echo "INDUSTRIAL_CI_IMAGE=${industrial_ci_image}" >> "${GITHUB_OUTPUT}"`.

Locations:

- `scripts/ci.sh:64`

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

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection, github-env-injection

**Notes:**

Fixed all 9 unpinned uses: references in action.yml by pinning to full 40-char commit SHAs (actions/checkout@v3, docker/setup-qemu-action@v2, docker/login-action@v3, docker/setup-buildx-action@v3, ASzc/change-string-case-action@v6 ×3, ros-industrial/industrial_ci@master, gacts/github-slug@v1). Fixed script injection in the 'Set up industrial_ci' step by moving ${{ inputs.build-context }} to an env: block as BUILD_CONTEXT and referencing it as $BUILD_CONTEXT in the shell. Fixed github-env-injection in scripts/ci.sh by sanitizing industrial_ci_image with printf/tr before writing to $GITHUB_OUTPUT.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection vulnerabilities:

1. action.yml (Run industrial_ci step): Replaced inline ${{ inputs.git-https-server }}, ${{ inputs.git-https-user }}, and ${{ inputs.git-https-password }} expressions in the AFTER_INIT_EMBED shell command string with shell variable references ($GIT_HTTPS_SERVER, $GIT_HTTPS_USER, $GIT_HTTPS_PASSWORD). Added GIT_HTTPS_PASSWORD, GIT_HTTPS_SERVER, and GIT_HTTPS_USER as separate env vars in the step's env block so the values are passed safely as environment variables rather than being interpolated into the command string.

2. scripts/ci.sh (slim build command): Replaced the unquoted ${SLIM_BUILD_ARGS} expansion with a bash array populated via xargs-based tokenization (using the while/read/printf pattern). The array is then expanded as "${slim_args[@]}" to preserve argument boundaries and prevent shell metacharacter injection from attacker-controlled input.

