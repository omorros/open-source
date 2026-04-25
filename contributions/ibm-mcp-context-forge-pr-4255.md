# IBM/mcp-context-forge — PR #4255 — Established TypeScript Playwright E2E suite for the React UI rewrite

**Repo:** [IBM/mcp-context-forge](https://github.com/IBM/mcp-context-forge)
**PR:** [#4255](https://github.com/IBM/mcp-context-forge/pull/4255)
**Issue:** [#4246](https://github.com/IBM/mcp-context-forge/issues/4246)
**Status:** merged
**Area:** testing
**Stack:** TypeScript, Playwright, React, Vite, Vitest, GitHub Actions
**Impact:** The new React admin UI on `epic/ui-rewrite` ships with a TypeScript Playwright E2E suite and dedicated CI workflow, isolated from the existing Python/HTMX tests so the rewrite has its own regression net from day one.

---

## Context
MCP Context Forge is rewriting the admin UI as a React app under `client/` on the `epic/ui-rewrite` branch. The legacy HTMX UI has Python-based Playwright tests, but they target HTMX/Jinja templates and don't apply to the React surface. Issue #4246 asked for a TypeScript Playwright suite that lives alongside the React code, shares its types, and runs as its own CI workflow.

## Problem
- **Symptom:** The new React UI had no E2E coverage. The existing Python Playwright suite couldn't be reused — different stack, different routes, different selectors, and no React-aware waits.
- **Root cause:** No prior E2E infrastructure existed under `client/`. The rewrite branch needed its own test runner, its own CI workflow, and a backend-mocking strategy so tests could run without spinning up the Python gateway.
- **Scope:** `client/` only. The legacy HTMX tests under `tests/playwright/` were left untouched.

## Reproduction (before fix)
1. Check out `epic/ui-rewrite`
2. Look for an E2E entry point under `client/`
- **Expected:** A Playwright suite plus CI workflow covering smoke and auth flows
- **Actual:** No suite, no workflow — the React UI had unit tests (Vitest) only

## Fix / Changes
- Added Playwright config at `client/playwright.config.ts` with a `webServer` block that boots Vite for the suite, plus a dedicated `client/tsconfig.e2e.json` so E2E types stay isolated from the app build
- Created `client/e2e/` with three layers:
  - `smoke/` — `app-loads.spec.ts`, `auth-redirect.spec.ts`, `static-assets.spec.ts` for fast unauthenticated checks
  - `auth/` — `login-flow.spec.ts`, `forgot-password.spec.ts`, `session.spec.ts` for authenticated flows
  - `fixtures/` — `api-mock.ts` (backend mocked via `page.route()`) and `auth.ts` (auth-state fixtures)
- Added `client/e2e/utils/paths.ts` and a `client/e2e/README.md` documenting the suite layout and run commands
- Added `.github/workflows/client-e2e.yml` with `npm` scripts (`e2e`, `e2e:ui`, `e2e:debug`) wired into `client/package.json`
- Pinned `actions/cache` to a SHA in the new workflow to satisfy IBM's allowlist policy (every other workflow in the repo pins by SHA)
- Added `playwright-report/` and `test-results/` to `.gitignore` and `client/.prettierignore`

## Testing / Verification
**Commands**
- `npm run lint`
- `npm run test` (Vitest)
- `npm run e2e` (Playwright)

**Checks**
- 13/13 Playwright specs pass plus 1/1 Vitest, full run in ~13.4s locally
- Lint suite green
- New `client-e2e.yml` workflow runs green on the merge commit after the SHA-pin fix

## Review notes
- **Feedback:** `marekdano` flagged that the first run of `Client E2E (Playwright)` failed and asked whether a checkout step was missing relative to `Lint & Test Client`. Also asked whether the suite should hit real API endpoints instead of mocking.
- **Iteration:** Pointed out that the checkout step was already in the workflow (lines 48–52); the real cause was an unpinned `actions/cache@v4`, which IBM's allowlist rejects. Fixed by pinning to SHA in commit `038fe4b`. On the API-mocking question, agreed to keep the mocked-backend smoke suite as-is for speed and scoped real-endpoint coverage as a follow-up.
- **Approval:** `gcgoncalves` approved and merged on Apr 24, 2026; `marekdano` confirmed *"keep the smoke test with the mock data now"* after the SHA-pin fix.
