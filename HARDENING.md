<!-- markdownlint-disable -->

# Hardening Report: threeal--cmake-action/v2.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **threeal--cmake-action/v2.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across all three workflow files are pinned to mutable version tags rather than immutable 40-character SHA commit hashes. This exposes the workflows to supply-chain attacks if any referenced action's tag is moved or compromised. Affected references include: actions/checkout@v4.2.2, actions/setup-node@v4.1.0, threeal/setup-yarn-action@v2.0.0, threeal/ctest-action@v1.1.0, seanmiddleditch/gha-setup-ninja@v5. Each should be replaced with the corresponding full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2`.

Locations:

- `.github/workflows/build.yaml:12`
- `.github/workflows/build.yaml:16`
- `.github/workflows/build.yaml:20`
- `.github/workflows/check.yaml:12`
- `.github/workflows/check.yaml:16`
- `.github/workflows/check.yaml:20`
- `.github/workflows/test.yaml:13`
- `.github/workflows/test.yaml:16`
- `.github/workflows/test.yaml:21`
- `.github/workflows/test.yaml:119`
- `.github/workflows/test.yaml:131`

### permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` key, and none of the individual jobs define job-level `permissions:` keys. Without explicit permissions, workflows inherit the repository's default token permissions (which may be broad write-all). Each workflow file should declare minimal required permissions at the top level or per-job, e.g. `permissions: read-all` or specific scopes like `contents: read`.

Locations:

- `.github/workflows/build.yaml:1`
- `.github/workflows/check.yaml:1`
- `.github/workflows/test.yaml:1`

### script-injection (severity: high)

Two `run:` steps in test.yaml directly interpolate GitHub Actions expressions inside shell commands (sub-rule a). (1) Line 54: `run: ${{ steps.cmake-action.outputs.build-dir }}/${{ matrix.os == 'windows-2022' && 'Debug/generate_sequence.exe' || 'generate_sequence' }} 5` — both `steps.*.outputs.*` and `matrix.*` values are substituted into the shell command string by the Actions runner before the shell executes it, allowing a malicious output value to inject arbitrary shell commands. (2) Line 148: `run: ninja -C ${{ steps.cmake-action.outputs.build-dir }}` — same issue with `steps.cmake-action.outputs.build-dir`. These should be moved to `env:` variables and then referenced as double-quoted shell variables, e.g. `env: BUILD_DIR: ${{ steps.cmake-action.outputs.build-dir }}` and `run: ninja -C "$BUILD_DIR"`.

Locations:

- `.github/workflows/test.yaml:54`
- `.github/workflows/test.yaml:148`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all three findings across build.yaml, check.yaml, and test.yaml:

1. unpinned-uses: Pinned all 5 unique action references to their full 40-char SHA hashes (actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683, actions/setup-node@39370e3970a6d050c480ffad4ff0ed4d3fdee5af, threeal/setup-yarn-action@ec8c075e62bc497968de40011c2b766f5e8f1ac5, threeal/ctest-action@8f90c568a3060a3cffb896e241be14bd3f53d526, seanmiddleditch/gha-setup-ninja@96bed6edff20d1dd61ecff9b75cc519d516e6401) with version tag comments.

2. permissions: Added top-level `permissions: contents: read` to all three workflow files.

3. script-injection: Fixed two injection points in test.yaml — (a) 'Run Sample Project' step moved build-dir and matrix.os into env vars BUILD_DIR/RUNNER_OS and used shell if/else for OS-specific binary selection; (b) 'Build Sample Project' (ninja) step moved build-dir into env var BUILD_DIR and used `ninja -C "$BUILD_DIR"`.

