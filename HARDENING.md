<!-- markdownlint-disable -->

# Hardening Report: ansible--ansible-content-actions/v0.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ansible--ansible-content-actions/v0.1.0** was hardened automatically. 13 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): ${{ inputs.base_ref }} and ${{ github.action_path }} are interpolated directly inside a run: shell command. An attacker-controlled base_ref value (e.g. containing shell metacharacters) is passed straight to the shell before any quoting occurs. The run: block is: `python3 ${{ github.action_path }}/validate_changelog.py --ref ${{ inputs.base_ref }}`

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:23`

### script-injection (severity: high)

Rule (a): ${{ inputs.working_directory }} and ${{ github.workspace }} are interpolated directly inside a run: shell command (inside an if/else block), and ${{ inputs.args }} is interpolated directly into the ansible-lint invocation. Attacker-controlled input values are passed to the shell without quoting. Offending lines: `if [[ -n "${{ inputs.working_directory }}" ]]`, `echo "working_directory=${{ inputs.working_directory }}"`, `echo "working_directory=${{ github.workspace }}"`, and `ansible-lint ${{ inputs.args }}`.

Locations:

- `.github/actions/run-ansible-lint/action.yaml:25`
- `.github/actions/run-ansible-lint/action.yaml:26`
- `.github/actions/run-ansible-lint/action.yaml:28`
- `.github/actions/run-ansible-lint/action.yaml:46`

### script-injection (severity: high)

Rule (a): ${{ inputs.scope }} is interpolated directly inside a run: shell command: `echo output=$(python -m tox --ansible --gh-matrix --matrix-scope ${{ inputs.scope }} --conf tox-ansible.ini)`. An attacker-controlled scope value containing shell metacharacters (`;`, `|`, `$(...)`, etc.) would be executed by the shell.

Locations:

- `.github/actions/generate-tox-ansible-matrix/action.yaml:43`

### script-injection (severity: high)

Rule (a): ${{ inputs.test-env }} is interpolated directly inside two run: shell commands: (1) `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` and (2) `python -m tox --ansible -e ${{ inputs.test-env }}`. An attacker-controlled test-env value containing shell metacharacters would be executed by the shell.

Locations:

- `.github/actions/run-sanity/action.yaml:27`
- `.github/actions/run-sanity/action.yaml:39`

### script-injection (severity: high)

Rule (a): ${{ inputs.test-env }} is interpolated directly inside two run: shell commands: (1) `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` and (2) `python -m tox --ansible -e ${{ inputs.test-env }}`. An attacker-controlled test-env value containing shell metacharacters would be executed by the shell.

Locations:

- `.github/actions/run-unit-galaxy/action.yaml:27`
- `.github/actions/run-unit-galaxy/action.yaml:46`

### github-env-injection (severity: high)

The run: block writes a value derived from ${{ inputs.scope }} (an untrusted input) to $GITHUB_OUTPUT without sanitization. The command `echo output=$(python -m tox ... --matrix-scope ${{ inputs.scope }} ...)` captures the command output and then `echo "envlist=$output" >> $GITHUB_OUTPUT` writes it. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write.

Locations:

- `.github/actions/generate-tox-ansible-matrix/action.yaml:44`

### github-env-injection (severity: high)

The run: block writes a value derived from ${{ inputs.test-env }} (an untrusted input) to $GITHUB_OUTPUT without sanitization. The command `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` then `echo "py_version=$output" >> $GITHUB_OUTPUT` writes the result. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write.

Locations:

- `.github/actions/run-sanity/action.yaml:28`

### github-env-injection (severity: high)

The run: block writes a value derived from ${{ inputs.test-env }} (an untrusted input) to $GITHUB_OUTPUT without sanitization. The command `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` then `echo "py_version=$output" >> $GITHUB_OUTPUT` writes the result. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write.

Locations:

- `.github/actions/run-unit-galaxy/action.yaml:28`

### unpinned-uses (severity: high)

Multiple uses: references use mutable version tags instead of full 40-character SHA digests, making the action vulnerable to supply-chain attacks if the tag is moved. Failing references: `actions/setup-python@v4` (line 17), `actions/github-script@v3` (line 21).

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:17`
- `.github/actions/ansible_validate_changelog/action.yaml:21`

### unpinned-uses (severity: high)

Multiple uses: references use mutable version tags instead of full 40-character SHA digests. Failing references: `actions/github-script@v3` (line 21), `actions/setup-python@v4` (line 27).

Locations:

- `.github/actions/generate-tox-ansible-matrix/action.yaml:21`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:27`

### unpinned-uses (severity: high)

uses: reference uses a mutable version tag instead of a full 40-character SHA digest. Failing reference: `actions/setup-python@v5` (line 33).

Locations:

- `.github/actions/run-ansible-lint/action.yaml:33`

### unpinned-uses (severity: high)

Multiple uses: references use mutable version tags instead of full 40-character SHA digests. Failing references: `actions/github-script@v3` (line 19), `actions/setup-python@v4` (line 33).

Locations:

- `.github/actions/run-sanity/action.yaml:19`
- `.github/actions/run-sanity/action.yaml:33`

### unpinned-uses (severity: high)

Multiple uses: references use mutable version tags instead of full 40-character SHA digests. Failing references: `actions/github-script@v3` (line 19), `actions/setup-python@v4` (line 33).

Locations:

- `.github/actions/run-unit-galaxy/action.yaml:19`
- `.github/actions/run-unit-galaxy/action.yaml:33`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all 5 action files:

1. ansible_validate_changelog/action.yaml: Pinned actions/setup-python@v4 to SHA, moved github.action_path and inputs.base_ref into env: block to prevent script injection.

2. run-ansible-lint/action.yaml: Pinned actions/setup-python@v5 to SHA, moved inputs.working_directory and github.workspace into env: block, and used xargs-based tokenization for inputs.args (a list-style input) to prevent script injection.

3. generate-tox-ansible-matrix/action.yaml: Pinned actions/github-script@v3 and actions/setup-python@v4 to SHAs, moved inputs.scope into env: block, and sanitized the tox output with tr -d '\n\r' before writing to GITHUB_OUTPUT.

4. run-sanity/action.yaml: Pinned actions/github-script@v3 and actions/setup-python@v4 to SHAs, moved inputs.test-env into env: block for both run steps, and sanitized py_version with tr -d '\n\r' before writing to GITHUB_OUTPUT.

5. run-unit-galaxy/action.yaml: Pinned actions/github-script@v3 and actions/setup-python@v4 to SHAs, moved inputs.test-env into env: block for both run steps, and sanitized py_version with tr -d '\n\r' before writing to GITHUB_OUTPUT.

