# IBM/mcp-context-forge — PR #3647 — Persisted admin table filters across HTMX pagination and partial refresh

**Repo:** [IBM/mcp-context-forge](https://github.com/IBM/mcp-context-forge)
**PR:** [#3647](https://github.com/IBM/mcp-context-forge/pull/3647)
**Issue:** [#2992](https://github.com/IBM/mcp-context-forge/issues/2992)
**Status:** merged
**Area:** feature / UI
**Stack:** HTML, JavaScript, HTMX, Jinja2
**Impact:** Admins no longer lose their search text, tag filters, or "show inactive" toggles when paginating through any of the 6 admin tables.

---

## Context
MCP Context Forge's admin dashboard has 6 paginated tables (servers, tools, resources, prompts, gateways, a2a-agents). Each supports search, tag filtering, and "show inactive" toggles. The tables use HTMX for partial page updates.

## Problem
- **Symptom:** Applying a search filter or tag, then clicking to page 2, discarded the active filters and showed unfiltered results.
- **Root cause:** The `pagination_controls.html` `loadPage()` function constructed HTMX request URLs using static Jinja2 template values baked in at initial render. After a search, clicking subsequent pages used the stale (empty) values, discarding active filters.
- **Scope:** All 6 admin tables were affected. Filters worked correctly within a single page but were lost on any pagination or partial refresh.

## Reproduction (before fix)
1. Navigate to any admin table (e.g., Tools)
2. Enter a search term or select a tag filter
3. Click page 2 in the pagination controls
4. Observe that filters are discarded and unfiltered results appear
- **Expected:** Page 2 shows filtered results matching the active search/tag
- **Actual:** Page 2 shows unfiltered results because the pagination URL used stale template values

## Fix / Changes
- **`pagination_controls.html`:** Modified `loadPage()` to read current `q`/`tags` parameters from the browser URL (the authoritative source updated by `updatePanelSearchStateInUrl`), overriding outdated template values
- **`admin.js`:** Implemented `updateFilterStatus()` function for displaying filtered row counts; added `htmx:afterSettle` listener to restore search inputs from URL state post-content swap
- **`admin.html`:** Inserted `aria-live="polite"` status spans adjacent to all 6 table search interfaces
- Layered strategy prioritizing DOM input values over URL params to handle both standard deployments and restricted iframe contexts

## Testing / Verification
**Checks**
- 27 differential JS tests added (`admin-pagination.test.js`)
- JS tests: 98/98 passing (vitest)
- Unit tests: 12,349/12,355 passing (6 pre-existing failures)
- E2E verification: 0 console errors confirmed via Playwright + Docker deployment

## Review notes
- **Feedback:** Maintainer recreated this PR from my original [#3204](https://github.com/IBM/mcp-context-forge/pull/3204) (itself from [#3017](https://github.com/IBM/mcp-context-forge/pull/3017)) during repository maintenance, crediting me as co-author.
- **Iteration:** During rebasing and hardening I discovered and fixed four bugs: (1) `TypeError` on `evt.detail` — added optional chaining, (2) stale filter on clear — changed logic to trust DOM values when present, (3) fragile pagination text reading — simplified to always show "Filters active", (4) `document.body` null crash — changed to `document.addEventListener` for script-load safety.
- **Approval:** Reviewer praised the "clean, well-architected solution" with proper layering and data flow.
