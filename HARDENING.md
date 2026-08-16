<!-- markdownlint-disable -->

# Hardening Report: cisagov--setup-env-github-action/v1.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cisagov--setup-env-github-action/v1.2.1** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references in every workflow file use mutable version tags instead of pinned 40-character SHA hashes, making the workflows vulnerable to supply-chain attacks if the referenced action tags are moved. Failing references include: cisagov/action-job-preamble@v1, cisagov/setup-env-github-action@v1, actions/checkout@v6 (and @v5), actions/setup-python@v6, actions/setup-go@v6, actions/cache@v5, hashicorp/setup-packer@v3, hashicorp/setup-terraform@v3, mxschmitt/action-tmate@v3, github/codeql-action/init@v4, github/codeql-action/autobuild@v4, github/codeql-action/analyze@v4, actions/dependency-review-action@v4, actions/labeler@v6, zyactions/semver@v1, crazy-max/ghaction-github-labeler@v5.

Locations:

- `.github/workflows/build.yml:40`
- `.github/workflows/codeql-analysis.yml:34`
- `.github/workflows/dependency-review.yml:27`
- `.github/workflows/label-prs.yml:26`
- `.github/workflows/release.yml:22`
- `.github/workflows/sync-labels.yml:14`

### script-injection (severity: high)

Rule (a): In release.yml, the 'Move tags' run block directly interpolates GitHub Actions expressions inside the shell script: `major_tag=v${{ steps.extract-semver-parts.outputs.major }}` and `major_minor_tag=${major_tag}.${{ steps.extract-semver-parts.outputs.minor }}`. These step outputs flow through YAML template substitution before the shell sees them, allowing an attacker who can influence the release tag to inject shell metacharacters.

Locations:

- `.github/workflows/release.yml:97`

### script-injection (severity: high)

Rule (b): In build.yml, four `go install` steps expand `${PACKAGE_URL}` and `${PACKAGE_VERSION}` without double-quoting. These env vars hold values sourced from `steps.setup-env.outputs.*` (workflow-controllable step outputs), so unquoted expansion allows shell metacharacter injection. Offending lines: `go install ${PACKAGE_URL}@${PACKAGE_VERSION}` in the Install go-critic, Install goimports, Install gosec, and Install staticcheck steps.

Locations:

- `.github/workflows/build.yml:119`
- `.github/workflows/build.yml:126`
- `.github/workflows/build.yml:133`
- `.github/workflows/build.yml:140`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all three findings across 6 workflow files:

1. **unpinned-uses**: Pinned all 17 action references across build.yml, codeql-analysis.yml, dependency-review.yml, label-prs.yml, release.yml, and sync-labels.yml to full 40-character SHA hashes using lookup_action_sha. Original tags preserved as inline comments.

2. **script-injection (release.yml)**: Moved `${{ steps.extract-semver-parts.outputs.major }}` and `${{ steps.extract-semver-parts.outputs.minor }}` from the `run:` shell block into the step's `env:` block as `SEMVER_MAJOR` and `SEMVER_MINOR`. The shell script now uses plain `${SEMVER_MAJOR}` and `${SEMVER_MINOR}` environment variable references.

3. **script-injection (build.yml)**: Added double-quotes around the `go install` arguments in all four Go tool install steps: `go install "${PACKAGE_URL}@${PACKAGE_VERSION}"`. The env vars were already correctly placed in the `env:` block; they just needed quoting to prevent shell metacharacter injection from workflow-controllable step outputs.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script-injection in .github/workflows/release.yml (move-tags step, line 103). All shell variable expansions of `${major_tag}` and `${major_minor_tag}` (derived from `SEMVER_MAJOR`/`SEMVER_MINOR` env vars) were unquoted, allowing shell metacharacter injection. Added double-quotes around all expansions: assignment lines (`major_tag="v${SEMVER_MAJOR}"`, `major_minor_tag="${major_tag}.${SEMVER_MINOR}"`), and all git command arguments (`git ls-remote`, `git push --delete`, `git tag`, `git push origin`). The env: block mapping was already correct (using `${{ }}` expressions in env vars rather than directly in the run script).

