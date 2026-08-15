<!-- markdownlint-disable -->

# Hardening Report: ansible--ansible-content-actions/v1.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ansible--ansible-content-actions/v1.1.0** was hardened automatically. 4 finding(s) were identified and resolved across 6 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ }} expressions into shell commands (sub-rule a), allowing script injection. Key instances:
- ansible_lint.yaml: `run: ansible-lint ${{ inputs.args }}` — attacker-controlled input injected directly into shell command.
- ansible_lint.yaml: run block interpolates `${{ inputs.working_directory }}` and `${{ github.workspace }}` directly in shell.
- sonarcloud.yaml: `run: ${{ inputs.test_command }}` — entire shell command is attacker-controlled input.
- sonarcloud.yaml: run block builds ARGS string with `${{ github.event.pull_request.head.ref }}`, `${{ github.event.pull_request.number }}`, `${{ github.event.pull_request.head.sha }}`, `${{ github.event.pull_request.base.ref }}` interpolated directly.
- draft_release.yaml: `VERSION=${{ steps.release_drafter.outputs.tag_name }}` interpolated directly in shell.
- cml_network_integration.yaml: `run: python3 -m pip install https://github.com/ansible/ansible/archive/${{ inputs.ansible_version }}.tar.gz` — input injected into URL in shell.
- cml_network_integration.yaml: `run: ansible-test network-integration --inventory "${{ steps.render_inventory.outputs.inventory_path }}"` — step output injected directly.
- network_integration.yaml: `run: python3 -m pip install https://github.com/ansible/ansible/archive/${{ matrix.ansible-version }}.tar.gz` — matrix value injected into URL.
- network_integration.yaml: `echo "CLABTITLE=${{ inputs.lab_title }}_PR${{ github.event.pull_request.number }}_..." >> $GITHUB_ENV` — inputs injected directly.
- network_integration.yaml: `run: echo ${{ env.CLABTITLE }}` — env context injected directly.
- run-ansible-lint/action.yaml: `run: ansible-lint ${{ inputs.args }}` — composite action input injected directly.
- run-sanity/action.yaml: `run: python -m tox --ansible -e ${{ inputs.test-env }}` — composite action input injected directly.
- run-unit-galaxy/action.yaml: `run: python -m tox --ansible -e ${{ inputs.test-env }}` — composite action input injected directly.
- generate-tox-ansible-matrix/action.yaml: `echo output=$(uv run tox ... --matrix-scope ${{ inputs.scope }} ...)` — composite action input injected directly.
- ansible_validate_changelog/action.yaml: `run: python3 ${{ github.action_path }}/validate_changelog.py --ref ${{ inputs.base_ref }}` — inputs injected directly.

Locations:

- `.github/workflows/ansible_lint.yaml:47`
- `.github/workflows/ansible_lint.yaml:34`
- `.github/workflows/sonarcloud.yaml:72`
- `.github/workflows/sonarcloud.yaml:60`
- `.github/workflows/draft_release.yaml:30`
- `.github/workflows/cml_network_integration.yaml:57`
- `.github/workflows/cml_network_integration.yaml:107`
- `.github/workflows/network_integration.yaml:72`
- `.github/workflows/network_integration.yaml:196`
- `.github/workflows/network_integration.yaml:200`
- `.github/actions/run-ansible-lint/action.yaml:43`
- `.github/actions/run-sanity/action.yaml:40`
- `.github/actions/run-unit-galaxy/action.yaml:44`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:38`
- `.github/actions/ansible_validate_changelog/action.yaml:22`

### github-env-injection (severity: high)

Multiple run: blocks write values derived from untrusted inputs to $GITHUB_OUTPUT or $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r'):
- ansible_lint.yaml: `echo "working_directory=${{ inputs.working_directory }}" >> $GITHUB_OUTPUT` and `echo "working_directory=${{ github.workspace }}" >> $GITHUB_OUTPUT` — unsanitized inputs written to GITHUB_OUTPUT.
- sonarcloud.yaml: ARGS string containing `${{ github.event.pull_request.head.ref }}`, `${{ github.event.pull_request.number }}`, `${{ github.event.pull_request.head.sha }}`, `${{ github.event.pull_request.base.ref }}` written to GITHUB_OUTPUT without sanitization.
- draft_release.yaml: `VERSION=${{ steps.release_drafter.outputs.tag_name }}` then `echo "VERSION=${VERSION#v}" >> $GITHUB_ENV` — step output written to GITHUB_ENV without sanitization.
- network_integration.yaml: `echo "CLABTITLE=${{ inputs.lab_title }}_PR${{ github.event.pull_request.number }}_..." >> $GITHUB_ENV` — inputs written to GITHUB_ENV without sanitization.
- run-sanity/action.yaml: `output=$(echo ${{ inputs.test-env }} | cut -d'-' -f2 | sed 's/py//'); echo "py_version=$output" >> $GITHUB_OUTPUT` — derived from unsanitized input.
- run-unit-galaxy/action.yaml: same pattern as run-sanity.
- generate-tox-ansible-matrix/action.yaml: `echo "envlist=$output" >> $GITHUB_OUTPUT` where $output is derived from `${{ inputs.scope }}`.

Locations:

- `.github/workflows/ansible_lint.yaml:34`
- `.github/workflows/sonarcloud.yaml:60`
- `.github/workflows/draft_release.yaml:30`
- `.github/workflows/network_integration.yaml:196`
- `.github/actions/run-sanity/action.yaml:28`
- `.github/actions/run-unit-galaxy/action.yaml:28`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:38`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and at least one job also lacks a `permissions:` key, meaning they run with the default (potentially write-all) token permissions:
- ack.yml: no permissions at file or job level.
- ansible_lint.yaml: no permissions at file or job level.
- build_import.yaml: no permissions at file or job level.
- changelog.yaml: no permissions at file or job level.
- cml_lab_create.yaml: no permissions at file or job level.
- cml_lab_destroy.yaml: no permissions at file or job level.
- cml_network_integration.yaml: no permissions at file or job level.
- draft_release.yaml: no permissions at file or job level.
- ee-build.yml: no permissions at file or job level.
- integration.yaml: no permissions at file or job level.
- network_integration.yaml: no permissions at file or job level.
- push.yml: no permissions at file or job level.
- refresh_ah_token.yaml: no permissions at file or job level.
- release.yaml: no permissions at file or job level.
- release_ah.yaml: no permissions at file or job level.
- release_galaxy.yaml: no permissions at file or job level.
- sanity.yaml: no permissions at file or job level.
- unit.yaml: no permissions at file or job level.
- upload_upstream_results.yaml: no permissions at file or job level.

