# IBM/mcp-context-forge — PR #3017 — Persisted admin table filters across HTMX pagination and partial refresh

**Repo:** [IBM/mcp-context-forge](https://github.com/IBM/mcp-context-forge)
**PR:** [#3017](https://github.com/IBM/mcp-context-forge/pull/3017)
**Issue:** [#2992](https://github.com/IBM/mcp-context-forge/issues/2992)
**Status:** confirmed
**Area:** feature / UI
**Stack:** HTML, JavaScript, HTMX, Jinja2
**Impact:** Admins no longer lose their search text, tag filters, or "show inactive" toggles when paginating through any of the 6 admin tables.

---

## Context
MCP Context Forge's admin dashboard has 6 paginated tables (servers, tools, resources, prompts, gateways, a2a-agents). Each supports search, tag filtering, and "show inactive" toggles. The tables use HTMX for partial page updates.

## Problem
- **Symptom:** Applying a search filter or tag, then clicking to page 2, discarded the active filters and showed unfiltered results.
- **Root cause:** The `pagination_controls.html` `loadPage()` function constructed HTMX request URLs using static Jinja2 template values baked in at initial render. When pagination controls rendered before a search, clicking subsequent pages used the stale (empty) values, discarding active filters.
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

## Testing / Verification
**Checks**
- JS tests 583/584 pass
- Unit tests 12,349/12,355 pass (pre-existing failures in both)

## Review notes
- **marekdano** (Collaborator, approved Feb 18): Praised the clean separation using URL as single source of truth; confirmed manual testing verified functionality.
- **crivetimihai** (Member, approved Feb 19): Called it a "good UX improvement," acknowledging that "filter status indicator and search/tag rehydration after HTMX swaps solve a real usability issue."
- **crivetimihai** (Feb 21): Thanked contributor, characterizing the work as preventing "frustrating UX where filters reset on page change."
- **Labels:** enhancement, SHOULD (P2), ui
- **Milestone:** Release 1.0.0-GA
