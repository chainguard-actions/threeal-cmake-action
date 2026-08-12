<!-- markdownlint-disable -->

# Hardening Report: threeal--cmake-action/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **threeal--cmake-action/v1.2.0** was hardened automatically. 32 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Process the inputs' step in action.yml directly interpolates ${{ inputs.* }} expressions inside the run: shell script. Every input (source-dir, build-dir, generator, c-compiler, cxx-compiler, c-flags, cxx-flags, options, args, run-build, build-args, run-test, test-args) is substituted verbatim into the shell before execution. An attacker-controlled input value containing shell metacharacters (e.g. '; malicious_cmd #') will be executed by the shell. Offending lines include: `if [ -n '${{ inputs.source-dir }}' ]`, `SOURCE_DIR="${{ inputs.source-dir }}"`, `BUILD_DIR="${{ inputs.source-dir }}/build"`, `ARGS="$ARGS -G '${{ inputs.generator }}'"`, `for OPT in ${{ inputs.options }}`, etc. All inputs must be passed via env: variables and then referenced as quoted shell variables.

Locations:

- `action.yml:50`
- `action.yml:51`
- `action.yml:55`
- `action.yml:56`
- `action.yml:57`
- `action.yml:61`
- `action.yml:63`
- `action.yml:66`
- `action.yml:68`
- `action.yml:71`
- `action.yml:73`
- `action.yml:76`
- `action.yml:78`
- `action.yml:81`
- `action.yml:83`
- `action.yml:85`
- `action.yml:87`
- `action.yml:91`
- `action.yml:93`
- `action.yml:97`
- `action.yml:99`
- `action.yml:103`
- `action.yml:105`

### script-injection (severity: high)

The 'Configure the CMake project', 'Build targets', and 'Run tests' steps in action.yml directly interpolate ${{ steps.process_inputs.outputs.cmake_args }}, ${{ steps.process_inputs.outputs.cmake_build_args }}, and ${{ steps.process_inputs.outputs.cmake_test_args }} into run: shell commands. These outputs are assembled from user-controlled inputs.* values and are injected directly into the shell command line without quoting or sanitization, enabling command injection. Example: `run: cmake ${{ steps.process_inputs.outputs.cmake_args }}`.

Locations:

- `action.yml:114`
- `action.yml:119`
- `action.yml:124`

### script-injection (severity: high)

The workflow file .github/workflows/test.yml directly interpolates ${{ matrix.os == 'windows' && '-C Debug' || '' }} and ${{ matrix.compiler == 'msvc' && '-C Debug' || '' }} expressions inside run: shell command strings. Any ${{ ... }} expression in a run: block is a script-injection risk because YAML template substitution occurs before the shell parses the command. Offending lines: `ctest ... -R hello_world ${{ matrix.os == 'windows' && '-C Debug' || '' }}` (appears twice) and `test-args: -R test ${{ matrix.compiler == 'msvc' && '-C Debug' || '' }}`.

Locations:

- `.github/workflows/test.yml:26`
- `.github/workflows/test.yml:34`

### github-env-injection (severity: high)

The 'Process the inputs' step in action.yml writes values derived from user-controlled inputs.* to $GITHUB_OUTPUT without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). The bash parameter expansion `${ARGS//[$'\t\r\n']}` used inline is not the required sanitization pattern and does not prevent newline injection via GITHUB_OUTPUT. Three writes are affected: `echo "cmake_args=${ARGS//[$'\t\r\n']}" >> $GITHUB_OUTPUT`, `echo "cmake_build_args=${BUILD_ARGS//[$'\t\r\n']}" >> $GITHUB_OUTPUT`, and `echo "cmake_test_args=${TEST_ARGS//[$'\t\r\n']}" >> $GITHUB_OUTPUT`. Each ARGS variable is built from inputs.* values interpolated directly into the shell script. A newline embedded in an input value can inject additional key=value pairs into GITHUB_OUTPUT.

Locations:

- `action.yml:89`
- `action.yml:101`
- `action.yml:107`

### unpinned-uses (severity: high)

All uses: references in .github/workflows/test.yml use a mutable version tag (@v3.5.3) instead of a full 40-character commit SHA. This means the action could be silently replaced with a malicious version. Affected references: `actions/checkout@v3.5.3` appears 6 times across all jobs (default-usage, specified-dir-usage, run-build-usage, run-test-usage, additional-flags-usage, specified-generator-and-compiler-usage). Each should be pinned to a full SHA, e.g. `actions/checkout@f43a0e5ff2bd294095638e18286ca9a3d1956744 # v3.5.3`.

Locations:

- `.github/workflows/test.yml:14`
- `.github/workflows/test.yml:41`
- `.github/workflows/test.yml:57`
- `.github/workflows/test.yml:70`
- `.github/workflows/test.yml:84`
- `.github/workflows/test.yml:100`

### missing-permissions (severity: medium)