Locations:

- `.github/workflows/ack.yml:1`
- `.github/workflows/ansible_lint.yaml:1`
- `.github/workflows/build_import.yaml:1`
- `.github/workflows/changelog.yaml:1`
- `.github/workflows/cml_lab_create.yaml:1`
- `.github/workflows/cml_lab_destroy.yaml:1`
- `.github/workflows/cml_network_integration.yaml:1`
- `.github/workflows/draft_release.yaml:1`
- `.github/workflows/ee-build.yml:1`
- `.github/workflows/integration.yaml:1`
- `.github/workflows/network_integration.yaml:1`
- `.github/workflows/push.yml:1`
- `.github/workflows/refresh_ah_token.yaml:1`
- `.github/workflows/release.yaml:1`
- `.github/workflows/release_ah.yaml:1`
- `.github/workflows/release_galaxy.yaml:1`
- `.github/workflows/sanity.yaml:1`
- `.github/workflows/unit.yaml:1`
- `.github/workflows/upload_upstream_results.yaml:1`

### unpinned-uses (severity: high)

Numerous `uses:` references across workflow files and composite action files use mutable tags or branch names (e.g. @v6, @v1, @v2, @v3, @v4, @v7, @v8, @main, @master, @release/v1) instead of pinned 40-character SHA commit hashes. This exposes the action to supply-chain attacks if the referenced tag or branch is moved or compromised. Affected files include all workflow files and composite action files. Examples of unpinned references found:
- actions/checkout@v6 (ansible_lint.yaml, build_import.yaml, changelog.yaml, check_label.yaml, ci.yaml, cml_lab_create.yaml, cml_lab_destroy.yaml, cml_network_integration.yaml, draft_release.yaml, ee-build.yml, integration.yaml, network_integration.yaml, release_ah.yaml, release_galaxy.yaml, sanity.yaml, unit.yaml)
- actions/setup-python@v6 (ansible_lint.yaml, ci.yaml, cml_lab_create.yaml, cml_lab_destroy.yaml, sanity.yaml, unit.yaml, run-ansible-lint/action.yaml, run-sanity/action.yaml, run-unit-galaxy/action.yaml, ansible_validate_changelog/action.yaml)
- release-drafter/release-drafter@v6 (check_label.yaml, draft_release.yaml)
- WyriHaximus/github-action-get-previous-tag@master (draft_release.yaml)
- SonarSource/sonarqube-scan-action@master (sonarcloud.yaml)
- jesusvasquez333/verify-pr-label-action@v1.4.0 (check_label.yaml)
- actions/add-to-project@main (check_label.yaml)
- ansible/team-devtools/.github/workflows/ack.yml@main (ack.yml)
- ansible/team-devtools/.github/workflows/push.yml@main (push.yml)
- ansible/ansible-content-actions/...@main (changelog.yaml, release.yaml)
- ansible-network/github_actions/...@main (cml_network_integration.yaml, network_integration.yaml)
- re-actors/alls-green@release/v1 (ci.yaml)
- redhat-actions/podman-login@v1, push-to-registry@v2 (ee-build.yml)
- coactions/upload-artifact@v4, actions/download-artifact@v7, actions/github-script@v8 (ee-build.yml, generate-tox-ansible-matrix/action.yaml, run-sanity/action.yaml)
- astral-sh/setup-uv@v7 (generate-tox-ansible-matrix/action.yaml)
- pre-commit/action@v3.0.1 (ci.yaml)

