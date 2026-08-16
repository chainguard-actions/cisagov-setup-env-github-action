<!-- markdownlint-disable -->

# Hardening Report: cisagov--setup-env-github-action/v1.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cisagov--setup-env-github-action/v1.3.1** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across every workflow file use mutable version tags instead of full 40-character SHA digests, making them vulnerable to supply-chain attacks if a tag is moved or a dependency is compromised. Affected references include: cisagov/action-job-preamble@v1, cisagov/setup-env-github-action@v1, actions/checkout@v6/@v5, actions/setup-python@v6, actions/setup-go@v6, actions/cache@v5, hashicorp/setup-packer@v3, hashicorp/setup-terraform@v4, mxschmitt/action-tmate@v3, github/codeql-action/init@v4, github/codeql-action/autobuild@v4, github/codeql-action/analyze@v4, actions/dependency-review-action@v4, actions/labeler@v6, zyactions/semver@v1, crazy-max/ghaction-github-labeler@v6, actions/github-script@v9.

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/codeql-analysis.yml:1`
- `.github/workflows/dependency-review.yml:1`
- `.github/workflows/label-prs.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/sync-labels.yml:1`
- `.github/workflows/verify.yml:1`

### script-injection (severity: high)

Sub-rule (a): The `run:` block in the `move-tags` step directly interpolates `${{ steps.extract-semver-parts.outputs.major }}` and `${{ steps.extract-semver-parts.outputs.minor }}` inside shell commands. These `steps.*.outputs.*` expressions are substituted by the YAML template engine before the shell ever sees them, allowing an attacker who can influence the semver action's outputs to inject arbitrary shell commands. Offending lines: `major_tag=v${{ steps.extract-semver-parts.outputs.major }}` and `major_minor_tag=${major_tag}.${{ steps.extract-semver-parts.outputs.minor }}`.

Locations:

- `.github/workflows/release.yml:96`

### script-injection (severity: high)

Sub-rule (b): Multiple `run:` steps in build.yml use unquoted shell variable expansions of env vars that hold `steps.*.outputs.*` values (workflow-controllable data). The pattern `go install ${PACKAGE_URL}@${PACKAGE_VERSION}` appears in four consecutive steps where PACKAGE_VERSION is set from `steps.setup-env.outputs.go-critic-version`, `steps.setup-env.outputs.goimports-version`, `steps.setup-env.outputs.gosec-version`, and `steps.setup-env.outputs.staticcheck-version`. Unquoted expansions allow shell metacharacter injection. The variables should be double-quoted: `go install "${PACKAGE_URL}@${PACKAGE_VERSION}"`.

Locations:

- `.github/workflows/build.yml:148`
- `.github/workflows/build.yml:155`
- `.github/workflows/build.yml:162`
- `.github/workflows/build.yml:169`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all three findings across 7 workflow files:

1. **unpinned-uses**: Pinned all `uses:` references to full 40-character SHA digests with original tags preserved as comments. Actions pinned: cisagov/action-job-preamble@v1→f3e1d43, cisagov/setup-env-github-action@v1→55d9b21, actions/checkout@v5→93cb6ef, actions/checkout@v6→df4cb1c, actions/setup-python@v6→ece7cb0, actions/setup-go@v6→924ae3a, actions/cache@v5→caa2961, hashicorp/setup-packer@v3→ce93c3c, hashicorp/setup-terraform@v4→dfe3c3f, mxschmitt/action-tmate@v3→35b54af, github/codeql-action/{init,autobuild,analyze}@v4→7188fc3, actions/dependency-review-action@v4→2031cfc, actions/labeler@v6→b8dd2d9, zyactions/semver@v1→add7fdd, crazy-max/ghaction-github-labeler@v6→548a7c3, actions/github-script@v9→3a2844b.

2. **script-injection (a)**: In release.yml `move-tags` step, moved `${{ steps.extract-semver-parts.outputs.major }}` and `${{ steps.extract-semver-parts.outputs.minor }}` out of the `run:` block into the step's `env:` block as `SEMVER_MAJOR` and `SEMVER_MINOR`. The shell script now uses plain `${SEMVER_MAJOR}` and `${SEMVER_MINOR}` variables.

3. **script-injection (b)**: In build.yml, the four `go install ${PACKAGE_URL}@${PACKAGE_VERSION}` commands now use double-quoted `"${PACKAGE_URL}@${PACKAGE_VERSION}"` to prevent shell metacharacter injection from unquoted variable expansions.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted variable expansions in the `move-tags` step of `.github/workflows/release.yml`. All occurrences of `${SEMVER_MAJOR}`, `${SEMVER_MINOR}`, `${major_tag}`, and `${major_minor_tag}` in the `run:` block are now wrapped in double quotes. This prevents shell metacharacters in workflow-controllable step output values from being interpreted by the shell, eliminating the command injection risk.

