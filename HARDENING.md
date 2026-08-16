<!-- markdownlint-disable -->

# Hardening Report: cisagov--setup-env-github-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cisagov--setup-env-github-action/v2.0.0** was hardened automatically. 2 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Every `uses:` reference across all workflow files is pinned to a mutable tag or version string rather than an immutable 40-character SHA commit hash. This exposes the workflows to supply-chain attacks if any referenced action is compromised or its tag is moved. Affected references include: cisagov/action-job-preamble@v1, cisagov/setup-env-github-action@v1, actions/checkout@v6 (and @v5), actions/setup-python@v6, actions/setup-go@v6, actions/cache@v5, hashicorp/setup-packer@v3, hashicorp/setup-terraform@v4, mxschmitt/action-tmate@v3, github/codeql-action/init@v4, github/codeql-action/autobuild@v4, github/codeql-action/analyze@v4, actions/dependency-review-action@v4, actions/labeler@v6, zyactions/semver@v1, crazy-max/ghaction-github-labeler@v6, actions/github-script@v9.

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/codeql-analysis.yml:1`
- `.github/workflows/dependency-review.yml:1`
- `.github/workflows/label-prs.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/sync-labels.yml:1`
- `.github/workflows/verify.yml:1`

### script-injection (severity: high)

Sub-rule (a) violation: The `move-tags` step in the `release` job directly interpolates `${{ steps.extract-semver-parts.outputs.major }}` and `${{ steps.extract-semver-parts.outputs.minor }}` inside a `run:` shell script. These `steps.*.outputs.*` values flow through YAML template substitution before the shell processes them, allowing an attacker who can influence the semver action's outputs to inject arbitrary shell commands. Offending lines: `major_tag=v${{ steps.extract-semver-parts.outputs.major }}` and `major_minor_tag=${major_tag}.${{ steps.extract-semver-parts.outputs.minor }}`. These values should be passed via an `env:` block and then referenced as double-quoted shell variables (e.g., `"$MAJOR"`).

Locations:

- `.github/workflows/release.yml:106`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all unpinned `uses:` references across 7 workflow files (build.yml, codeql-analysis.yml, dependency-review.yml, label-prs.yml, release.yml, sync-labels.yml, verify.yml) by pinning each to its full 40-character SHA commit hash with the original tag preserved as a comment. Fixed the script-injection vulnerability in release.yml's `move-tags` step by moving `${{ steps.extract-semver-parts.outputs.major }}` and `${{ steps.extract-semver-parts.outputs.minor }}` into an `env:` block as `MAJOR` and `MINOR` variables, then referencing them as double-quoted shell variables (`"$MAJOR"` and `"$MINOR"`) in the run script.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all four `go install` commands in `.github/workflows/build.yml` (Install go-critic, goimports, gosec, and staticcheck steps) by wrapping the unquoted `${PACKAGE_URL}@${PACKAGE_VERSION}` expansions in double quotes: `go install "${PACKAGE_URL}@${PACKAGE_VERSION}"`. This prevents shell metacharacter injection via the workflow-controllable `PACKAGE_VERSION` environment variable sourced from `steps.setup-env.outputs.*`.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed the move-tags step in .github/workflows/release.yml by double-quoting all occurrences of ${major_tag} and ${major_minor_tag} in shell commands. The assignment of major_minor_tag was also fixed from the mixed-quoting form `${major_tag}."$MINOR"` to the properly double-quoted form `"${major_tag}.$MINOR"`. This prevents shell metacharacters in the workflow-controllable MAJOR and MINOR env vars (derived from steps.extract-semver-parts.outputs.*) from being interpreted by the shell.

