<!-- markdownlint-disable -->

# Hardening Report: ansible--ansible-content-actions/v0.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ansible--ansible-content-actions/v0.1.0** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow and action files reference external actions using mutable tags or branch names instead of pinned full 40-character SHA commits, making them vulnerable to supply-chain attacks.

Failing references include:
- ack.yml: ansible/team-devtools/.github/workflows/ack.yml@main
- ansible_lint.yaml: actions/checkout@v4, actions/setup-python@v5
- build_import.yaml: actions/checkout@v4
- changelog.yaml: actions/checkout@v4, ansible/ansible-content-actions/.github/actions/ansible_validate_changelog@main
- check_label.yaml: actions/checkout@v4, release-drafter/release-drafter@v6, jesusvasquez333/verify-pr-label-action@v1.4.0, actions/add-to-project@main
- ci.yaml: actions/checkout@v4, actions/setup-python@v5, pre-commit/action@v3.0.1, re-actors/alls-green@release/v1
- draft_release.yaml: actions/checkout@v4, actions/setup-python@v5, release-drafter/release-drafter@v6, WyriHaximus/github-action-get-previous-tag@master
- integration.yaml: actions/checkout@v4, actions/setup-python@v5, ansible/ansible-content-actions/.github/actions/add_tox_ansible@main
- push.yml: ansible/team-devtools/.github/workflows/push.yml@main
- release.yaml: ansible/ansible-content-actions/.github/workflows/release_ah.yaml@main, ansible/ansible-content-actions/.github/workflows/release_galaxy.yaml@main
- release_ah.yaml: actions/checkout@v4
- release_galaxy.yaml: actions/checkout@v4
- sanity.yaml: actions/checkout@v4, actions/setup-python@v5, ansible/ansible-content-actions/.github/actions/add_tox_ansible@main
- unit.yaml: actions/checkout@v4, actions/setup-python@v5, ansible/ansible-content-actions/.github/actions/add_tox_ansible@main
- .github/actions/add_tox_ansible/action.yaml: (no uses:)
- .github/actions/ansible_validate_changelog/action.yaml: actions/setup-python@v4
- .github/actions/generate-tox-ansible-matrix/action.yaml: actions/github-script@v3, actions/setup-python@v4
- .github/actions/run-ansible-lint/action.yaml: actions/setup-python@v5
- .github/actions/run-sanity/action.yaml: actions/github-script@v3, actions/setup-python@v4
- .github/actions/run-unit-galaxy/action.yaml: actions/github-script@v3, actions/setup-python@v4

Locations:

- `.github/workflows/ack.yml:8`
- `.github/workflows/ansible_lint.yaml:24`
- `.github/workflows/build_import.yaml:12`
- `.github/workflows/changelog.yaml:13`
- `.github/workflows/check_label.yaml:10`
- `.github/workflows/ci.yaml:16`
- `.github/workflows/draft_release.yaml:16`
- `.github/workflows/integration.yaml:7`
- `.github/workflows/push.yml:11`
- `.github/workflows/release.yaml:23`
- `.github/workflows/release_ah.yaml:14`
- `.github/workflows/release_galaxy.yaml:13`
- `.github/workflows/sanity.yaml:7`
- `.github/workflows/unit.yaml:7`
- `.github/actions/ansible_validate_changelog/action.yaml:13`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:17`
- `.github/actions/run-ansible-lint/action.yaml:27`
- `.github/actions/run-sanity/action.yaml:17`
- `.github/actions/run-unit-galaxy/action.yaml:17`

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ }} expressions into shell commands, enabling script injection. An attacker-controlled value (e.g. via inputs, matrix, or github context) can inject arbitrary shell commands.

Violations (sub-rule a — direct expression interpolation in run:):

- ansible_lint.yaml 'Process inputs' step: `echo "working_directory=${{ inputs.working_directory }}" >> $GITHUB_OUTPUT` and `echo "working_directory=${{ github.workspace }}" >> $GITHUB_OUTPUT`; 'Run ansible-lint' step: `ansible-lint ${{ inputs.args }}`
- draft_release.yaml 'Remove the v prefix' step: `VERSION=${{ steps.release_drafter.outputs.tag_name }}`; 'Update the galaxy.yml version' step: `sed -i -e 's/version:.*/version: ${{ env.VERSION }}/' galaxy.yml`; 'Create PR for changelog' step: `git checkout -t -b ${{ env.BRANCH_NAME }}`
- integration.yaml 'Run tox integration tests' step: `python -m tox --ansible -e ${{ matrix.entry.name }} --conf tox-ansible.ini`
- sanity.yaml 'Run tox sanity tests' step: `python -m tox --ansible -e ${{ matrix.entry.name }} --conf tox-ansible.ini`
- unit.yaml 'Run tox unit tests' step: `python -m tox --ansible -e ${{ matrix.entry.name }} --conf tox-ansible.ini`
- release_ah.yaml 'Publish the collection on Automation Hub' step: `token=${{ secrets.ah_token }}`
- release_galaxy.yaml 'Publish the collection on Galaxy' step: `ansible-galaxy collection publish "${TARBALL}" --api-key "${{ secrets.ansible_galaxy_api_key }}"`
- .github/actions/generate-tox-ansible-matrix/action.yaml 'Generate matrix' step: `python -m tox --ansible --gh-matrix --matrix-scope ${{ inputs.scope }} --conf tox-ansible.ini`
- .github/actions/run-ansible-lint/action.yaml 'Run ansible-lint' step: `ansible-lint ${{ inputs.args }}`
- .github/actions/run-sanity/action.yaml 'Get python-version' step: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` and 'Run tox sanity tests' step: `python -m tox --ansible -e ${{ inputs.test-env }}`
- .github/actions/run-unit-galaxy/action.yaml 'Get python-version' step: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//')` and 'Run tox unit tests' step: `python -m tox --ansible -e ${{ inputs.test-env }}`
- .github/actions/ansible_validate_changelog/action.yaml 'Validate changelog' step: `python3 ${{ github.action_path }}/validate_changelog.py --ref ${{ inputs.base_ref }}`

