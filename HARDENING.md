<!-- markdownlint-disable -->

# Hardening Report: ansible--ansible-content-actions/v0.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ansible--ansible-content-actions/v0.1.1** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files and composite actions reference external actions at mutable tags or branch names instead of pinned 40-character SHA digests, making them vulnerable to supply-chain attacks.

Failing references include:
- .github/workflows/ack.yml: `ansible/team-devtools/.github/workflows/ack.yml@main`
- .github/workflows/ansible_lint.yaml: `actions/checkout@v4`, `actions/setup-python@v5`
- .github/workflows/build_import.yaml: `actions/checkout@v4`
- .github/workflows/changelog.yaml: `actions/checkout@v4`, `ansible/ansible-content-actions/.github/actions/ansible_validate_changelog@main`
- .github/workflows/check_label.yaml: `actions/checkout@v4`, `release-drafter/release-drafter@v6`, `jesusvasquez333/verify-pr-label-action@v1.4.0`, `actions/add-to-project@main`
- .github/workflows/ci.yaml: `actions/checkout@v4`, `actions/setup-python@v5`, `pre-commit/action@v3.0.1`, `re-actors/alls-green@release/v1`
- .github/workflows/draft_release.yaml: `actions/checkout@v4`, `actions/setup-python@v5`, `release-drafter/release-drafter@v6`, `WyriHaximus/github-action-get-previous-tag@master`
- .github/workflows/integration.yaml: `actions/checkout@v4`, `actions/setup-python@v5`, `ansible/ansible-content-actions/.github/actions/add_tox_ansible@main`
- .github/workflows/push.yml: `ansible/team-devtools/.github/workflows/push.yml@main`
- .github/workflows/release.yaml: `ansible/ansible-content-actions/.github/workflows/release_ah.yaml@main`, `ansible/ansible-content-actions/.github/workflows/release_galaxy.yaml@main`
- .github/workflows/release.yml: `actions/checkout@v4`
- .github/workflows/release_ah.yaml: `actions/checkout@v4`
- .github/workflows/release_galaxy.yaml: `actions/checkout@v4`
- .github/workflows/sanity.yaml: `actions/checkout@v4`, `actions/setup-python@v5`, `ansible/ansible-content-actions/.github/actions/add_tox_ansible@main`
- .github/workflows/unit.yaml: `actions/checkout@v4`, `actions/setup-python@v5`, `ansible/ansible-content-actions/.github/actions/add_tox_ansible@main`
- .github/actions/ansible_validate_changelog/action.yaml: `actions/setup-python@v4`
- .github/actions/generate-tox-ansible-matrix/action.yaml: `actions/github-script@v3`, `actions/setup-python@v4`
- .github/actions/run-ansible-lint/action.yaml: `actions/setup-python@v5`
- .github/actions/run-sanity/action.yaml: `actions/github-script@v3`, `actions/setup-python@v4`
- .github/actions/run-unit-galaxy/action.yaml: `actions/github-script@v3`, `actions/setup-python@v4`

Locations:

- `.github/workflows/ack.yml:8`
- `.github/workflows/ansible_lint.yaml:22`
- `.github/workflows/build_import.yaml:12`
- `.github/workflows/changelog.yaml:12`
- `.github/workflows/check_label.yaml:13`
- `.github/workflows/ci.yaml:16`
- `.github/workflows/draft_release.yaml:18`
- `.github/workflows/integration.yaml:10`
- `.github/workflows/push.yml:11`
- `.github/workflows/release.yaml:22`
- `.github/workflows/release.yml:22`
- `.github/workflows/release_ah.yaml:18`
- `.github/workflows/release_galaxy.yaml:18`
- `.github/workflows/sanity.yaml:10`
- `.github/workflows/unit.yaml:10`
- `.github/actions/ansible_validate_changelog/action.yaml:16`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:20`
- `.github/actions/run-ansible-lint/action.yaml:28`
- `.github/actions/run-sanity/action.yaml:18`
- `.github/actions/run-unit-galaxy/action.yaml:18`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate GitHub Actions expressions (`${{ ... }}`) inside shell command strings, enabling script injection. An attacker-controlled value (e.g. a branch name, input, or matrix entry) can inject arbitrary shell commands.

Violations (sub-rule a — direct expression interpolation in run:):

1. ansible_lint.yaml — `run:` block interpolates `${{ inputs.working_directory }}` and `${{ github.workspace }}` directly into shell, and `ansible-lint ${{ inputs.args }}` passes unsanitised input directly to the shell.
2. ansible_lint.yaml — `working-directory: ${{ steps.inputs.outputs.working_directory }}` and `run: ansible-lint ${{ inputs.args }}`.
3. draft_release.yaml — `VERSION=${{ steps.release_drafter.outputs.tag_name }}` interpolates step output directly into shell; `sed -i -e 's/version:.*/version: ${{ env.VERSION }}/' galaxy.yml`; `git checkout -t -b ${{ env.BRANCH_NAME }}`; `git push origin ${{ env.BRANCH_NAME }}`.
4. integration.yaml — `python -m tox --ansible -e ${{ matrix.entry.name }}` interpolates matrix value directly.
5. sanity.yaml — `python -m tox --ansible -e ${{ matrix.entry.name }}` interpolates matrix value directly.
6. unit.yaml — `python -m tox --ansible -e ${{ matrix.entry.name }}` interpolates matrix value directly.
7. .github/actions/ansible_validate_changelog/action.yaml — `python3 ${{ github.action_path }}/validate_changelog.py --ref ${{ inputs.base_ref }}` interpolates inputs directly.
8. .github/actions/generate-tox-ansible-matrix/action.yaml — `echo output=$(python -m tox ... --matrix-scope ${{ inputs.scope }} ...)` interpolates inputs directly.
9. .github/actions/run-ansible-lint/action.yaml — `if [[ -n "${{ inputs.working_directory }}" ]]` and `ansible-lint ${{ inputs.args }}` interpolate inputs directly.
10. .github/actions/run-sanity/action.yaml — `echo ${{ inputs.test-env }} | cut ...` and `python -m tox --ansible -e ${{ inputs.test-env }}` interpolate inputs directly.
11. .github/actions/run-unit-galaxy/action.yaml — `echo ${{ inputs.test-env }} | cut ...` and `python -m tox --ansible -e ${{ inputs.test-env }}` interpolate inputs directly.

Locations:

- `.github/workflows/ansible_lint.yaml:28`
- `.github/workflows/ansible_lint.yaml:47`
- `.github/workflows/draft_release.yaml:36`
- `.github/workflows/draft_release.yaml:44`
- `.github/workflows/draft_release.yaml:52`
- `.github/workflows/integration.yaml:50`
- `.github/workflows/sanity.yaml:47`
- `.github/workflows/unit.yaml:53`
- `.github/actions/ansible_validate_changelog/action.yaml:22`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:44`
- `.github/actions/run-ansible-lint/action.yaml:33`
- `.github/actions/run-ansible-lint/action.yaml:44`
- `.github/actions/run-sanity/action.yaml:26`
- `.github/actions/run-sanity/action.yaml:36`
- `.github/actions/run-unit-galaxy/action.yaml:26`
- `.github/actions/run-unit-galaxy/action.yaml:40`

### github-env-injection (severity: high)

Several `run:` blocks write values derived from untrusted inputs or step outputs to `$GITHUB_OUTPUT` or `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

1. .github/workflows/ansible_lint.yaml ("Process inputs" step): writes `${{ inputs.working_directory }}` and `${{ github.workspace }}` directly to `$GITHUB_OUTPUT` — no newline stripping.
   `echo "working_directory=${{ inputs.working_directory }}" >> $GITHUB_OUTPUT`
   `echo "working_directory=${{ github.workspace }}" >> $GITHUB_OUTPUT`

2. .github/workflows/draft_release.yaml ("Remove the v prefix" step): writes step output `${{ steps.release_drafter.outputs.tag_name }}` directly to `$GITHUB_ENV` — no sanitization.
   `VERSION=${{ steps.release_drafter.outputs.tag_name }}`
   `echo "VERSION=${VERSION#v}" >> $GITHUB_ENV`

3. .github/workflows/release.yml ("Retrieve version" step): writes values derived from `$GITHUB_REF` (an environment variable set by GitHub, but not sanitized) to `$GITHUB_OUTPUT` — no sanitization.
   `echo "tag=${tag}" >> $GITHUB_OUTPUT`
   `echo "version=${version}" >> $GITHUB_OUTPUT`
   `echo "major=${major}" >> $GITHUB_OUTPUT`

4. .github/actions/run-sanity/action.yaml ("Get python-version" step): writes step output derived from `${{ inputs.test-env }}` to `$GITHUB_OUTPUT` — no sanitization.
   `echo "py_version=$output" >> $GITHUB_OUTPUT`

5. .github/actions/run-unit-galaxy/action.yaml ("Get python-version" step): same pattern as run-sanity.
   `echo "py_version=$output" >> $GITHUB_OUTPUT`

6. .github/actions/generate-tox-ansible-matrix/action.yaml ("Generate matrix" step): writes output derived from `${{ inputs.scope }}` to `$GITHUB_OUTPUT` — no sanitization.
   `echo "envlist=$output" >> $GITHUB_OUTPUT`

Locations:

- `.github/workflows/ansible_lint.yaml:28`
- `.github/workflows/draft_release.yaml:36`
- `.github/workflows/release.yml:18`
- `.github/actions/run-sanity/action.yaml:26`
- `.github/actions/run-unit-galaxy/action.yaml:26`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:44`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and contain at least one job that also lacks a job-level `permissions:` key, meaning the default (broad) token permissions apply:

- .github/workflows/ack.yml — no top-level permissions; job `ack` has no permissions block.
- .github/workflows/ansible_lint.yaml — no top-level permissions; job `build` has no permissions block.
- .github/workflows/build_import.yaml — no top-level permissions; job `build-import` has no permissions block.
- .github/workflows/changelog.yaml — no top-level permissions; job `changelog` has no permissions block.
- .github/workflows/draft_release.yaml — no top-level permissions; job `update_release_draft` has no permissions block.
- .github/workflows/integration.yaml — no top-level permissions; jobs `tox-matrix` and `test` have no permissions block.
- .github/workflows/push.yml — no top-level permissions; job `ack` has no permissions block.
- .github/workflows/refresh_ah_token.yaml — no top-level permissions; job `refresh` has no permissions block.
- .github/workflows/release.yaml — no top-level permissions; jobs `release_automation_hub` and `release_galaxy` have no permissions block.
- .github/workflows/release_ah.yaml — no top-level permissions; job `release` has no permissions block.
- .github/workflows/release_galaxy.yaml — no top-level permissions; job `release` has no permissions block.
- .github/workflows/sanity.yaml — no top-level permissions; jobs `tox-matrix` and `test` have no permissions block.
- .github/workflows/unit.yaml — no top-level permissions; jobs `tox-matrix` and `test` have no permissions block.

Locations:

- `.github/workflows/ack.yml:1`
- `.github/workflows/ansible_lint.yaml:1`
- `.github/workflows/build_import.yaml:1`
- `.github/workflows/changelog.yaml:1`
- `.github/workflows/draft_release.yaml:1`
- `.github/workflows/integration.yaml:1`
- `.github/workflows/push.yml:1`
- `.github/workflows/refresh_ah_token.yaml:1`
- `.github/workflows/release.yaml:1`
- `.github/workflows/release_ah.yaml:1`
- `.github/workflows/release_galaxy.yaml:1`
- `.github/workflows/sanity.yaml:1`
- `.github/workflows/unit.yaml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all 4 finding types across 21 files:

**unpinned-uses**: Pinned all action references to full SHA digests:
- actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5
- actions/setup-python@v5 → @a26af69be951a213d495a4c3e4e4022e16d87065
- actions/setup-python@v4 → @7f4fc3e22c37d6ff65e88745f38bd3157c663f7c
- pre-commit/action@v3.0.1 → @2c7b3805fd2a0fd8c1884dcaebf91fc102a13ecd
- re-actors/alls-green@release/v1 → @05ac9388f0aebcb5727afa17fcccfecd6f8ec5fe
- release-drafter/release-drafter@v6 → @6a93d829887aa2e0748befe2e808c66c0ec6e4c7
- jesusvasquez333/verify-pr-label-action@v1.4.0 → @657d111bbbe13e22bbd55870f1813c699bde1401
- actions/add-to-project@main → @efd82a6ec20cabb6c67bbcdd709eb6fe878becda
- WyriHaximus/github-action-get-previous-tag@master → @2df46c7f3054f28f5cbc9b415c216de80ac1b756
- actions/github-script@v3 → @ffc2c79a5b2490bd33e0a41c1de74b877714d736
- ansible/team-devtools@main → @b7222d7e2bd43e21b247fcdcb1a4014534637da0
- ansible/ansible-content-actions@main → @41201c8e472d212b283090605b5120d2374261b4

**script-injection**: Moved all ${{ }} expressions from run: blocks into env: blocks and referenced as plain shell variables. Used ${VAR:+"$VAR"} for optional args like ansible-lint args.

**github-env-injection**: Added printf '%s' ... | tr -d '\n\r' sanitization before all GITHUB_OUTPUT/GITHUB_ENV writes that derive from user-controlled values.

**missing-permissions**: Added `permissions: {}` at top level of all 13 workflow files that were missing it.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all four script-injection findings across four workflow files:

1. `.github/workflows/release.yml` (lines 40-41): Added `env:` block with `MAJOR` and `TAG` variables for `steps.version.outputs.major` and `steps.version.outputs.tag`. Updated `git tag` and `git push` commands to use `"${MAJOR}"` and `"${TAG}"` shell variables.

2. `.github/workflows/release_ah.yaml` (lines 33, 47): Added `env:` block with `AH_TOKEN` for `secrets.ah_token`. Replaced `${{ secrets.ah_token != '' }}` conditional with `[[ -n "$AH_TOKEN" ]]` and replaced `token=${{ secrets.ah_token }}` in the heredoc with `token=$AH_TOKEN`. Changed `run: >` to `run: |` for proper multi-line shell handling.

3. `.github/workflows/release_galaxy.yaml` (lines 33, 37): Added `env:` block with `ANSIBLE_GALAXY_API_KEY` for `secrets.ansible_galaxy_api_key`. Replaced `${{ secrets.ansible_galaxy_api_key != '' }}` conditional with `[[ -n "$ANSIBLE_GALAXY_API_KEY" ]]` and replaced `--api-key "${{ secrets.ansible_galaxy_api_key }}"` with `--api-key "$ANSIBLE_GALAXY_API_KEY"`. Changed `run: >` to `run: |`.

4. `.github/workflows/refresh_ah_token.yaml` (line 27): Added `env:` block with `AH_TOKEN` for `secrets.ah_token`. Replaced `refresh_token="${{ secrets.ah_token }}"` with `-d "refresh_token=$AH_TOKEN"`. Changed `run: >-` to `run: |` with explicit line continuations.

