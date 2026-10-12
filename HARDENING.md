<!-- markdownlint-disable -->

# Hardening Report: ansible--ansible-content-actions/v0.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ansible--ansible-content-actions/v0.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple composite action run: blocks directly interpolate ${{ inputs.* }} and ${{ github.* }} expressions into shell commands (sub-rule a). The Actions runner substitutes these values before the shell parses them, enabling an attacker who controls the input to inject arbitrary shell commands.

- ansible_validate_changelog/action.yaml line 25: `python3 ${{ github.action_path }}/validate_changelog.py --ref ${{ inputs.base_ref }}`
- run-ansible-lint/action.yaml line 28: `if [[ -n "${{ inputs.working_directory }}" ]]`
- run-ansible-lint/action.yaml line 30: `echo "working_directory=${{ github.workspace }}"`
- run-ansible-lint/action.yaml line 48: `ansible-lint ${{ inputs.args }}`
- generate-tox-ansible-matrix/action.yaml line 42: `echo output=$(python -m tox ... --matrix-scope ${{ inputs.scope }} ...)`
- run-sanity/action.yaml line 26: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')`
- run-sanity/action.yaml line 38: `python -m tox --ansible -e ${{ inputs.test-env }}`
- run-unit-galaxy/action.yaml line 26: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')`
- run-unit-galaxy/action.yaml line 44: `python -m tox --ansible -e ${{ inputs.test-env }}`

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:25`
- `.github/actions/run-ansible-lint/action.yaml:28`
- `.github/actions/run-ansible-lint/action.yaml:48`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:42`
- `.github/actions/run-sanity/action.yaml:26`
- `.github/actions/run-sanity/action.yaml:38`
- `.github/actions/run-unit-galaxy/action.yaml:26`
- `.github/actions/run-unit-galaxy/action.yaml:44`

### github-env-injection (severity: high)

Values derived from untrusted inputs are written to $GITHUB_OUTPUT without sanitization (no `printf '%s' ... | tr -d '\n\r'` step).

- run-sanity/action.yaml line 27: `echo "py_version=$output" >> $GITHUB_OUTPUT` where $output is derived from `${{ inputs.test-env }}` on the previous line without sanitization.
- run-unit-galaxy/action.yaml line 27: same pattern — `echo "py_version=$output" >> $GITHUB_OUTPUT` where $output is derived from `${{ inputs.test-env }}`.
- generate-tox-ansible-matrix/action.yaml line 43: `echo "envlist=$output" >> $GITHUB_OUTPUT` where $output is derived from `${{ inputs.scope }}` on the previous line without sanitization.

An attacker-controlled newline in the input can inject arbitrary key=value pairs into the GitHub output environment.

Locations:

- `.github/actions/run-sanity/action.yaml:27`
- `.github/actions/run-unit-galaxy/action.yaml:27`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:43`

### unpinned-uses (severity: high)

All uses: references across the composite sub-actions use mutable version tags (e.g. @v3, @v4, @v5) instead of pinned 40-character SHA commit digests. This exposes the action to supply-chain attacks if the referenced tag is moved or the upstream repository is compromised.

Failing references:
- ansible_validate_changelog/action.yaml: `actions/setup-python@v4`
- generate-tox-ansible-matrix/action.yaml: `actions/github-script@v3`, `actions/setup-python@v4`
- run-ansible-lint/action.yaml: `actions/setup-python@v5`
- run-sanity/action.yaml: `actions/github-script@v3`, `actions/setup-python@v4`
- run-unit-galaxy/action.yaml: `actions/github-script@v3`, `actions/setup-python@v4`

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:18`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:21`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:27`
- `.github/actions/run-ansible-lint/action.yaml:36`
- `.github/actions/run-sanity/action.yaml:20`
- `.github/actions/run-sanity/action.yaml:31`
- `.github/actions/run-unit-galaxy/action.yaml:20`
- `.github/actions/run-unit-galaxy/action.yaml:32`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all 5 action files:

1. ansible_validate_changelog/action.yaml: Pinned setup-python@v4 to SHA. Moved github.action_path and inputs.base_ref into env: block (ACTION_PATH, BASE_REF).

2. run-ansible-lint/action.yaml: Pinned setup-python@v5 to SHA. Moved inputs.working_directory and github.workspace into env: block. For inputs.args (a list of arguments), used xargs tokenization with a bash array to safely pass arguments to ansible-lint.

3. generate-tox-ansible-matrix/action.yaml: Pinned github-script@v3 and setup-python@v4 to SHAs. Moved inputs.scope into env: block (INPUT_SCOPE). Added tr -d '\n\r' sanitization for both the scope input and the output before writing to GITHUB_OUTPUT.

4. run-sanity/action.yaml: Pinned github-script@v3 and setup-python@v4 to SHAs. Moved inputs.test-env into env: block (INPUT_TEST_ENV). Added tr -d '\n\r' sanitization for the input before use and the output before writing to GITHUB_OUTPUT. Also sanitized the final tox invocation by using the env var.

5. run-unit-galaxy/action.yaml: Same fixes as run-sanity — pinned SHAs, moved inputs.test-env to env: block, added sanitization for GITHUB_OUTPUT writes, and used env var in tox invocation.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Added '|| true' to the output assignment on line 50 of .github/actions/generate-tox-ansible-matrix/action.yaml. The bare assignment 'output=$(python -m tox ...)' would abort the step under errexit if the command failed. Upstream ran the command inside echo/printf which hid the exit status. Adding '|| true' preserves upstream's behavior where a non-zero exit from the python command doesn't abort the step, while still capturing the output in the variable.