Locations:

- `.github/workflows/ansible_lint.yaml:28`
- `.github/workflows/ansible_lint.yaml:47`
- `.github/workflows/draft_release.yaml:34`
- `.github/workflows/draft_release.yaml:44`
- `.github/workflows/draft_release.yaml:55`
- `.github/workflows/integration.yaml:47`
- `.github/workflows/sanity.yaml:45`
- `.github/workflows/unit.yaml:50`
- `.github/workflows/release_ah.yaml:27`
- `.github/workflows/release_galaxy.yaml:25`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:40`
- `.github/actions/run-ansible-lint/action.yaml:46`
- `.github/actions/run-sanity/action.yaml:24`
- `.github/actions/run-sanity/action.yaml:35`
- `.github/actions/run-unit-galaxy/action.yaml:24`
- `.github/actions/run-unit-galaxy/action.yaml:42`
- `.github/actions/ansible_validate_changelog/action.yaml:22`

### github-env-injection (severity: high)

Multiple run: blocks write values derived from untrusted inputs or workflow-controlled contexts to $GITHUB_OUTPUT or $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r').

Violations:

- ansible_lint.yaml 'Process inputs' step writes ${{ inputs.working_directory }} and ${{ github.workspace }} directly to $GITHUB_OUTPUT: `echo "working_directory=${{ inputs.working_directory }}" >> $GITHUB_OUTPUT`
- draft_release.yaml 'Remove the v prefix from the release drafter version' step writes step output to $GITHUB_ENV without sanitization: `echo "VERSION=${VERSION#v}" >> $GITHUB_ENV` where VERSION comes from `${{ steps.release_drafter.outputs.tag_name }}`
- .github/actions/run-sanity/action.yaml 'Get python-version from test-env' step writes a value derived from ${{ inputs.test-env }} to $GITHUB_OUTPUT: `echo "py_version=$output" >> $GITHUB_OUTPUT`
- .github/actions/run-unit-galaxy/action.yaml 'Get python-version from test-env' step writes a value derived from ${{ inputs.test-env }} to $GITHUB_OUTPUT: `echo "py_version=$output" >> $GITHUB_OUTPUT`
- .github/actions/generate-tox-ansible-matrix/action.yaml 'Generate matrix' step writes a value derived from ${{ inputs.scope }} to $GITHUB_OUTPUT: `echo "envlist=$output" >> $GITHUB_OUTPUT`

Locations:

- `.github/workflows/ansible_lint.yaml:28`
- `.github/workflows/draft_release.yaml:36`
- `.github/actions/run-sanity/action.yaml:26`
- `.github/actions/run-unit-galaxy/action.yaml:26`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:41`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and at least one job also lacks a `permissions:` key, meaning the workflow runs with the default (potentially write) token permissions:

