<!-- markdownlint-disable -->

# Hardening Report: threeal--cmake-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **threeal--cmake-action/v2.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files use tag-based (non-SHA) `uses:` references, making the workflows vulnerable to supply-chain attacks if a tag is moved. Failing references include: `actions/checkout@v4.1.7`, `actions/setup-node@v4.0.3`, `threeal/setup-yarn-action@v2.0.0`, `threeal/ctest-action@v1.1.0`, `seanmiddleditch/gha-setup-ninja@v5`. All should be pinned to full 40-character commit SHAs.

Locations:

- `.github/workflows/build.yaml:12`
- `.github/workflows/build.yaml:16`
- `.github/workflows/build.yaml:20`
- `.github/workflows/check.yaml:12`
- `.github/workflows/check.yaml:16`
- `.github/workflows/check.yaml:20`
- `.github/workflows/test.yaml:13`
- `.github/workflows/test.yaml:16`
- `.github/workflows/test.yaml:20`
- `.github/workflows/test.yaml:131`
- `.github/workflows/test.yaml:148`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` block, and no individual job within them defines job-level `permissions:` either. Without explicit permissions, workflows inherit the default repository permissions (which may include `write` access to contents, packages, etc.), violating the principle of least privilege.

Locations:

- `.github/workflows/build.yaml:1`
- `.github/workflows/check.yaml:1`
- `.github/workflows/test.yaml:1`

### script-injection (severity: high)

Two `run:` blocks in test.yaml directly interpolate `${{ ... }}` expressions into shell commands (sub-rule a). This causes the expression value to be substituted into the shell command string before the shell parses it, enabling injection of shell metacharacters.

1. Line 55: `run: ${{ steps.cmake-action.outputs.build-dir }}/${{ matrix.os == 'windows' && 'Debug/generate_sequence.exe' || 'generate_sequence' }} 5` — both `steps.cmake-action.outputs.build-dir` and `matrix.os` are interpolated directly.

2. Line 161: `run: ninja -C ${{ steps.cmake-action.outputs.build-dir }}` — `steps.cmake-action.outputs.build-dir` is interpolated directly into the shell command.

Fix: move the values into `env:` variables and reference them as double-quoted shell variables (e.g., `"$BUILD_DIR"`).

Locations:

- `.github/workflows/test.yaml:55`
- `.github/workflows/test.yaml:161`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across build.yaml, check.yaml, and test.yaml:

1. unpinned-uses: Pinned all 5 action references to full 40-char SHAs (actions/checkout@692973e3, actions/setup-node@1e60f620, threeal/setup-yarn-action@ec8c075e, threeal/ctest-action@8f90c568, seanmiddleditch/gha-setup-ninja@96bed6ed), preserving original tags as comments.

2. missing-permissions: Added `permissions: {}` top-level block to all three workflow files.

3. script-injection: Fixed two injection points in test.yaml — (a) the 'Run Sample Project' step in test-action job now uses env vars BUILD_DIR and MATRIX_OS with a shell if/else instead of inline ${{ }} expressions; (b) the 'Build Sample Project' step in test-action-with-custom-generator now uses env var BUILD_DIR with `ninja -C "$BUILD_DIR"` instead of inline ${{ steps.cmake-action.outputs.build-dir }}.