The workflow file .github/workflows/test.yml has no top-level `permissions:` key and no job-level `permissions:` key in any of its jobs (default-usage, specified-dir-usage, run-build-usage, run-test-usage, additional-flags-usage, specified-generator-and-compiler-usage). Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. A minimal `permissions: {}` or specific scopes (e.g. `contents: read`) should be declared.

Locations:

- `.github/workflows/test.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.source-dir }}" appears directly in run: block of step "Process the inputs"; move to env: map

Locations:

- `action.yml:57`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.source-dir }}" appears directly in run: block of step "Process the inputs"; move to env: map

Locations:

- `action.yml:58`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.build-dir }}" appears directly in run: block of step "Process the inputs"; move to env: map

Locations:

- `action.yml:62`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.build-dir }}" appears directly in run: block of step "Process the inputs"; move to env: map

Locations:

- `action.yml:63`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.source-dir }}" appears directly in run: block of step "Process the inputs"; move to env: map

Locations:

- `action.yml:64`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.source-dir }}" appears directly in run: block of step "Process the inputs"; move to env: map

Locations:

- `action.yml:65`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.generator }}" appears directly in run: block of step "Process the inputs"; move to env: map

Locations:

- `action.yml:69`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.generator }}" appears directly in run: block of step "Process the inputs"; move to env: map

Locations:

- `action.yml:70`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.c-compiler }}" appears directly in run: block of step "Process the inputs"; move to env: map

Locations:

- `action.yml:72`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.c-compiler }}" appears directly in run: block of step "Process the inputs"; move to env: map

Locations:

- `action.yml:73`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cxx-compiler }}" appears directly in run: block of step "Process the inputs"; move to env: map

Locations:

- `action.yml:75`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cxx-compiler }}" appears directly in run: block of step "Process the inputs"; move to env: map

Locations:

- `action.yml:76`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.c-flags }}" appears directly in run: block of step "Process the inputs"; move to env: map

Locations:

- `action.yml:78`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.c-flags }}" appears directly in run: block of step "Process the inputs"; move to env: map

Locations:

- `action.yml:79`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cxx-flags }}" appears directly in run: block of step "Process the inputs"; move to env: map

Locations:

- `action.yml:81`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cxx-flags }}" appears directly in run: block of step "Process the inputs"; move to env: map

Locations:

- `action.yml:82`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.options }}" appears directly in run: block of step "Process the inputs"; move to env: map

Locations:

- `action.yml:84`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.args }}" appears directly in run: block of step "Process the inputs"; move to env: map

Locations:

- `action.yml:87`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.args }}" appears directly in run: block of step "Process the inputs"; move to env: map

Locations:

- `action.yml:88`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.run-build }}" appears directly in run: block of step "Process the inputs"; move to env: map

Locations:

- `action.yml:92`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.run-test }}" appears directly in run: block of step "Process the inputs"; move to env: map

Locations:

- `action.yml:92`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.build-args }}" appears directly in run: block of step "Process the inputs"; move to env: map

Locations:

- `action.yml:94`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.build-args }}" appears directly in run: block of step "Process the inputs"; move to env: map

Locations:

- `action.yml:95`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.run-test }}" appears directly in run: block of step "Process the inputs"; move to env: map

Locations:

- `action.yml:100`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.test-args }}" appears directly in run: block of step "Process the inputs"; move to env: map

Locations:

- `action.yml:102`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.test-args }}" appears directly in run: block of step "Process the inputs"; move to env: map

Locations:

- `action.yml:103`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions, static-inline-injection

**Notes:**

Fixed all findings across action.yml and .github/workflows/test.yml:

1. action.yml - script-injection & static-inline-injection: Moved all ${{ inputs.* }} expressions to an env: block in the 'Process the inputs' step. Shell script now uses $INPUT_* env vars. List inputs (options, args, build-args, test-args) are tokenized with xargs for proper quote-aware splitting. Args are serialized with printf '%q' (shell-quoting) for safe eval in later steps. Later steps (Configure, Build, Run tests) use env: to receive step outputs and eval to safely expand the shell-quoted args. Changed shell from pwsh/bash conditional to bash for consistency.

2. action.yml - github-env-injection: GITHUB_OUTPUT writes now use 'printf \'%s\' "$VAR" | tr -d \'\n\r\'' sanitization as required, replacing the inline bash parameter expansion.

3. test.yml - script-injection: Moved ${{ matrix.os == 'windows' && '-C Debug' || '' }} expressions from run: blocks to env: vars with conditional shell logic. Fixed test-args by consolidating into a single ternary expression in the with: block (safe context).

4. test.yml - unpinned-uses: Pinned all 6 actions/checkout@v3.5.3 references to full SHA c85c95e3d7251135ab7dc9ce3241c5835cc595a9.

5. test.yml - missing-permissions: Added 'permissions: contents: read' at the top level of the workflow.

