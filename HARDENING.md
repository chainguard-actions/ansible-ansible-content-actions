<!-- markdownlint-disable -->

# Hardening Report: ansible--ansible-content-actions/v1.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ansible--ansible-content-actions/v1.1.0** was hardened automatically. 9 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): ${{ inputs.base_ref }} and ${{ github.action_path }} are directly interpolated inside a run: shell command. An attacker-controlled base_ref value can inject arbitrary shell commands. Offending line: `python3 ${{ github.action_path }}/validate_changelog.py --ref ${{ inputs.base_ref }}`

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:24`

### script-injection (severity: high)

Rule (a): ${{ inputs.scope }} is directly interpolated inside a run: shell command. An attacker-controlled scope value can inject arbitrary shell commands. Offending line: `echo output=$(uv run tox --ansible --gh-matrix --matrix-scope ${{ inputs.scope }} --conf tox-ansible.ini)`

Locations:

- `.github/actions/generate-tox-ansible-matrix/action.yaml:43`

### script-injection (severity: high)

Rule (a): ${{ inputs.working_directory }}, ${{ github.workspace }}, and ${{ inputs.args }} are directly interpolated inside run: shell commands. Attacker-controlled values can inject arbitrary shell commands. Offending lines: `if [[ -n "${{ inputs.working_directory }}" ]]`, `echo "working_directory=${{ inputs.working_directory }}"`, `echo "working_directory=${{ github.workspace }}"`, and `ansible-lint ${{ inputs.args }}`

Locations:

- `.github/actions/run-ansible-lint/action.yaml:27`
- `.github/actions/run-ansible-lint/action.yaml:28`
- `.github/actions/run-ansible-lint/action.yaml:30`
- `.github/actions/run-ansible-lint/action.yaml:50`

### script-injection (severity: high)

Rule (a): ${{ inputs.test-env }} is directly interpolated inside run: shell commands. An attacker-controlled test-env value can inject arbitrary shell commands. Offending lines: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` and `python -m tox --ansible -e ${{ inputs.test-env }}`

Locations:

- `.github/actions/run-sanity/action.yaml:28`
- `.github/actions/run-sanity/action.yaml:41`

### script-injection (severity: high)

Rule (a): ${{ inputs.test-env }} is directly interpolated inside run: shell commands. An attacker-controlled test-env value can inject arbitrary shell commands. Offending lines: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` and `python -m tox --ansible -e ${{ inputs.test-env }}`

Locations:

- `.github/actions/run-unit-galaxy/action.yaml:28`
- `.github/actions/run-unit-galaxy/action.yaml:48`

### github-env-injection (severity: high)

inputs.scope is interpolated directly into a shell command that computes $output, which is then written to $GITHUB_OUTPUT without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). A newline in inputs.scope could inject additional key=value pairs into GITHUB_OUTPUT. Offending lines: `echo output=$(uv run tox ... --matrix-scope ${{ inputs.scope }} ...)` followed by `echo "envlist=$output" >> $GITHUB_OUTPUT`

Locations:

- `.github/actions/generate-tox-ansible-matrix/action.yaml:43`

### github-env-injection (severity: high)

inputs.test-env is interpolated directly into a shell command that computes $output, which is then written to $GITHUB_OUTPUT without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). A newline in inputs.test-env could inject additional key=value pairs into GITHUB_OUTPUT. Offending lines: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` followed by `echo "py_version=$output" >> $GITHUB_OUTPUT`

Locations:

- `.github/actions/run-sanity/action.yaml:28`

### github-env-injection (severity: high)

inputs.test-env is interpolated directly into a shell command that computes $output, which is then written to $GITHUB_OUTPUT without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). A newline in inputs.test-env could inject additional key=value pairs into GITHUB_OUTPUT. Offending lines: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` followed by `echo "py_version=$output" >> $GITHUB_OUTPUT`

Locations:

- `.github/actions/run-unit-galaxy/action.yaml:28`

### unpinned-uses (severity: high)

All uses: references in composite action steps use mutable version tags instead of immutable 40-character SHA commit hashes, making the action vulnerable to supply-chain attacks if the referenced tag is moved or overwritten. Failing references: `actions/setup-python@v6`, `actions/checkout@v6`, `actions/github-script@v8`, `astral-sh/setup-uv@v7`, `actions/setup-python@v6`, `actions/github-script@v8`, `actions/setup-python@v6`, `actions/github-script@v8`, `actions/setup-python@v6`

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:16`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:18`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:26`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:31`
- `.github/actions/run-ansible-lint/action.yaml:35`
- `.github/actions/run-sanity/action.yaml:19`
- `.github/actions/run-sanity/action.yaml:32`
- `.github/actions/run-unit-galaxy/action.yaml:19`
- `.github/actions/run-unit-galaxy/action.yaml:33`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all 9 findings across 5 action files:

1. ansible_validate_changelog/action.yaml: Pinned actions/setup-python@v6 to SHA. Moved github.action_path and inputs.base_ref into env: block to prevent script injection.

2. generate-tox-ansible-matrix/action.yaml: Pinned actions/checkout@v6, actions/github-script@v8, astral-sh/setup-uv@v7 to SHAs. Moved inputs.scope into env: block. Sanitized output with tr -d '\n\r' before writing to GITHUB_OUTPUT to prevent env injection.

3. run-ansible-lint/action.yaml: Pinned actions/setup-python@v6 to SHA. Moved inputs.working_directory and github.workspace into env: block. Used xargs tokenization idiom for inputs.args (a list-style argument input) to safely split into array.

4. run-sanity/action.yaml: Pinned actions/github-script@v8 and actions/setup-python@v6 to SHAs. Moved inputs.test-env into env: block as TEST_ENV. Used printf '%s' | cut | sed pipeline and sanitized with tr -d '\n\r' before writing py_version to GITHUB_OUTPUT.

5. run-unit-galaxy/action.yaml: Same fixes as run-sanity for inputs.test-env - pinned actions, moved to env: block, sanitized GITHUB_OUTPUT writes.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Fixed two issues in .github/actions/generate-tox-ansible-matrix/action.yaml:
1. Added `|| true` to the `raw=$(uv run tox ...)` assignment so a non-zero exit doesn't abort the step under errexit, matching upstream's behavior where the command was inside echo/printf which hid the exit status.
2. Removed the `safe=$(printf '%s' "$raw" | tr -d '\n\r')` and `echo "envlist=$safe" >> "$GITHUB_OUTPUT"` lines. Upstream's step never writes `envlist` to $GITHUB_OUTPUT directly — tox-ansible writes it itself — so the hardened version was incorrectly overwriting that value.

