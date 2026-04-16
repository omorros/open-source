# IBM/mcp-context-forge — PR #4242 — Fixed docker-release workflow failing on every release via check-runs API and tag peeling

**Repo:** [IBM/mcp-context-forge](https://github.com/IBM/mcp-context-forge)
**PR:** [#4242](https://github.com/IBM/mcp-context-forge/pull/4242)
**Issue:** [#4200](https://github.com/IBM/mcp-context-forge/issues/4200)
**Status:** merged
**Area:** bugfix / CI
**Stack:** GitHub Actions, Bash, GitHub REST API
**Impact:** The `docker-release` workflow no longer aborts on every release tag — annotated tags resolve to their underlying commit SHA and pre-flight checks query the modern check-runs API instead of the legacy commit-status endpoint.

---

## Context
MCP Context Forge publishes multi-arch Docker images on release via `.github/workflows/docker-release.yml`. The workflow runs a pre-flight "Verify commit checks passed" step before re-tagging images for the release, and resolves the release tag to a commit SHA before querying that SHA's CI state.

## Problem
- **Symptom:** `docker-release.yml` failed at the "Verify commit checks passed" step on every release tag (RC-1, RC-2, RC-3), aborting the re-tag job with `{"state":"pending","statuses":[]}`.
- **Root cause:** Two bugs compounded. (1) The step queried the legacy `GET /commits/{sha}/status` endpoint, which only aggregates external commit-statuses (Travis/Jenkins/Codacy) — this repo uses GitHub Actions check-runs, so the legacy endpoint always returned `pending` with an empty list. (2) `git ls-remote --refs` on an annotated tag returns the tag-object SHA, not the underlying commit SHA, so even the correct endpoint would have been queried against the wrong SHA.
- **Scope:** Only the `docker-release` workflow was affected. The main build workflow already produced correctly tagged multi-arch images on GHCR, so the failure was noisy rather than release-blocking — but every release triggered a red run.

## Reproduction (before fix)
1. Push an annotated release tag (e.g. `v1.0.0-RC-3`)
2. Observe the `docker-release` workflow run
3. Wait for the "Verify commit checks passed" step
- **Expected:** Pre-flight check sees the completed GitHub Actions check-runs and passes
- **Actual:** Step receives `{"state":"pending","statuses":[]}` from the legacy API and aborts the job

## Fix / Changes
- Peeled annotated tags to their commit SHA using `git ls-remote refs/tags/$TAG^{}` with fallbacks for lightweight tags
- Replaced the legacy status endpoint with `GET /commits/{sha}/check-runs?per_page=100`, iterating only on completed runs with non-successful conclusions and ignoring runs still in progress
- Guarded against missing `.check_runs` keys and null JSON responses to avoid silent no-op successes
- Narrowed the job's permission from `statuses: read` to `checks: read` to match the new API
- Hardened the second commit against shell injection by validating the resolved SHA with a regex, deduplicating check-run entries, handling pagination, and surfacing API errors instead of masking them

## Testing / Verification
**Checks**
- Verified against the live upstream API that annotated tag `v1.0.0-RC-3` peels to the expected commit SHA
- Confirmed the check-runs endpoint returns 51 runs for that commit, all with `conclusion: success`
- Dry-ran the updated workflow logic locally with `gh api` to confirm no in-progress runs were treated as failures

## Review notes
- **Feedback:** Collaborator `jonpspri` replied *"Thanks. This came up in our release work but we hadn't had the time to address it yet. Reviewing and rebasing now."* and merged the PR after rebase.
