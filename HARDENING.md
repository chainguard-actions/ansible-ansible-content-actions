<!-- markdownlint-disable -->

# Hardening Report: ansible--ansible-content-actions/v1.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ansible--ansible-content-actions/v1.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ inputs.* }} and ${{ github.* }} expressions inside shell command strings (sub-rule a). This allows an attacker who controls those inputs to inject arbitrary shell commands.

Offending lines:
- ansible_validate_changelog/action.yaml: `python3 ${{ github.action_path }}/validate_changelog.py --ref ${{ inputs.base_ref }}` — both github.action_path and inputs.base_ref are interpolated directly into the shell command.
- generate-tox-ansible-matrix/action.yaml: `echo output=$(uv run tox --ansible --gh-matrix --matrix-scope ${{ inputs.scope }} --conf tox-ansible.ini)` — inputs.scope is interpolated directly.
- run-ansible-lint/action.yaml: `if [[ -n "${{ inputs.working_directory }}" ]]` and `echo "working_directory=${{ inputs.working_directory }}"` and `echo "working_directory=${{ github.workspace }}"` and `ansible-lint ${{ inputs.args }}` — multiple direct interpolations.
- run-sanity/action.yaml: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` — inputs.test-env interpolated directly.
- run-unit-galaxy/action.yaml: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` and `python -m tox --ansible -e ${{ inputs.test-env }}` — inputs.test-env interpolated directly.

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:23`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:38`
- `.github/actions/run-ansible-lint/action.yaml:23`
- `.github/actions/run-ansible-lint/action.yaml:42`
- `.github/actions/run-sanity/action.yaml:27`
- `.github/actions/run-unit-galaxy/action.yaml:27`
- `.github/actions/run-unit-galaxy/action.yaml:47`

### github-env-injection (severity: high)

Several run: blocks write values derived from untrusted inputs to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r').

- run-sanity/action.yaml: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//'); echo "py_version=$output" >> $GITHUB_OUTPUT` — the value of inputs.test-env flows (via $output) into $GITHUB_OUTPUT with no newline sanitization.
- run-unit-galaxy/action.yaml: same pattern — `output=$(echo ${{ inputs.test-env }} | ...); echo "py_version=$output" >> $GITHUB_OUTPUT`.
- generate-tox-ansible-matrix/action.yaml: `echo output=$(uv run tox ... --matrix-scope ${{ inputs.scope }} ...); echo "envlist=$output" >> $GITHUB_OUTPUT` — inputs.scope influences the value written to $GITHUB_OUTPUT without sanitization.

Locations:

- `.github/actions/run-sanity/action.yaml:27`
- `.github/actions/run-unit-galaxy/action.yaml:27`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:38`

### unpinned-uses (severity: high)

All uses: references in the composite action steps reference mutable version tags instead of immutable 40-character SHA digests, making the action vulnerable to supply-chain attacks if those tags are moved or compromised.

Unpinned references found:
- actions/setup-python@v4 (ansible_validate_changelog/action.yaml)
- actions/checkout@v4 (generate-tox-ansible-matrix/action.yaml)
- actions/github-script@v3 (generate-tox-ansible-matrix/action.yaml)
- astral-sh/setup-uv@v5 (generate-tox-ansible-matrix/action.yaml)
- actions/setup-python@v5 (run-ansible-lint/action.yaml)
- actions/github-script@v3 (run-sanity/action.yaml)
- actions/setup-python@v4 (run-sanity/action.yaml)
- actions/github-script@v3 (run-unit-galaxy/action.yaml)
- actions/setup-python@v4 (run-unit-galaxy/action.yaml)

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:15`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:15`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:21`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:27`
- `.github/actions/run-ansible-lint/action.yaml:28`
- `.github/actions/run-sanity/action.yaml:18`
- `.github/actions/run-sanity/action.yaml:33`
- `.github/actions/run-unit-galaxy/action.yaml:18`
- `.github/actions/run-unit-galaxy/action.yaml:34`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all 3 finding types across 5 action files:

1. unpinned-uses: Pinned all 9 action references to full SHA digests (actions/setup-python@v4→7f4fc3e, actions/setup-python@v5→a26af69, actions/checkout@v4→11d5960, actions/github-script@v3→ffc2c79, astral-sh/setup-uv@v5→d4b2f3b).

2. script-injection: Moved all ${{ inputs.* }} and ${{ github.* }} expressions from run: shell strings into env: blocks. For inputs.args in run-ansible-lint (a list-style argument), used the xargs tokenization idiom to preserve argument boundaries.

3. github-env-injection: Added sanitization (printf '%s' | tr -d '\n\r') before writing derived values to $GITHUB_OUTPUT in run-sanity, run-unit-galaxy, and generate-tox-ansible-matrix. Used two-step capture (raw=...; output=$(printf '%s' "$raw" | tr -d '\n\r')) to preserve errexit behavior.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Fixed two issues in .github/actions/generate-tox-ansible-matrix/action.yaml:
1. Added `|| true` to the `output=$(uv run tox ...)` assignment (line 50) so a non-zero exit doesn't abort the step under errexit, matching upstream's behavior where the command ran inside echo/printf's argument substitution.
2. Removed the `safe=` sanitization line and the `echo "envlist=$safe" >> "$GITHUB_OUTPUT"` line (line 53) since upstream's step never writes `envlist` to $GITHUB_OUTPUT — tox-ansible writes it itself during execution, and the hardened version was incorrectly overwriting that value.

