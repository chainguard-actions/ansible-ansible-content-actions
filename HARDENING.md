<!-- markdownlint-disable -->

# Hardening Report: ansible--ansible-content-actions/v1.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ansible--ansible-content-actions/v1.0.0** was hardened automatically. 13 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation of `${{ inputs.* }}` and `${{ github.* }}` inside `run:` shell blocks. In ansible_validate_changelog/action.yaml the 'Validate changelog' step runs: `python3 ${{ github.action_path }}/validate_changelog.py --ref ${{ inputs.base_ref }}` — both github.action_path and inputs.base_ref are interpolated directly into the shell command before the shell ever sees it, allowing an attacker-controlled base_ref to inject arbitrary shell commands.

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:25`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in `run:` block. In run-ansible-lint/action.yaml the 'Show inputs' step interpolates `${{ inputs.working_directory }}` and `${{ github.workspace }}` directly inside the shell script (lines 29-32), and the 'Run ansible-lint' step interpolates `${{ inputs.args }}` directly into the ansible-lint command (line 50). A caller-controlled `inputs.args` or `inputs.working_directory` can inject arbitrary shell commands.

Locations:

- `.github/actions/run-ansible-lint/action.yaml:29`
- `.github/actions/run-ansible-lint/action.yaml:30`
- `.github/actions/run-ansible-lint/action.yaml:32`
- `.github/actions/run-ansible-lint/action.yaml:50`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in `run:` block. In run-sanity/action.yaml the 'Get python-version from test-env' step runs `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` (line 27) and the 'Run tox sanity tests' step runs `python -m tox --ansible -e ${{ inputs.test-env }}` (line 40). The caller-supplied `inputs.test-env` is interpolated directly into the shell, enabling command injection.

Locations:

- `.github/actions/run-sanity/action.yaml:27`
- `.github/actions/run-sanity/action.yaml:40`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in `run:` block. In run-unit-galaxy/action.yaml the 'Get python-version from test-env' step runs `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` (line 27) and the 'Run tox unit tests' step runs `python -m tox --ansible -e ${{ inputs.test-env }}` (line 50). The caller-supplied `inputs.test-env` is interpolated directly into the shell, enabling command injection.

Locations:

- `.github/actions/run-unit-galaxy/action.yaml:27`
- `.github/actions/run-unit-galaxy/action.yaml:50`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in `run:` block. In generate-tox-ansible-matrix/action.yaml the 'Generate matrix' step runs `echo output=$(uv run tox --ansible --gh-matrix --matrix-scope ${{ inputs.scope }} --conf tox-ansible.ini)` — the caller-supplied `inputs.scope` is interpolated directly into the shell command, enabling command injection.

Locations:

- `.github/actions/generate-tox-ansible-matrix/action.yaml:43`

### github-env-injection (severity: high)

Unsanitized write of caller-controlled input to $GITHUB_OUTPUT. In run-sanity/action.yaml, `${{ inputs.test-env }}` is interpolated into a shell variable `output` (line 27) which is then written to $GITHUB_OUTPUT via `echo "py_version=$output" >> $GITHUB_OUTPUT` (line 28) without the required `printf '%s' ... | tr -d '\n\r'` sanitization step. A newline embedded in inputs.test-env can inject additional key=value pairs into the output file.

Locations:

- `.github/actions/run-sanity/action.yaml:27`
- `.github/actions/run-sanity/action.yaml:28`

### github-env-injection (severity: high)

Unsanitized write of caller-controlled input to $GITHUB_OUTPUT. In run-unit-galaxy/action.yaml, `${{ inputs.test-env }}` is interpolated into a shell variable `output` (line 27) which is then written to $GITHUB_OUTPUT via `echo "py_version=$output" >> $GITHUB_OUTPUT` (line 28) without the required `printf '%s' ... | tr -d '\n\r'` sanitization step.

Locations:

- `.github/actions/run-unit-galaxy/action.yaml:27`
- `.github/actions/run-unit-galaxy/action.yaml:28`

### github-env-injection (severity: high)

Unsanitized write of caller-controlled input to $GITHUB_OUTPUT. In generate-tox-ansible-matrix/action.yaml, `${{ inputs.scope }}` is interpolated into the shell variable `output` (line 43) which is then written to $GITHUB_OUTPUT via `echo "envlist=$output" >> $GITHUB_OUTPUT` (line 45) without the required `printf '%s' ... | tr -d '\n\r'` sanitization step.

Locations:

- `.github/actions/generate-tox-ansible-matrix/action.yaml:43`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:45`

### unpinned-uses (severity: high)

Multiple `uses:` references in composite action steps use mutable tag/version refs instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the referenced tag is moved or the repository is compromised. Failing references: `actions/setup-python@v4` (line 18).

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:18`

### unpinned-uses (severity: high)

Multiple `uses:` references use mutable tag/version refs. Failing references: `actions/checkout@v4` (line 18), `actions/github-script@v3` (line 25), `astral-sh/setup-uv@v5` (line 31).

Locations:

- `.github/actions/generate-tox-ansible-matrix/action.yaml:18`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:25`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:31`

### unpinned-uses (severity: high)

Uses reference `actions/setup-python@v5` is a mutable tag ref, not a pinned SHA digest.

Locations:

- `.github/actions/run-ansible-lint/action.yaml:37`

### unpinned-uses (severity: high)

Multiple `uses:` references use mutable tag/version refs. Failing references: `actions/github-script@v3` (line 17), `actions/setup-python@v4` (line 30).

Locations:

- `.github/actions/run-sanity/action.yaml:17`
- `.github/actions/run-sanity/action.yaml:30`

### unpinned-uses (severity: high)

Multiple `uses:` references use mutable tag/version refs. Failing references: `actions/github-script@v3` (line 17), `actions/setup-python@v4` (line 30).

Locations:

- `.github/actions/run-unit-galaxy/action.yaml:17`
- `.github/actions/run-unit-galaxy/action.yaml:30`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all 13 findings across 5 action files:

1. ansible_validate_changelog/action.yaml: Moved github.action_path and inputs.base_ref into env block; pinned actions/setup-python@v4 to SHA.

2. generate-tox-ansible-matrix/action.yaml: Moved inputs.scope into env block; sanitized output before writing to GITHUB_OUTPUT with tr -d '\n\r'; pinned actions/checkout@v4, actions/github-script@v3, astral-sh/setup-uv@v5 to SHAs.

3. run-ansible-lint/action.yaml: Moved inputs.working_directory, github.workspace into env block for Show inputs step; moved inputs.args into env block and tokenized with xargs into bash array for Run ansible-lint step; pinned actions/setup-python@v5 to SHA.

4. run-sanity/action.yaml: Moved inputs.test-env into env block; used printf '%s' to avoid echo special char interpretation; sanitized output before writing to GITHUB_OUTPUT; pinned actions/github-script@v3 and actions/setup-python@v4 to SHAs.

5. run-unit-galaxy/action.yaml: Same fixes as run-sanity; pinned actions/github-script@v3 and actions/setup-python@v4 to SHAs.

