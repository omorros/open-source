# IBM/mcp-context-forge — PR #3370 — Eliminated Playwright agents modal test flake by removing legacy hidden table

**Repo:** [IBM/mcp-context-forge](https://github.com/IBM/mcp-context-forge)
**PR:** [#3370](https://github.com/IBM/mcp-context-forge/pull/3370)
**Issue:** [#3333](https://github.com/IBM/mcp-context-forge/issues/3333)
**Status:** merged
**Area:** testing
**Stack:** Python, Playwright, HTML, HTMX
**Impact:** The `test_view_modal_different_agents_show_different_data` test no longer flakes — passed 5/5 consecutive runs against a live gateway.

---

## Context
MCP Context Forge's admin UI uses HTMX to load the agents table dynamically. Playwright E2E tests interact with agent rows via modal views (View/Edit/Test). The `admin.html` template still contained a 159-line legacy `<table>` block hidden with `style="display: none;"` alongside the active HTMX-loaded table.

## Problem
- **Symptom:** `test_view_modal_different_agents_show_different_data` timed out intermittently — View buttons existed but were not actionable.
- **Root cause:** `agents_table` used `self.agents_panel.locator("table")` which matched both the HTMX table and the hidden legacy table. When HTMX hadn't finished loading, Playwright resolved against the hidden table first, finding View buttons that existed in the DOM but were not interactable.
- **Scope:** Any test interacting with agent table rows was vulnerable. The flake surfaced most visibly in the modal data assertion test.

## Reproduction (before fix)
1. Run `test_view_modal_different_agents_show_different_data` repeatedly
2. Observe intermittent timeouts when Playwright targets the hidden legacy table
- **Expected:** Test consistently finds and clicks actionable View buttons
- **Actual:** Non-deterministic timeouts due to selector ambiguity between two matching tables

## Fix / Changes
- Removed the 159-line `<!-- Old table for reference (hidden) -->` block from `admin.html` — dead HTML with no JS references, no feature flags, and no code paths to unhide it
- Changed `agents_table` and `agents_table_body` selectors to target `#agents-table` and `#agents-table-body` directly by ID, matching the pattern used by all other page objects (tools, servers, gateways, prompts, resources)
- Added `wait_for_selector("#agents-table-body", state="attached")` in `wait_for_agents_panel_loaded()` to ensure the HTMX partial has loaded before tests proceed (using `attached` instead of `visible` because an empty `<tbody>` has zero height)
- Files changed: `mcpgateway/templates/admin.html`, `tests/playwright/pages/agents_page.py`

## Testing / Verification
**Checks**
- Target test passed 5/5 consecutive runs with no flakes against a live gateway
- View/Edit/Test modals, search, clear, tab navigation, and HTMX table reload all verified manually
- `make lint` and `make test` pass; coverage unaffected (test-only + template change)

## Review notes
- **Feedback:** Maintainer confirmed root cause analysis was correct — the hidden legacy table with no references was confirmed dead code. Rebased onto main and resolved one conflict in `agents_page.py`, choosing `self.page.locator("#agents-table")` to match the consistent pattern across all page objects.
