<!-- markdownlint-disable -->

# Hardening Report: github--branch-deploy/v11.1.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **github--branch-deploy/v11.1.4** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable version tags instead of full 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the tag is moved to a malicious commit. Unpinned references found:
- actions/checkout@v6 (in actions-config-validation.yml, copilot-setup-steps.yml, lint.yml, package-check.yml, test.yml, update-latest-release-tag.yml)
- actions/setup-node@v6 (in copilot-setup-steps.yml, lint.yml, package-check.yml, test.yml)
All should be replaced with their full SHA digest, e.g. `actions/checkout@<40-char-sha> # v6`.

Locations:

- `.github/workflows/actions-config-validation.yml:17`
- `.github/workflows/copilot-setup-steps.yml:14`
- `.github/workflows/copilot-setup-steps.yml:18`
- `.github/workflows/lint.yml:14`
- `.github/workflows/lint.yml:18`
- `.github/workflows/package-check.yml:15`
- `.github/workflows/package-check.yml:19`
- `.github/workflows/test.yml:14`
- `.github/workflows/test.yml:18`
- `.github/workflows/update-latest-release-tag.yml:21`

### script-injection (severity: high)

Rule (b) violation: In update-latest-release-tag.yml, the `run:` steps expand shell variables that hold user-controlled `workflow_dispatch` inputs without double-quoting them. The variables SOURCE_TAG and MAJOR_VERSION_TAG are set from `github.event.inputs.source_tag` and `github.event.inputs.major_version_tag` respectively. Using unquoted `${MAJOR_VERSION_TAG}` and `${SOURCE_TAG}` in shell commands allows an attacker to inject shell metacharacters (`;`, `|`, `&`, etc.) via the workflow_dispatch input fields.

Offending lines:
  `run: git tag -f ${MAJOR_VERSION_TAG} ${SOURCE_TAG}`  — both variables unquoted
  `run: git push origin ${MAJOR_VERSION_TAG} --force`   — variable unquoted

Fix: quote all expansions, e.g. `git tag -f "${MAJOR_VERSION_TAG}" "${SOURCE_TAG}"`.

Locations:

- `.github/workflows/update-latest-release-tag.yml:34`
- `.github/workflows/update-latest-release-tag.yml:39`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all unpinned action references across 5 workflow files: actions/checkout@v6 → pinned to SHA d23441a48e516b6c34aea4fa41551a30e30af803 and actions/setup-node@v6 → pinned to SHA 249970729cb0ef3589644e2896645e5dc5ba9c38. Fixed script injection in update-latest-release-tag.yml by double-quoting ${MAJOR_VERSION_TAG} and ${SOURCE_TAG} in the git tag and git push run steps. The ${{ github.event.inputs.* }} expressions were already correctly placed in the env: block; only the shell variable expansions needed quoting.

