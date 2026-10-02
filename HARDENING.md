<!-- markdownlint-disable -->

# Hardening Report: ika-rwth-aachen--docker-ros/v1.10.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ika-rwth-aachen--docker-ros/v1.10.0** was hardened automatically. 5 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 9 `uses:` references in action.yml are pinned to mutable tags or branch names instead of immutable 40-character commit SHAs. This exposes the action to supply-chain attacks where a compromised upstream tag could silently execute malicious code. Failing references: `actions/checkout@v3`, `docker/setup-qemu-action@v2`, `docker/login-action@v3`, `docker/setup-buildx-action@v3`, `ASzc/change-string-case-action@v6` (×3), `ros-industrial/industrial_ci@master`, `gacts/github-slug@v1`.

Locations:

- `action.yml:137`
- `action.yml:149`
- `action.yml:153`
- `action.yml:158`
- `action.yml:163`
- `action.yml:169`
- `action.yml:175`
- `action.yml:222`
- `action.yml:240`

### script-injection (severity: high)

Rule (a) violation: The 'Set up industrial_ci' step directly interpolates `${{ inputs.build-context }}` inside a `run:` shell command string: `run: test -f ${{ inputs.build-context }}/.repos || echo "repositories:" > ${{ inputs.build-context }}/.repos`. The `inputs.build-context` value is controlled by the calling workflow and is expanded by the GitHub Actions template engine before the shell ever sees it, allowing an attacker to inject arbitrary shell commands via a crafted `build-context` input value.

Locations:

- `action.yml:218`

### github-env-injection (severity: high)

In scripts/ci.sh, the variable `industrial_ci_image` is constructed from `IMAGE_NAME`, `IMAGE_TAG`, `DEV_IMAGE`, `_IMAGE_POSTFIX`, and `PLATFORM` — all inherited process env vars that are set directly from `inputs.*` values by action.yml (e.g. `IMAGE_NAME: ${{ inputs.image-name }}`, `IMAGE_TAG: ${{ inputs.image-tag }}`, `PLATFORM: ${{ inputs.platform }}`, `_IMAGE_POSTFIX: ${{ ... }}`). This composite value is then written to `$GITHUB_OUTPUT` via `echo "INDUSTRIAL_CI_IMAGE=${industrial_ci_image}" >> "${GITHUB_OUTPUT}"` without any newline-stripping sanitization (`printf '%s' ... | tr -d '\n\r'`). A calling workflow can supply a newline-containing input value to inject arbitrary key=value pairs into the GitHub output environment.

Locations:

- `scripts/ci.sh:72`

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

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection, github-env-injection

**Notes:**

Fixed all findings in action.yml and scripts/ci.sh:

1. unpinned-uses: Pinned all 9 uses: references to full commit SHAs:
   - actions/checkout@v3 → @a37ce9120846195fa4ece8f58b268e6043cb2f26
   - docker/setup-qemu-action@v2 → @2b82ce82d56a2a04d2637cd93a637ae1b359c0a7
   - docker/login-action@v3 → @c94ce9fb468520275223c153574b00df6fe4bcc9
   - docker/setup-buildx-action@v3 → @8d2750c68a42422c14e847fe6c8ac0403b4cbd6f
   - ASzc/change-string-case-action@v6 → @d0603cd0a7dd490be678164909f65c7737470a7f (×3)
   - ros-industrial/industrial_ci@master → @125164b9f1883cdf1858897a7146d1bebf2be5c6
   - gacts/github-slug@v1 → @c29c0ddd888a1703a6fc06c55a5f6ddd7beae490

2. script-injection / static-inline-injection: Moved ${{ inputs.build-context }} out of the 'Set up industrial_ci' run: block into an env: var BUILD_CONTEXT, then referenced it as "$BUILD_CONTEXT" in the shell command.

3. github-env-injection: In scripts/ci.sh, added newline sanitization before writing INDUSTRIAL_CI_IMAGE to $GITHUB_OUTPUT using: safe_industrial_ci_image="$(printf '%s' "${industrial_ci_image}" | tr -d '\n\r')".

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in the 'Run industrial_ci' step of action.yml. The AFTER_INIT_EMBED env var previously embedded ${{ inputs.git-https-server }}, ${{ inputs.git-https-user }}, and ${{ inputs.git-https-password }} directly into a shell command string, enabling shell metacharacter injection. Fixed by: (1) replacing the ${{ }} expressions in AFTER_INIT_EMBED with shell variable references ($GIT_HTTPS_SERVER, $GIT_HTTPS_USER, $GIT_HTTPS_PASSWORD), and (2) adding GIT_HTTPS_PASSWORD, GIT_HTTPS_SERVER, and GIT_HTTPS_USER as proper env: entries in the step, mapping the ${{ inputs.git-https-* }} expressions to environment variables. The ${{ }} expressions are now only used to set env var values (safe/data context), not embedded in shell command strings.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted expansion of ${SLIM_BUILD_ARGS} and ${ADDITIONAL_SLIM_BUILD_ARGS} in scripts/ci.sh line 97. Both variables are argument lists populated from workflow-controllable inputs (inputs.slim-build-args and inputs.additional-slim-build-args). Replaced the unquoted expansions with xargs-based tokenization into bash arrays (slim_build_args and additional_slim_build_args), guarded by if [ -n ... ] checks to prevent empty-value issues. The mint slim command now uses "${slim_build_args[@]}" and "${additional_slim_build_args[@]}" for safe, injection-proof argument passing.

