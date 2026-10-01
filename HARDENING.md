<!-- markdownlint-disable -->

# Hardening Report: ansible--ansible-content-actions/v0.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ansible--ansible-content-actions/v0.1.1** was hardened automatically. 13 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): ${{ github.action_path }} and ${{ inputs.base_ref }} are directly interpolated inside a run: shell command. An attacker controlling the calling workflow's inputs can inject arbitrary shell commands via inputs.base_ref.

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:25`

### script-injection (severity: high)

Rule (a): ${{ inputs.scope }} is directly interpolated inside a run: shell command (`echo output=$(python -m tox --ansible --gh-matrix --matrix-scope ${{ inputs.scope }} ...)`). An attacker-controlled input value can inject arbitrary shell commands.

Locations:

- `.github/actions/generate-tox-ansible-matrix/action.yaml:42`

### script-injection (severity: high)

Rule (a): ${{ inputs.working_directory }} and ${{ github.workspace }} are directly interpolated inside a run: shell block (lines 27-30). Additionally, ${{ inputs.args }} is directly interpolated in the ansible-lint run: command (line 48), allowing shell command injection via attacker-controlled inputs.

Locations:

- `.github/actions/run-ansible-lint/action.yaml:27`
- `.github/actions/run-ansible-lint/action.yaml:48`

### script-injection (severity: high)

Rule (a): ${{ inputs.test-env }} is directly interpolated inside two run: shell commands — once in a command substitution (`output=$(echo ${{ inputs.test-env }} | cut ...)`, line 26) and once as a tox -e argument (line 38). An attacker-controlled input can inject arbitrary shell commands.

Locations:

- `.github/actions/run-sanity/action.yaml:26`
- `.github/actions/run-sanity/action.yaml:38`

### script-injection (severity: high)

Rule (a): ${{ inputs.test-env }} is directly interpolated inside two run: shell commands — once in a command substitution (`output=$(echo ${{ inputs.test-env }} | cut ...)`, line 26) and once as a tox -e argument (line 44). An attacker-controlled input can inject arbitrary shell commands.

Locations:

- `.github/actions/run-unit-galaxy/action.yaml:26`
- `.github/actions/run-unit-galaxy/action.yaml:44`

### github-env-injection (severity: high)

The 'Generate matrix' step writes $output (derived directly from ${{ inputs.scope }} without sanitization) to $GITHUB_OUTPUT via `echo "envlist=$output" >> $GITHUB_OUTPUT`. A newline in inputs.scope could inject additional key=value pairs into the output file.

Locations:

- `.github/actions/generate-tox-ansible-matrix/action.yaml:43`

### github-env-injection (severity: high)

The 'Get python-version from test-env' step writes $output (derived from ${{ inputs.test-env }} via command substitution, without sanitization) to $GITHUB_OUTPUT via `echo "py_version=$output" >> $GITHUB_OUTPUT`. A newline embedded in inputs.test-env could inject additional key=value pairs.

Locations:

- `.github/actions/run-sanity/action.yaml:27`

### github-env-injection (severity: high)

The 'Get python-version from test-env' step writes $output (derived from ${{ inputs.test-env }} via command substitution, without sanitization) to $GITHUB_OUTPUT via `echo "py_version=$output" >> $GITHUB_OUTPUT`. A newline embedded in inputs.test-env could inject additional key=value pairs.

Locations:

- `.github/actions/run-unit-galaxy/action.yaml:27`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved: `actions/setup-python@v4` (line 17), which could be silently updated to a malicious version.

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:17`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags instead of full 40-character commit SHAs: `actions/github-script@v3` (line 21) and `actions/setup-python@v4` (line 27). These are vulnerable to supply-chain attacks if the tags are moved.

Locations:

- `.github/actions/generate-tox-ansible-matrix/action.yaml:21`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:27`

### unpinned-uses (severity: high)

The `uses:` reference `actions/setup-python@v5` (line 35) is pinned to a mutable tag instead of a full 40-character commit SHA, making it vulnerable to supply-chain attacks.

Locations:

- `.github/actions/run-ansible-lint/action.yaml:35`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags instead of full 40-character commit SHAs: `actions/github-script@v3` (line 18) and `actions/setup-python@v4` (line 30). These are vulnerable to supply-chain attacks if the tags are moved.

Locations:

- `.github/actions/run-sanity/action.yaml:18`
- `.github/actions/run-sanity/action.yaml:30`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags instead of full 40-character commit SHAs: `actions/github-script@v3` (line 18) and `actions/setup-python@v4` (line 30). These are vulnerable to supply-chain attacks if the tags are moved.

Locations:

- `.github/actions/run-unit-galaxy/action.yaml:18`
- `.github/actions/run-unit-galaxy/action.yaml:30`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all 5 action files:

1. ansible_validate_changelog/action.yaml: Moved github.action_path and inputs.base_ref to env block (ACTION_PATH, BASE_REF); pinned setup-python@v4 to SHA 7f4fc3e.

2. generate-tox-ansible-matrix/action.yaml: Moved inputs.scope to env block (INPUT_SCOPE); sanitized output with tr -d '\n\r' before writing to GITHUB_OUTPUT; pinned github-script@v3 to SHA ffc2c79 and setup-python@v4 to SHA 7f4fc3e.

3. run-ansible-lint/action.yaml: Moved inputs.working_directory and github.workspace to env block; used xargs tokenization with bash array for inputs.args (argument list); pinned setup-python@v5 to SHA a26af69.

4. run-sanity/action.yaml: Moved inputs.test-env to env block (INPUT_TEST_ENV); used printf instead of echo to avoid injection; sanitized py_version output with tr -d '\n\r' before writing to GITHUB_OUTPUT; pinned github-script@v3 to SHA ffc2c79 and setup-python@v4 to SHA 7f4fc3e.

5. run-unit-galaxy/action.yaml: Same fixes as run-sanity; pinned github-script@v3 to SHA ffc2c79 and setup-python@v4 to SHA 7f4fc3e.

