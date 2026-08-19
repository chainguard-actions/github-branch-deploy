<!-- markdownlint-disable -->

# Hardening Report: github--branch-deploy/v11.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **github--branch-deploy/v11.1.0** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable version tags instead of immutable 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved.

Failing references:
- `actions/checkout@v5` in actions-config-validation.yml, codeql-analysis.yml, copilot-setup-steps.yml, lint.yml, package-check.yml, test.yml, update-latest-release-tag.yml
- `actions/setup-node@v5` in copilot-setup-steps.yml, lint.yml, package-check.yml, test.yml
- `github/codeql-action/init@v3`, `github/codeql-action/autobuild@v3`, `github/codeql-action/analyze@v3` in codeql-analysis.yml

Locations:

- `.github/workflows/actions-config-validation.yml:18`
- `.github/workflows/codeql-analysis.yml:26`
- `.github/workflows/codeql-analysis.yml:31`
- `.github/workflows/codeql-analysis.yml:35`
- `.github/workflows/codeql-analysis.yml:38`
- `.github/workflows/copilot-setup-steps.yml:14`
- `.github/workflows/copilot-setup-steps.yml:19`
- `.github/workflows/lint.yml:14`
- `.github/workflows/lint.yml:19`
- `.github/workflows/package-check.yml:14`
- `.github/workflows/package-check.yml:19`
- `.github/workflows/test.yml:14`
- `.github/workflows/test.yml:19`
- `.github/workflows/update-latest-release-tag.yml:22`

### script-injection (severity: high)

Rule (b) violation: In `update-latest-release-tag.yml`, the `run:` steps expand shell variables `${MAJOR_VERSION_TAG}` and `${SOURCE_TAG}` without double-quoting them. These variables are set from `github.event.inputs.source_tag` and `github.event.inputs.major_version_tag`, which are user-controlled via `workflow_dispatch`. An attacker with permission to trigger the workflow could inject shell metacharacters (e.g. spaces, semicolons, backticks) through these inputs.

Offending lines:
- `run: git tag -f ${MAJOR_VERSION_TAG} ${SOURCE_TAG}` (line 35) — both variables unquoted
- `run: git push origin ${MAJOR_VERSION_TAG} --force` (line 41) — variable unquoted

Fix: quote the expansions: `"${MAJOR_VERSION_TAG}"` and `"${SOURCE_TAG}"`.

Locations:

- `.github/workflows/update-latest-release-tag.yml:35`
- `.github/workflows/update-latest-release-tag.yml:41`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Pinned all mutable action tags to full 40-char commit SHAs: actions/checkout@v5 → 93cb6efe..., actions/setup-node@v5 → a0853c24..., github/codeql-action/{init,autobuild,analyze}@v3 → b7351df7... across all 7 affected workflow files. Fixed script-injection in update-latest-release-tag.yml by double-quoting ${MAJOR_VERSION_TAG} and ${SOURCE_TAG} shell variable expansions in both run steps (lines 35 and 41). The codeql-analysis.yml was rewritten after it became corrupted during sequential edits.

### Iteration 2

**Fixes applied:** unpinned-uses

**Notes:**

Pinned `actions/setup-node@v5` to full commit SHA `a0853c24544627f65ddf259abe73b1d18a591444` in both `.github/workflows/copilot-setup-steps.yml` and `.github/workflows/package-check.yml`. Both files also had corrupted/garbled content (mixed lines from a partial prior edit) which was cleaned up during the rewrite. The human-readable `# v5` comment was preserved alongside the pinned SHA for readability.