Locations:

- `.github/workflows/ansible_lint.yaml:22`
- `.github/workflows/build_import.yaml:9`
- `.github/workflows/changelog.yaml:10`
- `.github/workflows/check_label.yaml:12`
- `.github/workflows/ci.yaml:14`
- `.github/workflows/cml_lab_create.yaml:36`
- `.github/workflows/cml_lab_destroy.yaml:22`
- `.github/workflows/cml_network_integration.yaml:48`
- `.github/workflows/draft_release.yaml:18`
- `.github/workflows/ee-build.yml:22`
- `.github/workflows/integration.yaml:9`
- `.github/workflows/network_integration.yaml:60`
- `.github/workflows/release.yaml:24`
- `.github/workflows/release_ah.yaml:22`
- `.github/workflows/release_galaxy.yaml:18`
- `.github/workflows/sanity.yaml:12`
- `.github/workflows/sonarcloud.yaml:44`
- `.github/workflows/unit.yaml:14`
- `.github/workflows/ack.yml:13`
- `.github/workflows/push.yml:12`
- `.github/actions/ansible_validate_changelog/action.yaml:16`
- `.github/actions/generate-tox-ansible-matrix/action.yaml:18`
- `.github/actions/run-ansible-lint/action.yaml:32`
- `.github/actions/run-sanity/action.yaml:18`
- `.github/actions/run-unit-galaxy/action.yaml:18`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, missing-permissions, unpinned-uses

**Notes:**

Fixed all four finding types across all identified files:

1. script-injection: Moved all ${{ }} expressions from run: blocks into env: blocks and referenced them as plain environment variables. Used ${VAR:+"$VAR"} for optional positional arguments (e.g., ansible-lint args). Fixed in: ansible_lint.yaml, sonarcloud.yaml, draft_release.yaml, cml_network_integration.yaml, network_integration.yaml, run-ansible-lint/action.yaml, run-sanity/action.yaml, run-unit-galaxy/action.yaml, generate-tox-ansible-matrix/action.yaml, ansible_validate_changelog/action.yaml, integration.yaml, sanity.yaml, unit.yaml.

2. github-env-injection: Added printf '%s' ... | tr -d '\n\r' sanitization before writing to $GITHUB_OUTPUT and $GITHUB_ENV in: ansible_lint.yaml, sonarcloud.yaml, draft_release.yaml, network_integration.yaml, run-sanity/action.yaml, run-unit-galaxy/action.yaml, generate-tox-ansible-matrix/action.yaml.

3. missing-permissions: Added top-level permissions: blocks to all 19 workflow files that were missing them (ack.yml, ansible_lint.yaml, build_import.yaml, changelog.yaml, cml_lab_create.yaml, cml_lab_destroy.yaml, cml_network_integration.yaml, draft_release.yaml, ee-build.yml, integration.yaml, network_integration.yaml, push.yml, refresh_ah_token.yaml, release.yaml, release_ah.yaml, release_galaxy.yaml, sanity.yaml, unit.yaml, upload_upstream_results.yaml).

4. unpinned-uses: Pinned all 18 unique action references to full 40-character SHA hashes with original tag preserved as comment. All workflow files and composite actions updated.

### Iteration 2

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all 6 findings across 4 workflow files:

1. **release.yml - unpinned-uses**: Pinned `actions/checkout@v6` to full SHA `d23441a48e516b6c34aea4fa41551a30e30af803`.

2. **release.yml - script-injection**: Moved `${{ steps.version.outputs.major }}` and `${{ steps.version.outputs.tag }}` out of the `run:` block into an `env:` block as `MAJOR` and `TAG`, then referenced them as `"$MAJOR"` and `"$TAG"` in the shell script.

3. **draft_release.yaml - script-injection**: Fixed three locations: (a) `antsibull-changelog release` command now uses `$VERSION` env var instead of `${{ env.VERSION }}`; (b) `sed` command now uses `$VERSION` env var; (c) `Create PR for changelog` step restructured to use `$BRANCH_NAME` from env block with proper quoting on `git checkout -t -b "$BRANCH_NAME"` and `git push origin "$BRANCH_NAME"`.

