<!-- markdownlint-disable -->

# Hardening Report: ansible--ansible-content-actions/v1.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ansible--ansible-content-actions/v1.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate GitHub Actions expressions (`${{ inputs.* }}`, `${{ github.* }}`) inside shell command strings, enabling script injection. Sub-rule (a): the expression is substituted into the shell command before the shell parses it, allowing an attacker-controlled value to inject arbitrary shell commands.

Offending lines:
- ansible_validate_changelog/action.yaml line 24: `python3 ${{ github.action_path }}/validate_changelog.py --ref ${{ inputs.base_ref }}`
- generate-tox-ansible-matrix/action.yaml line 43: `echo output=$(uv run tox --ansible --gh-matrix --matrix-scope ${{ inputs.scope }} --conf tox-ansible.ini)`
- run-sanity/action.yaml line 28: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')`
- run-sanity/action.yaml line 41: `python -m tox --ansible -e ${{ inputs.test-env }}`
- run-unit-galaxy/action.yaml line 28: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')`
- run-unit-galaxy/action.yaml line 47: `python -m tox --ansible -e ${{ inputs.test-env }}`
- run-ansible-lint/action.yaml line 26: `if [[ -n "${{ inputs.working_directory }}" ]]; then`
- run-ansible-lint/action.yaml line 27: `echo "working_directory=${{ inputs.working_directory }}"`
- run-ansible-lint/action.yaml line 29: `echo "working_directory=${{ github.workspace }}"`
- run-ansible-lint/action.yaml line 47: `ansible-lint ${{ inputs.args }}`

Fix: move all expression values into `env:` variables and reference them as quoted shell variables (e.g., `"$INPUT_ARGS"`).

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:24`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:43`
- `.github/actions/run-sanity/action.yaml:28`
- `.github/actions/run-sanity/action.yaml:41`
- `.github/actions/run-unit-galaxy/action.yaml:28`
- `.github/actions/run-unit-galaxy/action.yaml:47`
- `.github/actions/run-ansible-lint/action.yaml:26`
- `.github/actions/run-ansible-lint/action.yaml:47`

### github-env-injection (severity: high)

Untrusted `inputs.*` values are interpolated directly into shell variables and then written to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker can inject newlines into the input to poison `$GITHUB_OUTPUT` with arbitrary key=value pairs, potentially overwriting outputs consumed by downstream steps.

Offending patterns:
- generate-tox-ansible-matrix/action.yaml line 43-45: `${{ inputs.scope }}` is embedded in a command whose stdout is captured into `$output`, then `echo "envlist=$output" >> $GITHUB_OUTPUT` — no newline sanitization.
- run-sanity/action.yaml line 28-29: `${{ inputs.test-env }}` is piped through `cut`/`sed` into `$output`, then `echo "py_version=$output" >> $GITHUB_OUTPUT` — no newline sanitization.
- run-unit-galaxy/action.yaml line 28-29: same pattern as run-sanity — `${{ inputs.test-env }}` flows unsanitized into `$GITHUB_OUTPUT`.

Locations:

- `.github/actions/generate-tox-ansible-matrix/action.yaml:43`
- `.github/actions/run-sanity/action.yaml:28`
- `.github/actions/run-unit-galaxy/action.yaml:28`

### unpinned-uses (severity: high)

All `uses:` references in the composite action steps use mutable version tags instead of pinned 40-character SHA digests. This exposes the action to supply-chain attacks: a compromised or force-pushed tag could silently substitute malicious code.

Unpinned references found:
- ansible_validate_changelog/action.yaml: `actions/setup-python@v4`
- generate-tox-ansible-matrix/action.yaml: `actions/checkout@v4`, `actions/github-script@v3`, `astral-sh/setup-uv@v5`
- run-ansible-lint/action.yaml: `actions/setup-python@v5`
- run-sanity/action.yaml: `actions/github-script@v3`, `actions/setup-python@v4`
- run-unit-galaxy/action.yaml: `actions/github-script@v3`, `actions/setup-python@v4`

Fix: pin every `uses:` to a full 40-character commit SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/actions/ansible_validate_changelog/action.yaml:16`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:18`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:25`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:31`
- `.github/actions/run-ansible-lint/action.yaml:34`
- `.github/actions/run-sanity/action.yaml:19`
- `.github/actions/run-sanity/action.yaml:32`
- `.github/actions/run-unit-galaxy/action.yaml:19`
- `.github/actions/run-unit-galaxy/action.yaml:32`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three finding types across five action files:

1. unpinned-uses: Pinned all uses: references to full 40-char SHAs: actions/setup-python@v4→7f4fc3e, actions/setup-python@v5→a26af69, actions/checkout@v4→11d5960, actions/github-script@v3→ffc2c79, astral-sh/setup-uv@v5→d4b2f3b.

2. script-injection: Moved all ${{ }} expressions from run: blocks into env: blocks and referenced them as shell variables. For inputs.args in run-ansible-lint (an argument list), used the xargs tokenization pattern (while IFS= read -r -d '' t; do args+=("$t"); done < <(printf '%s' "$INPUT_ARGS" | xargs printf '%s\0')) to safely split the list.

3. github-env-injection: Added newline sanitization (printf '%s' "$raw" | tr -d '\n\r') before writing values to $GITHUB_OUTPUT in generate-tox-ansible-matrix, run-sanity, and run-unit-galaxy. Used two-step capture (raw then output) to preserve exit status under errexit.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Added '|| true' to the bare assignment 'raw=$(uv run tox --ansible --gh-matrix --matrix-scope "$SCOPE" --conf tox-ansible.ini)' in .github/actions/generate-tox-ansible-matrix/action.yaml line 50. This preserves upstream's behavior where the command was run inside printf/echo (hiding its exit status), so a non-zero exit from 'uv run tox' no longer aborts the step under bash's errexit mode.

