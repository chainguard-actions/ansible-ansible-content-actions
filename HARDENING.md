<!-- markdownlint-disable -->

# Hardening Report: ansible--ansible-content-actions/v0.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ansible--ansible-content-actions/v0.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ }} expressions into shell commands, enabling script injection. Rule (a) violations:

- ansible_validate_changelog/action.yaml line 24: `python3 ${{ github.action_path }}/validate_changelog.py --ref ${{ inputs.base_ref }}` — both github.action_path and inputs.base_ref are interpolated directly into the shell command.
- generate-tox-ansible-matrix/action.yaml line 44: `echo output=$(python -m tox --ansible --gh-matrix --matrix-scope ${{ inputs.scope }} --conf tox-ansible.ini)` — inputs.scope is interpolated directly.
- run-ansible-lint/action.yaml lines 24–26: `${{ inputs.working_directory }}` and `${{ github.workspace }}` interpolated in a run: block; line 44: `ansible-lint ${{ inputs.args }}` — inputs.args interpolated directly into the shell command.
- run-sanity/action.yaml line 28: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` and line 42: `python -m tox --ansible -e ${{ inputs.test-env }}` — inputs.test-env interpolated directly.
- run-unit-galaxy/action.yaml line 28: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` and line 49: `python -m tox --ansible -e ${{ inputs.test-env }}` — inputs.test-env interpolated directly.

An attacker controlling these inputs can inject arbitrary shell commands. Fix: move all ${{ }} values into env: variables and reference them as quoted shell variables (e.g., "$INPUT_ARGS").

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:24`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:44`
- `.github/actions/run-ansible-lint/action.yaml:24`
- `.github/actions/run-ansible-lint/action.yaml:44`
- `.github/actions/run-sanity/action.yaml:28`
- `.github/actions/run-sanity/action.yaml:42`
- `.github/actions/run-unit-galaxy/action.yaml:28`
- `.github/actions/run-unit-galaxy/action.yaml:49`

### github-env-injection (severity: high)

Three steps write values derived from untrusted inputs to $GITHUB_OUTPUT without the required sanitization step (printf '%s' "$VAR" | tr -d '\n\r'). An attacker can inject newlines into the output to poison subsequent steps that read from GITHUB_OUTPUT.

- generate-tox-ansible-matrix/action.yaml line 45: `echo "envlist=$output" >> $GITHUB_OUTPUT` where $output is derived from `${{ inputs.scope }}` on line 44 (itself a direct script-injection). No sanitization is applied before the write.
- run-sanity/action.yaml line 29: `echo "py_version=$output" >> $GITHUB_OUTPUT` where $output is derived from `${{ inputs.test-env }}` on line 28. No sanitization is applied.
- run-unit-galaxy/action.yaml line 29: `echo "py_version=$output" >> $GITHUB_OUTPUT` where $output is derived from `${{ inputs.test-env }}` on line 28. No sanitization is applied.

Fix: apply `safe=$(printf '%s' "$output" | tr -d '\n\r')` before each write to $GITHUB_OUTPUT.

Locations:

- `.github/actions/generate-tox-ansible-matrix/action.yaml:45`
- `.github/actions/run-sanity/action.yaml:29`
- `.github/actions/run-unit-galaxy/action.yaml:29`

### unpinned-uses (severity: high)

All uses: references in the composite action files use mutable version tags instead of full 40-character commit SHA digests. A compromised or hijacked tag can silently substitute malicious code.

Failing references:
- .github/actions/ansible_validate_changelog/action.yaml: `actions/setup-python@v4`
- .github/actions/generate-tox-ansible-matrix/action.yaml: `actions/github-script@v3`, `actions/setup-python@v4`
- .github/actions/run-ansible-lint/action.yaml: `actions/setup-python@v5`
- .github/actions/run-sanity/action.yaml: `actions/github-script@v3`, `actions/setup-python@v4`
- .github/actions/run-unit-galaxy/action.yaml: `actions/github-script@v3`, `actions/setup-python@v4`

Fix: pin every uses: reference to a full SHA, e.g. `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`.

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:17`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:22`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:27`
- `.github/actions/run-ansible-lint/action.yaml:32`
- `.github/actions/run-sanity/action.yaml:20`
- `.github/actions/run-sanity/action.yaml:33`
- `.github/actions/run-unit-galaxy/action.yaml:20`
- `.github/actions/run-unit-galaxy/action.yaml:34`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all 5 action files:

1. ansible_validate_changelog/action.yaml: Pinned setup-python@v4 to SHA; moved github.action_path and inputs.base_ref to env: vars.

2. generate-tox-ansible-matrix/action.yaml: Pinned github-script@v3 and setup-python@v4 to SHAs; moved inputs.scope to env: var; added sanitization (tr -d '\n\r') before writing to GITHUB_OUTPUT.

3. run-ansible-lint/action.yaml: Pinned setup-python@v5 to SHA; moved inputs.working_directory and github.workspace to env: vars in Show inputs step; moved inputs.args to env: var with xargs-based tokenization into an array for safe multi-argument expansion.

4. run-sanity/action.yaml: Pinned github-script@v3 and setup-python@v4 to SHAs; moved inputs.test-env to env: var in both the get-python-version step and Run tox step; added sanitization before writing py_version to GITHUB_OUTPUT.

5. run-unit-galaxy/action.yaml: Pinned github-script@v3 and setup-python@v4 to SHAs; moved inputs.test-env to env: var in both the get-python-version step and Run tox step; added sanitization before writing py_version to GITHUB_OUTPUT.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Fixed two issues in .github/actions/generate-tox-ansible-matrix/action.yaml: (1) Added `|| true` to the `output=$(python -m tox ...)` assignment on line 49 to preserve upstream's behavior where a non-zero exit was hidden (upstream ran the command inside echo/printf's argument substitution); (2) Removed the `safe=$(...)` sanitization and `echo "envlist=$safe" >> "$GITHUB_OUTPUT"` lines because upstream's step never writes `envlist` to $GITHUB_OUTPUT — tox-ansible writes it itself, and the hardened version was incorrectly overwriting that value.

