<!-- markdownlint-disable -->

# Hardening Report: ansible--ansible-content-actions/v1.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ansible--ansible-content-actions/v1.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple composite actions use mutable tag-based `uses:` references instead of pinned 40-character SHA digests, making them vulnerable to supply-chain attacks if the referenced tag is moved or overwritten.

Failing references:
- `.github/actions/ansible_validate_changelog/action.yaml`: `actions/setup-python@v6`
- `.github/actions/generate-tox-ansible-matrix/action.yaml`: `actions/checkout@v6`, `actions/github-script@v8`, `astral-sh/setup-uv@v7`
- `.github/actions/run-ansible-lint/action.yaml`: `actions/setup-python@v6`
- `.github/actions/run-sanity/action.yaml`: `actions/github-script@v8`, `actions/setup-python@v6`
- `.github/actions/run-unit-galaxy/action.yaml`: `actions/github-script@v8`, `actions/setup-python@v6`

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:17`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:18`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:24`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:29`
- `.github/actions/run-ansible-lint/action.yaml:33`
- `.github/actions/run-sanity/action.yaml:18`
- `.github/actions/run-sanity/action.yaml:30`
- `.github/actions/run-unit-galaxy/action.yaml:18`
- `.github/actions/run-unit-galaxy/action.yaml:31`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ inputs.* }}` and `${{ github.* }}` expressions inside shell command strings (rule a). This allows an attacker who controls the input values to inject arbitrary shell commands.

Offending lines:
- `ansible_validate_changelog/action.yaml` line 24: `python3 ${{ github.action_path }}/validate_changelog.py --ref ${{ inputs.base_ref }}` — both `github.action_path` and `inputs.base_ref` are interpolated directly into the shell command.
- `generate-tox-ansible-matrix/action.yaml` line 43: `echo output=$(uv run tox --ansible --gh-matrix --matrix-scope ${{ inputs.scope }} --conf tox-ansible.ini)` — `inputs.scope` interpolated unquoted.
- `run-ansible-lint/action.yaml` line 26: `if [[ -n "${{ inputs.working_directory }}" ]]` and line 28: `echo "working_directory=${{ inputs.working_directory }}"` / `${{ github.workspace }}` — inputs interpolated inside run block.
- `run-ansible-lint/action.yaml` line 46: `ansible-lint ${{ inputs.args }}` — `inputs.args` interpolated directly, allowing arbitrary argument injection.
- `run-sanity/action.yaml` line 27: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` — `inputs.test-env` interpolated unquoted inside command substitution.
- `run-sanity/action.yaml` line 40: `python -m tox --ansible -e ${{ inputs.test-env }}` — direct interpolation.
- `run-unit-galaxy/action.yaml` line 27: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` — same as run-sanity.
- `run-unit-galaxy/action.yaml` line 46: `python -m tox --ansible -e ${{ inputs.test-env }}` — direct interpolation.

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:24`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:43`
- `.github/actions/run-ansible-lint/action.yaml:26`
- `.github/actions/run-ansible-lint/action.yaml:46`
- `.github/actions/run-sanity/action.yaml:27`
- `.github/actions/run-sanity/action.yaml:40`
- `.github/actions/run-unit-galaxy/action.yaml:27`
- `.github/actions/run-unit-galaxy/action.yaml:46`

### github-env-injection (severity: high)

Several `run:` blocks write values derived from untrusted `inputs.*` to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker can inject newlines into the input to poison the output file and set arbitrary environment variables or outputs for subsequent steps.

Offending steps:
- `run-sanity/action.yaml` line 28: `echo "py_version=$output" >> $GITHUB_OUTPUT` — `$output` is derived from `${{ inputs.test-env }}` (interpolated on line 27) without any newline stripping before the write.
- `run-unit-galaxy/action.yaml` line 28: `echo "py_version=$output" >> $GITHUB_OUTPUT` — same pattern, `$output` derived from `${{ inputs.test-env }}`.
- `generate-tox-ansible-matrix/action.yaml` line 44: `echo "envlist=$output" >> $GITHUB_OUTPUT` — `$output` is derived from a command that includes the unquoted `${{ inputs.scope }}` value (line 43), with no sanitization before the write.

Locations:

- `.github/actions/run-sanity/action.yaml:28`
- `.github/actions/run-unit-galaxy/action.yaml:28`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all 5 action files:

1. ansible_validate_changelog/action.yaml: Pinned actions/setup-python@v6 to SHA ece7cb06...; moved github.action_path and inputs.base_ref into env: block as ACTION_PATH and BASE_REF.

2. generate-tox-ansible-matrix/action.yaml: Pinned actions/checkout@v6 (d23441a4...), actions/github-script@v8 (ed597411...), astral-sh/setup-uv@v7 (37802adc...); moved inputs.scope into env: as SCOPE; sanitized $output with tr -d '\n\r' before writing to GITHUB_OUTPUT.

3. run-ansible-lint/action.yaml: Pinned actions/setup-python@v6 (ece7cb06...); moved inputs.working_directory, github.workspace into env:; moved inputs.args into env: as INPUT_ARGS and used xargs tokenization (list-style arg) to safely pass to ansible-lint.

4. run-sanity/action.yaml: Pinned actions/github-script@v8 (ed597411...) and actions/setup-python@v6 (ece7cb06...); moved inputs.test-env into env: as TEST_ENV in both steps; sanitized output with tr -d '\n\r' before writing py_version to GITHUB_OUTPUT.

5. run-unit-galaxy/action.yaml: Pinned actions/github-script@v8 (ed597411...) and actions/setup-python@v6 (ece7cb06...); moved inputs.test-env into env: as TEST_ENV in both steps; sanitized output with tr -d '\n\r' before writing py_version to GITHUB_OUTPUT.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Fixed .github/actions/generate-tox-ansible-matrix/action.yaml line 50: added '|| true' to the output assignment 'output=$(uv run tox --ansible --gh-matrix --matrix-scope "$SCOPE" --conf tox-ansible.ini) || true'. This preserves upstream's behavior where the command was run inside echo/printf which hid its exit status, preventing the step from aborting under bash errexit when the uv command exits non-zero.

