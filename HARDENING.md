<!-- markdownlint-disable -->

# Hardening Report: ansible--ansible-content-actions/v1.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ansible--ansible-content-actions/v1.0.1** was hardened automatically. 13 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): ${{ inputs.scope }} is interpolated directly inside a run: shell command in the 'Generate matrix' step. An attacker-controlled value is passed directly to the shell before quoting, enabling command injection. Offending line: `echo output=$(uv run tox --ansible --gh-matrix --matrix-scope ${{ inputs.scope }} --conf tox-ansible.ini)`

Locations:

- `.github/actions/generate-tox-ansible-matrix/action.yaml:40`

### script-injection (severity: high)

Sub-rule (a): ${{ inputs.test-env }} is interpolated directly inside two run: shell commands in run-sanity. (1) 'Get python-version from test-env' step: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')`. (2) 'Run tox sanity tests' step: `python -m tox --ansible -e ${{ inputs.test-env }}`. Both allow command injection via the inputs.test-env value.

Locations:

- `.github/actions/run-sanity/action.yaml:27`
- `.github/actions/run-sanity/action.yaml:40`

### script-injection (severity: high)

Sub-rule (a): ${{ inputs.test-env }} is interpolated directly inside two run: shell commands in run-unit-galaxy. (1) 'Get python-version from test-env' step: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')`. (2) 'Run tox unit tests' step: `python -m tox --ansible -e ${{ inputs.test-env }}`. Both allow command injection via the inputs.test-env value.

Locations:

- `.github/actions/run-unit-galaxy/action.yaml:27`
- `.github/actions/run-unit-galaxy/action.yaml:46`

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ inputs.* }} and ${{ github.* }} expressions are interpolated directly inside run: shell commands in run-ansible-lint. (1) 'Show inputs' step uses `${{ inputs.working_directory }}` (lines 26-28) and `${{ github.workspace }}` (line 28). (2) 'Run ansible-lint' step uses `${{ inputs.args }}` directly: `ansible-lint ${{ inputs.args }}` (line 46). An attacker controlling inputs.args or inputs.working_directory can inject arbitrary shell commands.

Locations:

- `.github/actions/run-ansible-lint/action.yaml:26`
- `.github/actions/run-ansible-lint/action.yaml:27`
- `.github/actions/run-ansible-lint/action.yaml:28`
- `.github/actions/run-ansible-lint/action.yaml:46`

### script-injection (severity: high)

Sub-rule (a): ${{ github.action_path }} and ${{ inputs.base_ref }} are interpolated directly inside a run: shell command in the 'Validate changelog' step: `python3 ${{ github.action_path }}/validate_changelog.py --ref ${{ inputs.base_ref }}`. The inputs.base_ref value (defaulting to github.event.pull_request.base.ref) is attacker-controlled and injected directly into the shell command.

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:25`

### github-env-injection (severity: high)

The 'Generate matrix' step writes `$output` to $GITHUB_OUTPUT without sanitization. The value of `$output` is derived directly from `${{ inputs.scope }}` interpolated into the shell command, making it attacker-controlled. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write: `echo "envlist=$output" >> $GITHUB_OUTPUT`.

Locations:

- `.github/actions/generate-tox-ansible-matrix/action.yaml:41`

### github-env-injection (severity: high)

The 'Get python-version from test-env' step writes `$output` to $GITHUB_OUTPUT without sanitization. The value of `$output` is derived from `${{ inputs.test-env }}` interpolated into the shell command via `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write: `echo "py_version=$output" >> $GITHUB_OUTPUT`.

Locations:

- `.github/actions/run-sanity/action.yaml:28`

### github-env-injection (severity: high)

The 'Get python-version from test-env' step writes `$output` to $GITHUB_OUTPUT without sanitization. The value of `$output` is derived from `${{ inputs.test-env }}` interpolated into the shell command via `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write: `echo "py_version=$output" >> $GITHUB_OUTPUT`.

Locations:

- `.github/actions/run-unit-galaxy/action.yaml:28`

### unpinned-uses (severity: high)

Multiple uses: references in this composite action use mutable version tags instead of pinned 40-character SHA digests: `actions/checkout@v4` (line 18), `actions/github-script@v3` (line 25), `astral-sh/setup-uv@v5` (line 30). These are vulnerable to supply-chain attacks if the tag is moved.

Locations:

- `.github/actions/generate-tox-ansible-matrix/action.yaml:18`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:25`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:30`

### unpinned-uses (severity: high)

Multiple uses: references in this composite action use mutable version tags instead of pinned 40-character SHA digests: `actions/github-script@v3` (line 20), `actions/setup-python@v4` (line 31). These are vulnerable to supply-chain attacks if the tag is moved.

Locations:

- `.github/actions/run-sanity/action.yaml:20`
- `.github/actions/run-sanity/action.yaml:31`

### unpinned-uses (severity: high)

Multiple uses: references in this composite action use mutable version tags instead of pinned 40-character SHA digests: `actions/github-script@v3` (line 20), `actions/setup-python@v4` (line 31). These are vulnerable to supply-chain attacks if the tag is moved.

Locations:

- `.github/actions/run-unit-galaxy/action.yaml:20`
- `.github/actions/run-unit-galaxy/action.yaml:31`

### unpinned-uses (severity: high)

The uses: reference `actions/setup-python@v5` (line 34) uses a mutable version tag instead of a pinned 40-character SHA digest, making it vulnerable to supply-chain attacks if the tag is moved.

Locations:

- `.github/actions/run-ansible-lint/action.yaml:34`

### unpinned-uses (severity: high)

The uses: reference `actions/setup-python@v4` (line 17) uses a mutable version tag instead of a pinned 40-character SHA digest, making it vulnerable to supply-chain attacks if the tag is moved.

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all 13 findings across 5 files:

1. generate-tox-ansible-matrix/action.yaml: Pinned actions/checkout@v4, actions/github-script@v3, astral-sh/setup-uv@v5 to full SHAs. Moved inputs.scope to SCOPE env var. Added tr -d '\n\r' sanitization before writing to GITHUB_OUTPUT.

2. run-sanity/action.yaml: Pinned actions/github-script@v3 and actions/setup-python@v4 to full SHAs. Moved inputs.test-env to TEST_ENV env var in both the 'Get python-version' step (using printf instead of echo) and 'Run tox sanity tests' step. Added sanitization before GITHUB_OUTPUT write.

3. run-unit-galaxy/action.yaml: Same fixes as run-sanity — pinned SHAs, moved inputs.test-env to TEST_ENV env var in both steps, added GITHUB_OUTPUT sanitization.

4. run-ansible-lint/action.yaml: Pinned actions/setup-python@v5 to full SHA. Moved inputs.working_directory and github.workspace to env vars in 'Show inputs' step. For inputs.args (a list-style input), used xargs-based tokenization into a bash array to safely pass arguments to ansible-lint.

5. ansible_validate_changelog/action.yaml: Pinned actions/setup-python@v4 to full SHA. Moved github.action_path to ACTION_PATH env var and inputs.base_ref to BASE_REF env var, then used them double-quoted in the shell command.

