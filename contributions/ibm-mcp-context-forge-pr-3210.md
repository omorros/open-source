# IBM/mcp-context-forge — PR #3210 — Eliminated Playwright E2E test flakiness with JWT-first auth and HTMX-aware waits

**Repo:** [IBM/mcp-context-forge](https://github.com/IBM/mcp-context-forge)
**PR:** [#3210](https://github.com/IBM/mcp-context-forge/pull/3210)
**Issue:** [#3105](https://github.com/IBM/mcp-context-forge/issues/3105)
**Status:** merged
**Area:** testing
**Stack:** Python, Playwright, JWT
**Impact:** Playwright E2E tests run reliably across parallel workers without intermittent auth failures or timing-dependent flakiness.

---

## Context
MCP Context Forge includes Playwright-based end-to-end tests that run across multiple parallel workers. The tests authenticate against the admin UI and interact with HTMX-powered pages.

## Problem
- **Symptom:** E2E tests failed intermittently with authentication errors and timing-related assertion failures. Failures were non-deterministic and varied between runs.
- **Root cause:** Two issues: (1) Tests used shared mutable login state (`ADMIN_ACTIVE_PASSWORD`) which caused multi-worker routing conflicts when parallel workers attempted to authenticate simultaneously. (2) Hard-coded 4-second `sleep` calls were used instead of waiting for actual page readiness, causing flaky timing on slower CI environments.
- **Scope:** All Playwright E2E tests were affected. Failures were more frequent on CI than locally due to resource contention.

## Reproduction (before fix)
1. Run Playwright tests with multiple workers (`pytest --numprocesses=auto`)
2. Observe intermittent failures across different test files
3. Re-running the same tests may pass
- **Expected:** Consistent pass/fail results across runs
- **Actual:** Non-deterministic failures due to auth conflicts and timing sensitivity

## Fix / Changes
- Replaced shared mutable login state with per-fixture JWT cookie injection, eliminating multi-worker routing conflicts entirely
- Substituted all hard-coded 4-second `sleep` calls with DOM-based waiting that monitors in-flight HTMX requests: `wait_for_function("() => !document.querySelector('.htmx-request')")`
- Form-based login remains available via `PLAYWRIGHT_DISABLE_JWT_FALLBACK=true` for non-JWT environments
- Files changed: `tests/playwright/conftest.py`, `tests/playwright/pages/users_page.py`

## Testing / Verification
**Checks**
- Full Playwright suite passes consistently across multiple parallel workers
- No more intermittent failures observed

## Review notes
- **Feedback:** Maintainer recreated this PR from my original [#3112](https://github.com/IBM/mcp-context-forge/pull/3112) during repository maintenance, crediting me as co-author. Merged Feb 28, 2026.
