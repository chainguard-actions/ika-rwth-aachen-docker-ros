<!-- markdownlint-disable -->

# Hardening Report: ika-rwth-aachen--docker-ros/v1.9.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ika-rwth-aachen--docker-ros/v1.9.0** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 9 `uses:` references in action.yml are pinned to mutable tags or branch names instead of immutable 40-character commit SHAs. This exposes the action to supply-chain attacks where a compromised upstream action tag could silently execute malicious code. Failing references:
- `actions/checkout@v3` (line 161)
- `docker/setup-qemu-action@v2` (line 183)
- `docker/login-action@v3` (line 186)
- `docker/setup-buildx-action@v3` (line 193)
- `ASzc/change-string-case-action@v6` (line 197)
- `ASzc/change-string-case-action@v6` (line 203)
- `ASzc/change-string-case-action@v6` (line 209)
- `ros-industrial/industrial_ci@master` (line 255) — branch reference, especially dangerous
- `gacts/github-slug@v1` (line 270)

Locations:

- `action.yml:161`
- `action.yml:183`
- `action.yml:186`
- `action.yml:193`
- `action.yml:197`
- `action.yml:203`
- `action.yml:209`
- `action.yml:255`
- `action.yml:270`

### script-injection (severity: high)

Sub-rule (a): The 'Set up industrial_ci' step directly interpolates `${{ inputs.build-context }}` inside a `run:` shell command string. The offending line is:

  run: test -f ${{ inputs.build-context }}/.repos || echo "repositories:" > ${{ inputs.build-context }}/.repos

Because `inputs.build-context` is caller-controlled, an attacker can supply a value such as `x; malicious-command #` to execute arbitrary shell commands on the runner. The value must be passed through an `env:` variable and the variable must be double-quoted in the shell script to be safe.

Locations:

- `action.yml:251`

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

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection

**Notes:**

Pinned all 9 `uses:` references to immutable commit SHAs: actions/checkout@v3 → a37ce9120846195fa4ece8f58b268e6043cb2f26, docker/setup-qemu-action@v2 → 2b82ce82d56a2a04d2637cd93a637ae1b359c0a7, docker/login-action@v3 → c94ce9fb468520275223c153574b00df6fe4bcc9, docker/setup-buildx-action@v3 → 8d2750c68a42422c14e847fe6c8ac0403b4cbd6f, ASzc/change-string-case-action@v6 (×3) → d0603cd0a7dd490be678164909f65c7737470a7f, ros-industrial/industrial_ci@master → 125164b9f1883cdf1858897a7146d1bebf2be5c6, gacts/github-slug@v1 → c29c0ddd888a1703a6fc06c55a5f6ddd7beae490. Fixed script injection in 'Set up industrial_ci' step by moving ${{ inputs.build-context }} to an env: variable BUILD_CONTEXT and referencing it as "$BUILD_CONTEXT" in the shell script.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Three fixes applied:
1. action.yml AFTER_INIT_EMBED: Replaced ${{ inputs.git-https-* }} template expressions with plain shell variable references ($GIT_HTTPS_SERVER, $GIT_HTTPS_USER, $GIT_HTTPS_PASSWORD) and added those as env vars to the 'Run industrial_ci' step's env block.
2. scripts/ci.sh line 97: Replaced unquoted ${SLIM_BUILD_ARGS} ${ADDITIONAL_SLIM_BUILD_ARGS} expansion with xargs-based tokenization into a bash array (slim_args), then expanded with "${slim_args[@]}" to prevent shell metacharacter injection.
3. scripts/ci.sh line 72: Added newline sanitization before writing industrial_ci_image to $GITHUB_OUTPUT using printf '%s' ... | tr -d '\n\r'.