4. **ee-build.yml - github-env-injection**: Added `printf '%s' ... | tr -d '\n\r'` sanitization before writing `IMAGE_TAG` to `$GITHUB_ENV` in the `Define environment variables for PR` step.

5. **ee-build.yml - script-injection**: Moved `${{ env.EE }}`, `${{ env.IMAGE_TAG }}`, and `${{ github.sha }}` into `env:` blocks as `EE_NAME`, `IMAGE_TAG_VAL`, and `GIT_SHA` in both the `(devel) Build image` and `Build image and create artifact` steps. All shell references now use properly quoted `"$EE_NAME"`, `"$IMAGE_TAG_VAL"`, `"$GIT_SHA"`.

6. **sonarcloud.yaml - script-injection**: Replaced `eval "$TEST_COMMAND"` with `bash -c "$TEST_COMMAND"` to avoid the dangerous `eval` pattern (the `TEST_COMMAND` was already correctly placed in the `env:` block).

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed three script-injection findings:
1. refresh_ah_token.yaml (line 28): Moved `${{ secrets.ah_token }}` out of the `run:` block into an `env:` variable `AH_TOKEN`, referenced as `"$AH_TOKEN"` in the curl command.
2. ee-build.yml (line 195): Moved four `${{ env.IMAGE_REGISTRY }}`, `${{ github.repository_owner }}`, `${{ env.EE }}`, and `${{ env.IMAGE_TAG }}` expressions in the 'Print summary' step into an `env:` block with names SUMMARY_IMAGE_REGISTRY, SUMMARY_REPO_OWNER, SUMMARY_EE, SUMMARY_IMAGE_TAG, and referenced them as plain shell variables.
3. sonarcloud.yaml (line 117): Replaced `bash -c "$TEST_COMMAND"` with writing the command to a temporary script file using `printf '%s\n' "$TEST_COMMAND" > _test_runner.sh` and executing it with `bash _test_runner.sh`, eliminating the `bash -c` injection vector.

### Iteration 4

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 5 findings across 3 workflow files:

1. sonarcloud.yaml (line 113): Replaced writing TEST_COMMAND to a shell script file and executing it with `read -ra cmd_args <<< "$TEST_COMMAND"` + `"${cmd_args[@]}"` to avoid executing untrusted file content as a shell script.

2. network_integration.yaml (line 291): Replaced vulnerable `sed -i "s/title: $NETWORK_OS/title: $CLABTITLE/"` with a Python heredoc using `str.replace()` to safely substitute values from environment variables, avoiding sed delimiter injection.

3. network_integration.yaml (line 171): Replaced `for spec in $INPUT_SPECS` loops (vulnerable to word splitting/glob expansion) with `while IFS= read -r spec; done <<< "$INPUT_SPECS"` loops and changed COLLECTIONS_TO_INSTALL to a bash array for safe element handling.

4. cml_network_integration.yaml (line 131): Replaced ansible command with unquoted variables in double-quoted strings with a jq-based approach that builds a proper JSON object for the -e flag, passing each value safely via --arg/--argjson parameters.

5. cml_network_integration.yaml (line 148): Added sanitization of $out_file before writing to GITHUB_OUTPUT using `printf '%s' "$out_file" | tr -d '\n\r'` to prevent newline injection.

### Iteration 5

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in 4 workflow files:
1. cml_lab_create.yaml (line 66): Added `safe_title=$(printf '%s' "$clean_title" | tr -d '\n\r')` before writing to GITHUB_OUTPUT.
2. cml_lab_destroy.yaml (line 54): Same fix for the 'Determine lab title' step.
3. sanity.yaml (line 52): Added `safe_merged=$(printf '%s' "$merged" | tr -d '\n\r')` before writing the merged matrix to GITHUB_OUTPUT.
4. unit.yaml (line 55): Same fix for the 'Merge matrix with extra entries' step.
In each case the sanitized variable (safe_title / safe_merged) is used in the echo statement instead of the original unsanitized variable, preventing newline-based injection into $GITHUB_OUTPUT.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Resolve topology path' step in .github/workflows/cml_lab_create.yaml. Added sanitization of the `topo_abs` value before writing to $GITHUB_OUTPUT: `safe_topo_abs=$(printf '%s' "$topo_abs" | tr -d '\n\r')` followed by `echo "path=$safe_topo_abs" >> "$GITHUB_OUTPUT"`. This prevents a malicious caller from injecting newlines via `inputs.topology_path` to write arbitrary key=value pairs into $GITHUB_OUTPUT.

