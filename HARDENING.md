<!-- markdownlint-disable -->

# Hardening Report: cisagov--setup-env-github-action/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cisagov--setup-env-github-action/v1.2.0** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in a run: shell block. In the 'move-tags' step, `${{ steps.extract-semver-parts.outputs.major }}` and `${{ steps.extract-semver-parts.outputs.minor }}` are interpolated directly into shell commands: `major_tag=v${{ steps.extract-semver-parts.outputs.major }}` and `major_minor_tag=${major_tag}.${{ steps.extract-semver-parts.outputs.minor }}`. These step outputs flow through YAML template substitution before the shell sees them, enabling command injection if the semver action produces malicious output.

Locations:

- `.github/workflows/release.yml:107`
- `.github/workflows/release.yml:108`

### script-injection (severity: high)

Sub-rule (b): Unquoted shell variable expansion of workflow-controllable data. In the 'Install go-critic', 'Install goimports', 'Install gosec', and 'Install staticcheck' steps, `${PACKAGE_URL}` and `${PACKAGE_VERSION}` are used unquoted in `go install ${PACKAGE_URL}@${PACKAGE_VERSION}`. These env vars are sourced from `steps.setup-env.outputs.*` (workflow-controllable step outputs). Additionally, in the 'Clone ATX headers branch from terraform-docs fork' step, `$TERRAFORM_DOCS_REPO_BRANCH_NAME`, `$TERRAFORM_DOCS_REPO_DEPTH`, and `$TERRAFORM_DOCS_REPO_URL` are used unquoted in a `git clone` command; these are set in the top-level `env:` block and are workflow-controllable. All these unquoted expansions allow shell metacharacter injection.

Locations:

- `.github/workflows/build.yml:163`
- `.github/workflows/build.yml:169`
- `.github/workflows/build.yml:175`
- `.github/workflows/build.yml:181`
- `.github/workflows/build.yml:191`
- `.github/workflows/build.yml:192`
- `.github/workflows/build.yml:193`

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable version tags instead of full 40-character commit SHA digests. This exposes the workflow to supply-chain attacks if any referenced action's tag is moved or compromised. Unpinned references found: cisagov/action-job-preamble@v1, cisagov/setup-env-github-action@v1, actions/checkout@v5, actions/setup-python@v6, actions/setup-go@v6, actions/cache@v4, hashicorp/setup-packer@v3, hashicorp/setup-terraform@v3, mxschmitt/action-tmate@v3.

Locations:

- `.github/workflows/build.yml:36`
- `.github/workflows/build.yml:72`
- `.github/workflows/build.yml:100`
- `.github/workflows/build.yml:104`
- `.github/workflows/build.yml:110`
- `.github/workflows/build.yml:120`
- `.github/workflows/build.yml:127`
- `.github/workflows/build.yml:155`
- `.github/workflows/build.yml:159`
- `.github/workflows/build.yml:213`

### unpinned-uses (severity: high)

Workflow references GitHub Actions using mutable version tags instead of full 40-character commit SHA digests. Unpinned references found: cisagov/action-job-preamble@v1, actions/checkout@v5, github/codeql-action/init@v3, github/codeql-action/autobuild@v3, github/codeql-action/analyze@v3.

Locations:

- `.github/workflows/codeql-analysis.yml:32`
- `.github/workflows/codeql-analysis.yml:68`
- `.github/workflows/codeql-analysis.yml:100`
- `.github/workflows/codeql-analysis.yml:108`
- `.github/workflows/codeql-analysis.yml:116`
- `.github/workflows/codeql-analysis.yml:128`

### unpinned-uses (severity: high)

Workflow references GitHub Actions using mutable version tags instead of full 40-character commit SHA digests. Unpinned references found: cisagov/action-job-preamble@v1, actions/checkout@v5, actions/dependency-review-action@v4.

Locations:

- `.github/workflows/dependency-review.yml:27`
- `.github/workflows/dependency-review.yml:60`
- `.github/workflows/dependency-review.yml:88`
- `.github/workflows/dependency-review.yml:93`

### unpinned-uses (severity: high)

Workflow references GitHub Actions using mutable version tags instead of full 40-character commit SHA digests. Unpinned references found: cisagov/action-job-preamble@v1, actions/labeler@v6.

Locations:

- `.github/workflows/label-prs.yml:27`
- `.github/workflows/label-prs.yml:60`
- `.github/workflows/label-prs.yml:88`

### unpinned-uses (severity: high)

Workflow references GitHub Actions using mutable version tags instead of full 40-character commit SHA digests. Unpinned references found: cisagov/action-job-preamble@v1, zyactions/semver@v1, actions/checkout@v5.

Locations:

- `.github/workflows/release.yml:27`
- `.github/workflows/release.yml:60`
- `.github/workflows/release.yml:88`
- `.github/workflows/release.yml:93`

### unpinned-uses (severity: high)

Workflow references GitHub Actions using mutable version tags instead of full 40-character commit SHA digests. Unpinned references found: cisagov/action-job-preamble@v1, actions/checkout@v5, crazy-max/ghaction-github-labeler@v5.

Locations:

- `.github/workflows/sync-labels.yml:27`
- `.github/workflows/sync-labels.yml:60`
- `.github/workflows/sync-labels.yml:88`
- `.github/workflows/sync-labels.yml:91`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed all findings across 6 workflow files:

1. script-injection (release.yml): Moved ${{ steps.extract-semver-parts.outputs.major }} and ${{ steps.extract-semver-parts.outputs.minor }} into the step's env: block as SEMVER_MAJOR and SEMVER_MINOR. All shell variable expansions in the run block are now double-quoted.

2. script-injection (build.yml): Quoted ${PACKAGE_URL}@${PACKAGE_VERSION} in all four go install commands, and quoted $TERRAFORM_DOCS_REPO_BRANCH_NAME, $TERRAFORM_DOCS_REPO_DEPTH, and $TERRAFORM_DOCS_REPO_URL in the git clone command.

3. unpinned-uses (build.yml): Pinned cisagov/action-job-preamble@v1→f3e1d43, cisagov/setup-env-github-action@v1→55d9b21, actions/checkout@v5→fbc6f39, actions/setup-python@v6→ece7cb0, actions/setup-go@v6→924ae3a, actions/cache@v4→0057852, hashicorp/setup-packer@v3→ce93c3c, hashicorp/setup-terraform@v3→b9cd54a, mxschmitt/action-tmate@v3→35b54af.

4. unpinned-uses (codeql-analysis.yml): Pinned cisagov/action-job-preamble@v1→f3e1d43, actions/checkout@v5→fbc6f39, github/codeql-action/init@v3→4187e74, github/codeql-action/autobuild@v3→4187e74, github/codeql-action/analyze@v3→4187e74.

5. unpinned-uses (dependency-review.yml): Pinned cisagov/action-job-preamble@v1→f3e1d43 (×2), actions/checkout@v5→fbc6f39, actions/dependency-review-action@v4→2031cfc.

6. unpinned-uses (label-prs.yml): Pinned cisagov/action-job-preamble@v1→f3e1d43 (×2), actions/labeler@v6→b8dd2d9.

7. unpinned-uses (release.yml): Pinned cisagov/action-job-preamble@v1→f3e1d43 (×2), zyactions/semver@v1→add7fdd, actions/checkout@v5→fbc6f39.

8. unpinned-uses (sync-labels.yml): Pinned cisagov/action-job-preamble@v1→f3e1d43 (×2), actions/checkout@v5→fbc6f39, crazy-max/ghaction-github-labeler@v5→24d110a.

