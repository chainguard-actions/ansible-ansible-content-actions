<!-- markdownlint-disable -->

# Hardening Report: ansible--ansible-content-actions/v1.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ansible--ansible-content-actions/v1.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks interpolate ${{ }} expressions directly into shell commands, violating rule (a). This allows an attacker-controlled value to be interpreted as shell code before the shell ever sees it.

1. `.github/actions/ansible_validate_changelog/action.yaml` — `run: python3 ${{ github.action_path }}/validate_changelog.py --ref ${{ inputs.base_ref }}` — both `github.action_path` and `inputs.base_ref` are interpolated directly.

2. `.github/actions/generate-tox-ansible-matrix/action.yaml` — `run: echo output=$(uv run tox --ansible --gh-matrix --matrix-scope ${{ inputs.scope }} --conf tox-ansible.ini)` — `inputs.scope` is interpolated directly.

3. `.github/actions/run-ansible-lint/action.yaml` — `run: if [[ -n "${{ inputs.working_directory }}" ]]` and `echo "working_directory=${{ inputs.working_directory }}"` and `echo "working_directory=${{ github.workspace }}"` — inputs and github context interpolated directly; also `run: ansible-lint ${{ inputs.args }}` — `inputs.args` interpolated directly.

4. `.github/actions/run-sanity/action.yaml` — `run: output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` and `run: python -m tox --ansible -e ${{ inputs.test-env }}` — `inputs.test-env` interpolated directly in both steps.

5. `.github/actions/run-unit-galaxy/action.yaml` — `run: output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` and `run: python -m tox --ansible -e ${{ inputs.test-env }}` — `inputs.test-env` interpolated directly in both steps.

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:24`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:44`
- `.github/actions/run-ansible-lint/action.yaml:22`
- `.github/actions/run-ansible-lint/action.yaml:44`
- `.github/actions/run-sanity/action.yaml:27`
- `.github/actions/run-sanity/action.yaml:43`
- `.github/actions/run-unit-galaxy/action.yaml:27`
- `.github/actions/run-unit-galaxy/action.yaml:55`

### github-env-injection (severity: high)

Several run: blocks write values derived from untrusted inputs (${{ inputs.* }}) to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

1. `.github/actions/generate-tox-ansible-matrix/action.yaml` — `inputs.scope` is interpolated into the shell command that computes `$output`, and then `echo "envlist=$output" >> $GITHUB_OUTPUT` writes it unsanitized.

2. `.github/actions/run-sanity/action.yaml` — `inputs.test-env` is interpolated into the command computing `$output`, and then `echo "py_version=$output" >> $GITHUB_OUTPUT` writes it unsanitized.

3. `.github/actions/run-unit-galaxy/action.yaml` — same pattern: `inputs.test-env` flows into `$output` which is written to `$GITHUB_OUTPUT` without sanitization.

Locations:

- `.github/actions/generate-tox-ansible-matrix/action.yaml:44`
- `.github/actions/run-sanity/action.yaml:27`
- `.github/actions/run-unit-galaxy/action.yaml:27`

### unpinned-uses (severity: high)

All `uses:` references across the composite action files use mutable version tags instead of immutable 40-character commit SHA hashes. This exposes the action to supply-chain attacks where a tag is moved to point to malicious code.

Failing references:
- `actions/setup-python@v4` (ansible_validate_changelog/action.yaml, run-sanity/action.yaml, run-unit-galaxy/action.yaml)
- `actions/checkout@v4` (generate-tox-ansible-matrix/action.yaml)
- `actions/github-script@v3` (generate-tox-ansible-matrix/action.yaml, run-sanity/action.yaml, run-unit-galaxy/action.yaml)
- `astral-sh/setup-uv@v5` (generate-tox-ansible-matrix/action.yaml)
- `actions/setup-python@v5` (run-ansible-lint/action.yaml)

All should be pinned to full SHA digests, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:17`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:16`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:22`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:27`
- `.github/actions/run-ansible-lint/action.yaml:30`
- `.github/actions/run-sanity/action.yaml:18`
- `.github/actions/run-sanity/action.yaml:24`
- `.github/actions/run-unit-galaxy/action.yaml:18`
- `.github/actions/run-unit-galaxy/action.yaml:24`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all 5 action files:

1. ansible_validate_changelog/action.yaml: Pinned actions/setup-python@v4 to SHA. Moved github.action_path and inputs.base_ref into env: block as ACTION_PATH and BASE_REF.

2. generate-tox-ansible-matrix/action.yaml: Pinned actions/checkout@v4, actions/github-script@v3, and astral-sh/setup-uv@v5 to SHAs. Moved inputs.scope into env: block as SCOPE. Added sanitization (tr -d '\n\r') before writing to GITHUB_OUTPUT.

3. run-ansible-lint/action.yaml: Pinned actions/setup-python@v5 to SHA. Moved inputs.working_directory and github.workspace into env: block. Moved inputs.args into env: block as INPUT_ARGS and tokenized it with xargs into a bash array (since it's a list-style argument input).

4. run-sanity/action.yaml: Pinned actions/github-script@v3 and actions/setup-python@v4 to SHAs. Moved inputs.test-env into env: block as TEST_ENV in both steps. Added sanitization before writing py_version to GITHUB_OUTPUT.

5. run-unit-galaxy/action.yaml: Pinned actions/github-script@v3 and actions/setup-python@v4 to SHAs. Moved inputs.test-env into env: block as TEST_ENV in both steps. Added sanitization before writing py_version to GITHUB_OUTPUT.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Fixed two issues in .github/actions/generate-tox-ansible-matrix/action.yaml: (1) Added '|| true' to the 'raw=$(uv run tox ...)' assignment so a non-zero exit doesn't abort the step under errexit, matching upstream's behavior where the command ran inside echo/printf which hid its exit status. (2) Removed 'echo "envlist=$output" >> "$GITHUB_OUTPUT"' because upstream's step never writes envlist to GITHUB_OUTPUT directly — tox-ansible writes it itself during execution, and the hardened version was incorrectly overwriting that value.

