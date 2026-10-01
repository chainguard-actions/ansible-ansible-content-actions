<!-- markdownlint-disable -->

# Hardening Report: ansible--ansible-content-actions/v1.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ansible--ansible-content-actions/v1.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple composite action run: blocks directly interpolate ${{ }} expressions into shell commands (rule a), allowing an attacker who controls the input to inject arbitrary shell commands.

- .github/actions/ansible_validate_changelog/action.yaml line 26: `python3 ${{ github.action_path }}/validate_changelog.py --ref ${{ inputs.base_ref }}` — both github.action_path and inputs.base_ref are interpolated directly.
- .github/actions/run-ansible-lint/action.yaml line 29: `if [[ -n "${{ inputs.working_directory }}" ]]` — inputs.working_directory interpolated in shell test.
- .github/actions/run-ansible-lint/action.yaml line 30: `echo "working_directory=${{ inputs.working_directory }}"` — direct interpolation.
- .github/actions/run-ansible-lint/action.yaml line 32: `echo "working_directory=${{ github.workspace }}"` — github context interpolated.
- .github/actions/run-ansible-lint/action.yaml line 50: `ansible-lint ${{ inputs.args }}` — unquoted, unsanitized inputs.args passed directly to shell.
- .github/actions/generate-tox-ansible-matrix/action.yaml line 44: `echo output=$(uv run tox --ansible --gh-matrix --matrix-scope ${{ inputs.scope }} --conf tox-ansible.ini)` — inputs.scope interpolated unquoted.
- .github/actions/run-sanity/action.yaml line 27: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` — inputs.test-env interpolated unquoted inside command substitution.
- .github/actions/run-sanity/action.yaml line 41: `python -m tox --ansible -e ${{ inputs.test-env }}` — inputs.test-env interpolated unquoted.
- .github/actions/run-unit-galaxy/action.yaml line 27: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` — inputs.test-env interpolated unquoted.
- .github/actions/run-unit-galaxy/action.yaml line 50: `python -m tox --ansible -e ${{ inputs.test-env }}` — inputs.test-env interpolated unquoted.

Fix: move all ${{ }} values into env: variables and double-quote every shell expansion.

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:26`
- `.github/actions/run-ansible-lint/action.yaml:29`
- `.github/actions/run-ansible-lint/action.yaml:30`
- `.github/actions/run-ansible-lint/action.yaml:32`
- `.github/actions/run-ansible-lint/action.yaml:50`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:44`
- `.github/actions/run-sanity/action.yaml:27`
- `.github/actions/run-sanity/action.yaml:41`
- `.github/actions/run-unit-galaxy/action.yaml:27`
- `.github/actions/run-unit-galaxy/action.yaml:50`

### github-env-injection (severity: high)

Three run: steps write values derived from untrusted inputs to $GITHUB_OUTPUT without applying the required sanitization step (printf '%s' ... | tr -d '\n\r'). An attacker-controlled newline in the input can inject arbitrary key=value pairs into the output file, poisoning subsequent steps.

- .github/actions/generate-tox-ansible-matrix/action.yaml line 46: `echo "envlist=$output" >> $GITHUB_OUTPUT` — $output is derived from ${{ inputs.scope }} (line 44) without sanitization.
- .github/actions/run-sanity/action.yaml line 28: `echo "py_version=$output" >> $GITHUB_OUTPUT` — $output is derived from ${{ inputs.test-env }} (line 27) without sanitization.
- .github/actions/run-unit-galaxy/action.yaml line 28: `echo "py_version=$output" >> $GITHUB_OUTPUT` — $output is derived from ${{ inputs.test-env }} (line 27) without sanitization.

Fix: sanitize before writing, e.g.: `safe=$(printf '%s' "$output" | tr -d '\n\r'); echo "py_version=$safe" >> "$GITHUB_OUTPUT"`

Locations:

- `.github/actions/generate-tox-ansible-matrix/action.yaml:46`
- `.github/actions/run-sanity/action.yaml:28`
- `.github/actions/run-unit-galaxy/action.yaml:28`

### unpinned-uses (severity: high)

All uses: references in the composite actions use mutable version tags instead of immutable 40-character commit SHA digests. A compromised or malicious tag update could silently alter the code executed by these actions (supply-chain attack).

Failing references:
- .github/actions/ansible_validate_changelog/action.yaml: `actions/setup-python@v6`
- .github/actions/generate-tox-ansible-matrix/action.yaml: `actions/checkout@v6`, `actions/github-script@v8`, `astral-sh/setup-uv@v7`
- .github/actions/run-ansible-lint/action.yaml: `actions/setup-python@v6`
- .github/actions/run-sanity/action.yaml: `actions/github-script@v8`, `actions/setup-python@v6`
- .github/actions/run-unit-galaxy/action.yaml: `actions/github-script@v8`, `actions/setup-python@v6`

Fix: pin every uses: to a full 40-hex-character SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:18`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:18`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:25`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:31`
- `.github/actions/run-ansible-lint/action.yaml:37`
- `.github/actions/run-sanity/action.yaml:18`
- `.github/actions/run-sanity/action.yaml:31`
- `.github/actions/run-unit-galaxy/action.yaml:18`
- `.github/actions/run-unit-galaxy/action.yaml:33`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all three finding types across five action files:

1. unpinned-uses: Pinned all uses: references to full 40-char SHAs: actions/setup-python@v6→ece7cb06..., actions/checkout@v6→d23441a4..., actions/github-script@v8→ed597411..., astral-sh/setup-uv@v7→37802adc...

2. script-injection: Moved all ${{ }} expressions from run: blocks into env: blocks and referenced them as shell variables. For inputs.args (a list), used xargs-based tokenization into a bash array to preserve argument boundaries safely.

3. github-env-injection: Added sanitization (printf '%s' "$output" | tr -d '\n\r') before writing derived values to $GITHUB_OUTPUT in generate-tox-ansible-matrix, run-sanity, and run-unit-galaxy.

