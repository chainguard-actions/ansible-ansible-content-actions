<!-- markdownlint-disable -->

# Hardening Report: ansible--ansible-content-actions/v1.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ansible--ansible-content-actions/v1.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple run: blocks directly interpolate ${{ }} expressions, allowing script injection. In ansible_validate_changelog/action.yaml line 25: `python3 ${{ github.action_path }}/validate_changelog.py --ref ${{ inputs.base_ref }}` — both github.action_path and inputs.base_ref are interpolated directly into the shell command. In run-ansible-lint/action.yaml lines 28-30: `if [[ -n "${{ inputs.working_directory }}" ]]` and `echo "working_directory=${{ inputs.working_directory }}"` and `echo "working_directory=${{ github.workspace }}"` — inputs and github context interpolated directly. In run-ansible-lint/action.yaml line 49: `ansible-lint ${{ inputs.args }}` — attacker-controlled input directly in shell. In generate-tox-ansible-matrix/action.yaml line 44: `echo output=$(uv run tox --ansible --gh-matrix --matrix-scope ${{ inputs.scope }} --conf tox-ansible.ini)` — inputs.scope interpolated directly. In run-sanity/action.yaml line 27: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` and line 39: `python -m tox --ansible -e ${{ inputs.test-env }}` — inputs.test-env interpolated directly. In run-unit-galaxy/action.yaml line 27: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` and line 43: `python -m tox --ansible -e ${{ inputs.test-env }}` — inputs.test-env interpolated directly. All of these allow an attacker to inject arbitrary shell commands via crafted input values.

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:25`
- `.github/actions/run-ansible-lint/action.yaml:28`
- `.github/actions/run-ansible-lint/action.yaml:49`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:44`
- `.github/actions/run-sanity/action.yaml:27`
- `.github/actions/run-sanity/action.yaml:39`
- `.github/actions/run-unit-galaxy/action.yaml:27`
- `.github/actions/run-unit-galaxy/action.yaml:43`

### github-env-injection (severity: high)

Multiple run: blocks write values derived from untrusted inputs to $GITHUB_OUTPUT without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). In generate-tox-ansible-matrix/action.yaml line 46: `echo "envlist=$output" >> $GITHUB_OUTPUT` where $output is derived from `${{ inputs.scope }}` interpolated on line 44. In run-sanity/action.yaml line 28: `echo "py_version=$output" >> $GITHUB_OUTPUT` where $output is derived from `${{ inputs.test-env }}` interpolated on line 27. In run-unit-galaxy/action.yaml line 28: `echo "py_version=$output" >> $GITHUB_OUTPUT` where $output is derived from `${{ inputs.test-env }}` interpolated on line 27. An attacker can inject newlines into these inputs to poison GITHUB_OUTPUT with arbitrary key-value pairs.

Locations:

- `.github/actions/generate-tox-ansible-matrix/action.yaml:46`
- `.github/actions/run-sanity/action.yaml:28`
- `.github/actions/run-unit-galaxy/action.yaml:28`

### unpinned-uses (severity: high)

Multiple uses: references across the composite action files use mutable tags or version strings instead of full 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the referenced tag is moved or the upstream repository is compromised. Failing references include: actions/setup-python@v4 (ansible_validate_changelog, run-sanity, run-unit-galaxy), actions/github-script@v3 (generate-tox-ansible-matrix, run-sanity, run-unit-galaxy), actions/checkout@v4 (generate-tox-ansible-matrix), astral-sh/setup-uv@v5 (generate-tox-ansible-matrix), actions/setup-python@v5 (run-ansible-lint).

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:18`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:17`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:25`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:31`
- `.github/actions/run-ansible-lint/action.yaml:36`
- `.github/actions/run-sanity/action.yaml:18`
- `.github/actions/run-sanity/action.yaml:31`
- `.github/actions/run-unit-galaxy/action.yaml:18`
- `.github/actions/run-unit-galaxy/action.yaml:31`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all findings across 5 action files:

**unpinned-uses**: Pinned all uses: references to full SHA hashes:
- actions/setup-python@v4 → @7f4fc3e22c37d6ff65e88745f38bd3157c663f7c
- actions/setup-python@v5 → @a26af69be951a213d495a4c3e4e4022e16d87065
- actions/github-script@v3 → @ffc2c79a5b2490bd33e0a41c1de74b877714d736
- actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262
- astral-sh/setup-uv@v5 → @d4b2f3b6ecc6e67c4457f6d3e41ec42d3d0fcb86

**script-injection**: Moved all ${{ }} expressions from run: blocks into env: blocks and referenced as plain shell variables. For inputs.args (a list-style input in run-ansible-lint), used the xargs tokenization pattern with a bash array to preserve argument boundaries.

**github-env-injection**: Added sanitization (tr -d '\n\r') before writing values derived from untrusted inputs to $GITHUB_OUTPUT in generate-tox-ansible-matrix, run-sanity, and run-unit-galaxy. Used the two-step capture pattern (raw=$(cmd); output=$(printf '%s' "$raw" | tr -d '\n\r')) to preserve exit codes under errexit.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Fixed .github/actions/generate-tox-ansible-matrix/action.yaml line 50: added '|| true' to the bare assignment 'output=$(uv run tox --ansible --gh-matrix --matrix-scope "$INPUT_SCOPE" --conf tox-ansible.ini)' so that a non-zero exit from the uv command does not abort the step under bash's errexit mode, matching upstream behavior where the command was run inside echo/printf which hid the command's exit status.

