<!-- markdownlint-disable -->

# Hardening Report: ansible--ansible-content-actions/v0.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ansible--ansible-content-actions/v0.1.1** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Direct ${{ }} expression interpolation inside run: shell blocks. Sub-rule (a): inputs.* and github.* context values are interpolated directly into shell commands, allowing an attacker-controlled value to inject arbitrary shell commands.

- ansible_validate_changelog/action.yaml: `python3 ${{ github.action_path }}/validate_changelog.py --ref ${{ inputs.base_ref }}`
- generate-tox-ansible-matrix/action.yaml: `echo output=$(python -m tox --ansible --gh-matrix --matrix-scope ${{ inputs.scope }} --conf tox-ansible.ini)`
- run-ansible-lint/action.yaml: `if [[ -n "${{ inputs.working_directory }}" ]]`, `echo "working_directory=${{ github.workspace }}"`, and `ansible-lint ${{ inputs.args }}`
- run-sanity/action.yaml: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` and `python -m tox --ansible -e ${{ inputs.test-env }}`
- run-unit-galaxy/action.yaml: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` and `python -m tox --ansible -e ${{ inputs.test-env }}`

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:24`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:43`
- `.github/actions/run-ansible-lint/action.yaml:28`
- `.github/actions/run-ansible-lint/action.yaml:50`
- `.github/actions/run-sanity/action.yaml:28`
- `.github/actions/run-sanity/action.yaml:42`
- `.github/actions/run-unit-galaxy/action.yaml:28`
- `.github/actions/run-unit-galaxy/action.yaml:47`

### github-env-injection (severity: high)

Unsanitized writes to $GITHUB_OUTPUT derived from untrusted inputs. In each case, a shell variable ($output) is populated from a ${{ inputs.* }} expression interpolated directly in the run: block, then written to $GITHUB_OUTPUT without the required `printf '%s' ... | tr -d '\n\r'` sanitization step.

- generate-tox-ansible-matrix/action.yaml: `$output` is derived from `${{ inputs.scope }}` and written via `echo "envlist=$output" >> $GITHUB_OUTPUT`
- run-sanity/action.yaml: `$output` is derived from `${{ inputs.test-env }}` and written via `echo "py_version=$output" >> $GITHUB_OUTPUT`
- run-unit-galaxy/action.yaml: `$output` is derived from `${{ inputs.test-env }}` and written via `echo "py_version=$output" >> $GITHUB_OUTPUT`

Locations:

- `.github/actions/generate-tox-ansible-matrix/action.yaml:44`
- `.github/actions/run-sanity/action.yaml:29`
- `.github/actions/run-unit-galaxy/action.yaml:29`

### unpinned-uses (severity: high)

Multiple `uses:` references pin to mutable version tags instead of full 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the tag is moved or the referenced action is compromised.

Failing references:
- `uses: actions/github-script@v3` (generate-tox-ansible-matrix/action.yaml, run-sanity/action.yaml, run-unit-galaxy/action.yaml)
- `uses: actions/setup-python@v4` (ansible_validate_changelog/action.yaml, generate-tox-ansible-matrix/action.yaml, run-sanity/action.yaml, run-unit-galaxy/action.yaml)
- `uses: actions/setup-python@v5` (run-ansible-lint/action.yaml)

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:16`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:21`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:27`
- `.github/actions/run-ansible-lint/action.yaml:36`
- `.github/actions/run-sanity/action.yaml:20`
- `.github/actions/run-sanity/action.yaml:32`
- `.github/actions/run-unit-galaxy/action.yaml:21`
- `.github/actions/run-unit-galaxy/action.yaml:32`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all findings across 5 action files:

1. unpinned-uses: Pinned actions/setup-python@v4 to SHA 7f4fc3e22c37d6ff65e88745f38bd3157c663f7c, actions/setup-python@v5 to SHA a26af69be951a213d495a4c3e4e4022e16d87065, and actions/github-script@v3 to SHA ffc2c79a5b2490bd33e0a41c1de74b877714d736 in all affected files.

2. script-injection: Moved all ${{ }} expressions from run: blocks into env: blocks and referenced them as plain shell variables. For inputs.args in run-ansible-lint (an argument list), used the xargs tokenization pattern to preserve argument boundaries.

3. github-env-injection: Added printf '%s' ... | tr -d '\n\r' sanitization before writing derived values to $GITHUB_OUTPUT in generate-tox-ansible-matrix, run-sanity, and run-unit-galaxy. Used a two-step capture (raw=...; output=$(printf '%s' "$raw" | tr -d '\n\r')) to preserve errexit behavior.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Fixed two issues in .github/actions/generate-tox-ansible-matrix/action.yaml:
1. Added `|| true` to the `raw=$(python -m tox ...)` assignment so that a non-zero exit from tox doesn't abort the step under errexit, matching upstream's behavior where the command was run inside printf/echo which hid its exit status.
2. Removed the `echo "envlist=$output" >> "$GITHUB_OUTPUT"` line (and the now-unnecessary `output=` sanitization line) because tox-ansible writes `envlist` to $GITHUB_OUTPUT itself; the explicit write was incorrectly replacing that value.

