<!-- markdownlint-disable -->

# Hardening Report: ansible--ansible-content-actions/v0.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ansible--ansible-content-actions/v0.1.1** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ }} expressions into shell commands, enabling script injection. An attacker-controlled input value can break out of the shell command context and execute arbitrary code.

- ansible_validate_changelog/action.yaml: `python3 ${{ github.action_path }}/validate_changelog.py --ref ${{ inputs.base_ref }}` — both github.action_path and inputs.base_ref are interpolated directly into the shell command (sub-rule a).
- generate-tox-ansible-matrix/action.yaml: `--matrix-scope ${{ inputs.scope }}` — inputs.scope is interpolated directly into the shell command (sub-rule a).
- run-ansible-lint/action.yaml: `if [[ -n "${{ inputs.working_directory }}" ]]`, `echo "working_directory=${{ inputs.working_directory }}"`, `echo "working_directory=${{ github.workspace }}"`, and `ansible-lint ${{ inputs.args }}` — inputs.* and github.* are interpolated directly into run: blocks (sub-rule a).
- run-sanity/action.yaml: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` and `python -m tox --ansible -e ${{ inputs.test-env }}` — inputs.test-env is interpolated directly (sub-rule a).
- run-unit-galaxy/action.yaml: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` and `python -m tox --ansible -e ${{ inputs.test-env }}` — inputs.test-env is interpolated directly (sub-rule a).

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:24`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:47`
- `.github/actions/run-ansible-lint/action.yaml:23`
- `.github/actions/run-ansible-lint/action.yaml:46`
- `.github/actions/run-sanity/action.yaml:27`
- `.github/actions/run-sanity/action.yaml:44`
- `.github/actions/run-unit-galaxy/action.yaml:27`
- `.github/actions/run-unit-galaxy/action.yaml:55`

### github-env-injection (severity: high)

Three run: blocks write values derived from ${{ inputs.* }} expressions to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker-supplied newline in the input can inject arbitrary key=value pairs into the GitHub output environment.

- generate-tox-ansible-matrix/action.yaml: `${{ inputs.scope }}` is interpolated into the shell command that sets `$output`, which is then written via `echo "envlist=$output" >> $GITHUB_OUTPUT` with no sanitization.
- run-sanity/action.yaml: `${{ inputs.test-env }}` is interpolated into `output=$(echo ${{ inputs.test-env }} | ...)`, and `$output` is written via `echo "py_version=$output" >> $GITHUB_OUTPUT` with no sanitization.
- run-unit-galaxy/action.yaml: Same pattern as run-sanity — `${{ inputs.test-env }}` flows into `$output` which is written to `$GITHUB_OUTPUT` without sanitization.

Locations:

- `.github/actions/generate-tox-ansible-matrix/action.yaml:48`
- `.github/actions/run-sanity/action.yaml:28`
- `.github/actions/run-unit-galaxy/action.yaml:28`

### unpinned-uses (severity: high)

All uses: references across the composite action files use mutable tag refs (@v3, @v4, @v5) instead of immutable 40-character SHA commit hashes. This exposes the action to supply-chain attacks where a compromised upstream action tag could silently execute malicious code.

Failing references:
- actions/setup-python@v4 (ansible_validate_changelog, generate-tox-ansible-matrix, run-sanity, run-unit-galaxy)
- actions/github-script@v3 (generate-tox-ansible-matrix, run-sanity, run-unit-galaxy)
- actions/setup-python@v5 (run-ansible-lint)
- actions/setup-python@v4 (run-ansible-lint)

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:17`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:20`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:27`
- `.github/actions/run-ansible-lint/action.yaml:31`
- `.github/actions/run-sanity/action.yaml:20`
- `.github/actions/run-sanity/action.yaml:33`
- `.github/actions/run-unit-galaxy/action.yaml:20`
- `.github/actions/run-unit-galaxy/action.yaml:34`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all 5 action files:

1. ansible_validate_changelog/action.yaml: Pinned actions/setup-python@v4 to SHA. Moved github.action_path and inputs.base_ref into env: block as ACTION_PATH and BASE_REF.

2. generate-tox-ansible-matrix/action.yaml: Pinned actions/github-script@v3 and actions/setup-python@v4 to SHAs. Moved inputs.scope into env: block as MATRIX_SCOPE. Added sanitization (tr -d '\n\r') before writing to $GITHUB_OUTPUT.

3. run-ansible-lint/action.yaml: Pinned actions/setup-python@v5 to SHA. Moved inputs.working_directory and github.workspace into env: block. Moved inputs.args into env: block and used xargs tokenization (since it's a list-style argument input) to safely expand it as separate arguments to ansible-lint.

4. run-sanity/action.yaml: Pinned actions/github-script@v3 and actions/setup-python@v4 to SHAs. Moved inputs.test-env into env: block as TEST_ENV in both steps. Added two-step capture (raw then sanitized output) before writing py_version to $GITHUB_OUTPUT.

5. run-unit-galaxy/action.yaml: Pinned actions/github-script@v3 and actions/setup-python@v4 to SHAs. Moved inputs.test-env into env: block as TEST_ENV in both steps. Added two-step capture (raw then sanitized output) before writing py_version to $GITHUB_OUTPUT.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Fixed .github/actions/generate-tox-ansible-matrix/action.yaml line 49: added '|| true' to the bare assignment 'output=$(python -m tox --ansible --gh-matrix --matrix-scope "$MATRIX_SCOPE" --conf tox-ansible.ini)' to preserve upstream's behavior where the command was run inside echo/printf "$(...)", hiding the command's exit status and allowing the step to continue even on non-zero exit.

