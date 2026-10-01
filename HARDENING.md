<!-- markdownlint-disable -->

# Hardening Report: ika-rwth-aachen--docker-ros/v1.9.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ika-rwth-aachen--docker-ros/v1.9.0** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 9 `uses:` references in action.yml use mutable version tags or branch names instead of pinned 40-character commit SHA digests. This exposes the action to supply-chain attacks where a compromised upstream action tag could execute malicious code. Failing references: `actions/checkout@v3`, `docker/setup-qemu-action@v2`, `docker/login-action@v3`, `docker/setup-buildx-action@v3`, `ASzc/change-string-case-action@v6` (×3), `ros-industrial/industrial_ci@master`, `gacts/github-slug@v1`.

Locations:

- `action.yml:143`
- `action.yml:153`
- `action.yml:156`
- `action.yml:161`
- `action.yml:165`
- `action.yml:171`
- `action.yml:177`
- `action.yml:219`
- `action.yml:233`

### script-injection (severity: high)

Sub-rule (a): The 'Set up industrial_ci' step directly interpolates the `${{ inputs.build-context }}` expression inside a `run:` shell command string: `run: test -f ${{ inputs.build-context }}/.repos || echo "repositories:" > ${{ inputs.build-context }}/.repos`. A calling workflow can supply a malicious value for `build-context` (e.g. containing shell metacharacters or command substitution) that will be expanded by the shell before execution, enabling arbitrary command injection.

Locations:

- `action.yml:215`

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

Pinned all 9 unpinned `uses:` references to full 40-character commit SHAs: actions/checkout@v3 → a37ce9120846195fa4ece8f58b268e6043cb2f26, docker/setup-qemu-action@v2 → 2b82ce82d56a2a04d2637cd93a637ae1b359c0a7, docker/login-action@v3 → c94ce9fb468520275223c153574b00df6fe4bcc9, docker/setup-buildx-action@v3 → 8d2750c68a42422c14e847fe6c8ac0403b4cbd6f, ASzc/change-string-case-action@v6 (×3) → d0603cd0a7dd490be678164909f65c7737470a7f, ros-industrial/industrial_ci@master → 125164b9f1883cdf1858897a7146d1bebf2be5c6, gacts/github-slug@v1 → c29c0ddd888a1703a6fc06c55a5f6ddd7beae490. Fixed script injection in 'Set up industrial_ci' step by moving `${{ inputs.build-context }}` into an `env:` block as `BUILD_CONTEXT` and referencing it as `"$BUILD_CONTEXT"` in the shell script.

### Iteration 2

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed two security issues in hardened/action/scripts/ci.sh:

1. github-env-injection (line 76): Sanitized `industrial_ci_image` before writing to GITHUB_OUTPUT by capturing it through `printf '%s' ... | tr -d '\n\r'` into `safe_industrial_ci_image`, then writing that sanitized value. This prevents newline injection into GITHUB_OUTPUT from attacker-controlled branch names or inputs.

2. script-injection (line 107): Replaced unquoted `${SLIM_BUILD_ARGS}` and `${ADDITIONAL_SLIM_BUILD_ARGS}` expansions with xargs-based quote-aware tokenization into bash arrays (`slim_args` and `additional_slim_args`). Each is guarded with `if [ -n "$VAR" ]` to prevent xargs from emitting an empty token on empty input. The arrays are expanded as `"${slim_args[@]}"` and `"${additional_slim_args[@]}"`, preserving argument boundaries and preventing shell metacharacter injection from attacker-controlled inputs.

