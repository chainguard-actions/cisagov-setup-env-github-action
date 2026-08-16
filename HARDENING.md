<!-- markdownlint-disable -->

# Hardening Report: cisagov--setup-env-github-action/v1.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cisagov--setup-env-github-action/v1.2.2** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across every workflow file use mutable version tags instead of pinned 40-character SHA commit hashes, making the workflows vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved.

Failing references include (non-exhaustive):
- `cisagov/action-job-preamble@v1` (all workflow files)
- `cisagov/setup-env-github-action@v1` (build.yml)
- `actions/checkout@v6` (build.yml, codeql-analysis.yml, dependency-review.yml, sync-labels.yml, verify.yml)
- `actions/checkout@v5` (release.yml)
- `actions/setup-python@v6` (build.yml)
- `actions/setup-go@v6` (build.yml)
- `actions/cache@v5` (build.yml)
- `hashicorp/setup-packer@v3` (build.yml)
- `hashicorp/setup-terraform@v4` (build.yml)
- `mxschmitt/action-tmate@v3` (build.yml)
- `github/codeql-action/init@v4` (codeql-analysis.yml)
- `github/codeql-action/autobuild@v4` (codeql-analysis.yml)
- `github/codeql-action/analyze@v4` (codeql-analysis.yml)
- `actions/dependency-review-action@v4` (dependency-review.yml)
- `actions/labeler@v6` (label-prs.yml)
- `zyactions/semver@v1` (release.yml)
- `crazy-max/ghaction-github-labeler@v6` (sync-labels.yml)
- `actions/github-script@v8` (verify.yml)

Locations:

- `.github/workflows/build.yml:43`
- `.github/workflows/build.yml:83`
- `.github/workflows/build.yml:87`
- `.github/workflows/build.yml:91`
- `.github/workflows/build.yml:96`
- `.github/workflows/build.yml:103`
- `.github/workflows/build.yml:113`
- `.github/workflows/build.yml:117`
- `.github/workflows/build.yml:168`
- `.github/workflows/codeql-analysis.yml:34`
- `.github/workflows/codeql-analysis.yml:72`
- `.github/workflows/codeql-analysis.yml:82`
- `.github/workflows/codeql-analysis.yml:90`
- `.github/workflows/codeql-analysis.yml:97`
- `.github/workflows/codeql-analysis.yml:107`
- `.github/workflows/dependency-review.yml:27`
- `.github/workflows/dependency-review.yml:62`
- `.github/workflows/dependency-review.yml:68`
- `.github/workflows/dependency-review.yml:72`
- `.github/workflows/label-prs.yml:25`
- `.github/workflows/label-prs.yml:60`
- `.github/workflows/label-prs.yml:72`
- `.github/workflows/release.yml:20`
- `.github/workflows/release.yml:55`
- `.github/workflows/release.yml:68`
- `.github/workflows/release.yml:74`
- `.github/workflows/sync-labels.yml:14`
- `.github/workflows/sync-labels.yml:49`
- `.github/workflows/sync-labels.yml:60`
- `.github/workflows/sync-labels.yml:63`
- `.github/workflows/verify.yml:31`
- `.github/workflows/verify.yml:66`
- `.github/workflows/verify.yml:72`
- `.github/workflows/verify.yml:82`

### script-injection (severity: high)

Sub-rule (a): The 'Move tags' step in release.yml directly interpolates `${{ steps.extract-semver-parts.outputs.major }}` and `${{ steps.extract-semver-parts.outputs.minor }}` inside a `run:` shell script. These `steps.*.outputs.*` context values are substituted by the GitHub Actions template engine before the shell parses the command, allowing an attacker who can influence the semver action's outputs to inject arbitrary shell commands.

Offending lines:
  major_tag=v${{ steps.extract-semver-parts.outputs.major }}
  major_minor_tag=${major_tag}.${{ steps.extract-semver-parts.outputs.minor }}

Locations:

- `.github/workflows/release.yml:79`

### script-injection (severity: high)

Sub-rule (b): The 'Install go-critic', 'Install goimports', 'Install gosec', and 'Install staticcheck' steps in build.yml expand `${PACKAGE_VERSION}` unquoted inside `run:` commands. `PACKAGE_VERSION` is sourced from `steps.setup-env.outputs.*` (a `steps.*.outputs.*` context), which is a workflow-controllable value. The unquoted shell expansion `go install ${PACKAGE_URL}@${PACKAGE_VERSION}` allows shell metacharacters in the version string to be interpreted by the shell.

Offending pattern (repeated for each tool):
  run: go install ${PACKAGE_URL}@${PACKAGE_VERSION}

Locations:

- `.github/workflows/build.yml:126`
- `.github/workflows/build.yml:132`
- `.github/workflows/build.yml:138`
- `.github/workflows/build.yml:144`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all unpinned `uses:` references across 6 workflow files (build.yml, codeql-analysis.yml, dependency-review.yml, label-prs.yml, release.yml, sync-labels.yml, verify.yml) by pinning each to its full 40-character SHA with the original tag preserved as a comment. Fixed script injection in release.yml by moving `${{ steps.extract-semver-parts.outputs.major }}` and `${{ steps.extract-semver-parts.outputs.minor }}` into the step's `env:` block (as SEMVER_MAJOR and SEMVER_MINOR) and referencing them as plain shell variables. Fixed script injection in build.yml by double-quoting the `go install ${PACKAGE_URL}@${PACKAGE_VERSION}` commands so shell metacharacters in PACKAGE_VERSION cannot be interpreted as shell commands.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed the script-injection finding in .github/workflows/release.yml by double-quoting all expansions of ${major_tag} and ${major_minor_tag} in the move-tags step. The variables were already properly sourced via the env: block (SEMVER_MAJOR and SEMVER_MINOR), but the derived shell variables major_tag and major_minor_tag were used unquoted in 7 git commands. All expansions are now double-quoted: git ls-remote, git push --delete (×2 each), git tag (×2), and git push origin.

