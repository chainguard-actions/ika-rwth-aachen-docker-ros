<!-- markdownlint-disable -->

# Hardening Report: ika-rwth-aachen--docker-ros/v1.10.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **ika-rwth-aachen--docker-ros/v1.10.0** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 9 `uses:` references in action.yml are pinned to mutable tags or branch names rather than immutable 40-character commit SHAs. This exposes the action to supply-chain attacks. Failing references: `actions/checkout@v3`, `docker/setup-qemu-action@v2`, `docker/login-action@v3`, `docker/setup-buildx-action@v3`, `ASzc/change-string-case-action@v6` (used 3 times), `ros-industrial/industrial_ci@master` (branch reference — especially dangerous), `gacts/github-slug@v1`.

Locations:

- `action.yml:148`
- `action.yml:155`
- `action.yml:159`
- `action.yml:163`
- `action.yml:168`
- `action.yml:173`
- `action.yml:178`
- `action.yml:237`
- `action.yml:264`

### script-injection (severity: high)

Sub-rule (a): The 'Set up industrial_ci' run: block directly interpolates `${{ inputs.build-context }}` into the shell command string. This allows an attacker who controls the `build-context` input to inject arbitrary shell commands. Offending line: `run: test -f ${{ inputs.build-context }}/.repos || echo "repositories:" > ${{ inputs.build-context }}/.repos`

Locations:

- `action.yml:233`

### github-env-injection (severity: high)

In scripts/ci.sh, the variable `industrial_ci_image` is constructed from `IMAGE_NAME`, `IMAGE_TAG`, `DEV_IMAGE_NAME`, `DEV_IMAGE_TAG`, and `_IMAGE_POSTFIX` — all of which are set from `inputs.*` values passed via env vars in action.yml. This value is written directly to `$GITHUB_OUTPUT` with `echo "INDUSTRIAL_CI_IMAGE=${industrial_ci_image}" >> "${GITHUB_OUTPUT}"` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline embedded in any of these inputs could inject arbitrary key=value pairs into GITHUB_OUTPUT.

Locations:

- `scripts/ci.sh:62`

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

Fixed all 5 findings: (1) Pinned all 9 uses: references to full commit SHAs: actions/checkout@v3→f43a0e5, docker/setup-qemu-action@v2→2b82ce8, docker/login-action@v3→c94ce9f, docker/setup-buildx-action@v3→8d2750c, ASzc/change-string-case-action@v6→d0603cd (3 occurrences), ros-industrial/industrial_ci@master→125164b, gacts/github-slug@v1→83cd3d9. (2) Fixed script injection in 'Set up industrial_ci' step by moving ${{ inputs.build-context }} into an env: block as BUILD_CONTEXT and referencing it as "$BUILD_CONTEXT" in the shell script. (3) Fixed github-env-injection in scripts/ci.sh by sanitizing industrial_ci_image with printf '%s' ... | tr -d '\n\r' before writing to $GITHUB_OUTPUT.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings:

1. action.yml (line 247, 'Set up industrial_ci' step): Changed `"$BUILD_CONTEXT/.repos"` to `"${BUILD_CONTEXT}/.repos"` for both occurrences in the run command. The variable was already sourced from the `env:` block (not a raw `${{ }}` expression), and both occurrences are now properly double-quoted with braces to prevent word splitting.

2. scripts/ci.sh (line 107): Replaced unquoted `${SLIM_BUILD_ARGS}` and `${ADDITIONAL_SLIM_BUILD_ARGS}` expansions with array-based splitting using `IFS=' ' read -ra`. The arrays are then expanded as `"${_slim_build_args[@]}"` and `"${_additional_slim_build_args[@]}"`, which properly handles multi-word arguments while preventing shell metacharacter injection from user-controlled input.