- ack.yml: no top-level permissions, job 'ack' has no permissions
- ansible_lint.yaml: no top-level permissions, job 'build' has no permissions
- build_import.yaml: no top-level permissions, job 'build-import' has no permissions
- changelog.yaml: no top-level permissions, job 'changelog' has no permissions
- draft_release.yaml: no top-level permissions, job 'update_release_draft' has no permissions
- integration.yaml: no top-level permissions, jobs 'tox-matrix' and 'test' have no permissions
- push.yml: no top-level permissions, job 'ack' has no permissions
- release.yaml: no top-level permissions, jobs 'release_automation_hub' and 'release_galaxy' have no permissions
- release_ah.yaml: no top-level permissions, job 'release' has no permissions
- release_galaxy.yaml: no top-level permissions, job 'release' has no permissions
- sanity.yaml: no top-level permissions, jobs 'tox-matrix' and 'test' have no permissions
- unit.yaml: no top-level permissions, jobs 'tox-matrix' and 'test' have no permissions

Locations:

- `.github/workflows/ack.yml:1`
- `.github/workflows/ansible_lint.yaml:1`
- `.github/workflows/build_import.yaml:1`
- `.github/workflows/changelog.yaml:1`
- `.github/workflows/draft_release.yaml:1`
- `.github/workflows/integration.yaml:1`
- `.github/workflows/push.yml:1`
- `.github/workflows/release.yaml:1`
- `.github/workflows/release_ah.yaml:1`
- `.github/workflows/release_galaxy.yaml:1`
- `.github/workflows/sanity.yaml:1`
- `.github/workflows/unit.yaml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all findings across 14 workflow files and 5 action files:

**unpinned-uses**: Pinned all action references to full SHA commits:
- actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262
- actions/setup-python@v5 → @a26af69be951a213d495a4c3e4e4022e16d87065
- actions/setup-python@v4 → @7f4fc3e22c37d6ff65e88745f38bd3157c663f7c
- pre-commit/action@v3.0.1 → @2c7b3805fd2a0fd8c1884dcaebf91fc102a13ecd
- re-actors/alls-green@release/v1 → @05ac9388f0aebcb5727afa17fcccfecd6f8ec5fe
- release-drafter/release-drafter@v6 → @6a93d829887aa2e0748befe2e808c66c0ec6e4c7
- jesusvasquez333/verify-pr-label-action@v1.4.0 → @657d111bbbe13e22bbd55870f1813c699bde1401
- actions/add-to-project@main → @00a39cd6fa0633a59f7ae3bd4edada12a688183f
- WyriHaximus/github-action-get-previous-tag@master → @2df46c7f3054f28f5cbc9b415c216de80ac1b756
- actions/github-script@v3 → @ffc2c79a5b2490bd33e0a41c1de74b877714d736
- ansible/team-devtools@main → @52cd2a024bb3d3a22bba5d9169798bb9b13848a4
- ansible/ansible-content-actions@main → @11fc6c1c1a2421cac44cc5c40b6f72656745f1f9

**script-injection**: Moved all ${{ }} expressions from run: blocks to env: blocks. Used ${VAR:+"$VAR"} for optional args (ansible-lint args). Used "$VAR" for required args (tox -e, git checkout branch, sed version).

**github-env-injection**: Added printf '%s' ... | tr -d '\n\r' sanitization before writing to $GITHUB_OUTPUT/$GITHUB_ENV in: ansible_lint.yaml (working_directory), draft_release.yaml (VERSION), run-sanity/action.yaml (py_version), run-unit-galaxy/action.yaml (py_version), generate-tox-ansible-matrix/action.yaml (envlist).

**missing-permissions**: Added `permissions: {}` top-level block to all 12 workflow files that lacked it: ack.yml, ansible_lint.yaml, build_import.yaml, changelog.yaml, draft_release.yaml, integration.yaml, push.yml, release.yaml, release_ah.yaml, release_galaxy.yaml, sanity.yaml, unit.yaml. check_label.yaml already had job-level permissions and received a top-level `permissions: {}` as well.

### Iteration 2

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed three findings across two workflow files:
1. release.yml (script-injection): Moved `${{ steps.version.outputs.major }}` and `${{ steps.version.outputs.tag }}` into an `env:` block (MAJOR, TAG) in the 'Force update major tag' step. The run script now uses plain shell variables `${MAJOR}` and `${TAG}`, preventing template-engine substitution before the shell sees the values.
2. refresh_ah_token.yaml (script-injection): Moved `${{ secrets.ah_token }}` into an `env:` block (AH_TOKEN) in the 'Refresh the automation hub token' step. The curl command now references `${AH_TOKEN}` as a plain environment variable.
3. refresh_ah_token.yaml (missing-permissions): Added `permissions: {}` at the top level of the workflow, since this workflow only invokes curl and requires no GitHub token permissions.

