# IBM/mcp-context-forge — PR #3100 — Fixed broken Fetch Tools button caused by tojson quote corruption

**Repo:** [IBM/mcp-context-forge](https://github.com/IBM/mcp-context-forge)
**PR:** [#3100](https://github.com/IBM/mcp-context-forge/pull/3100)
**Issue:** [#3082](https://github.com/IBM/mcp-context-forge/issues/3082)
**Status:** confirmed
**Area:** bugfix
**Stack:** HTML, Jinja2, JavaScript, HTMX
**Impact:** The "Fetch Tools" button on the Gateways page now works correctly in HTMX partials, matching the behavior of the initial page load.

---

## Context
MCP Context Forge's admin Gateways page includes a "Fetch Tools" button for each gateway. The page renders via two paths: initial load from `admin.html` and partial updates from `gateways_partial.html` via HTMX.

## Problem
- **Symptom:** Clicking the "Fetch Tools" button in the HTMX partial view did nothing — the button was broken.
- **Root cause:** The `gateways_partial.html` template used Jinja2's `|tojson` filter to render gateway parameters into the `onclick` attribute. `tojson` outputs double-quoted JSON strings, but the `onclick` attribute itself was wrapped in double quotes, corrupting the HTML parsing.
- **Scope:** Only the HTMX partial (`gateways_partial.html`) was affected. The same button worked correctly in `admin.html` (initial page load), which already used single-quoted JS literals.

## Reproduction (before fix)
1. Navigate to admin Gateways page
2. Trigger an HTMX partial refresh (e.g., via pagination or filtering)
3. Click "Fetch Tools" on any gateway
4. Nothing happens
- **Expected:** Tools are fetched for the selected gateway
- **Actual:** Button is non-functional due to corrupted `onclick` attribute

## Fix / Changes
- Single-line change in `gateways_partial.html:41`: replaced `|tojson` filter with single-quoted JS literals, matching the working pattern already used in `admin.html`
- The fix aligns the partial template to the same quoting convention as the initial page load template

## Testing / Verification
**Checks**
- `make lint` passes
- `make test` passes
- `make coverage` >= 80%

## Review notes
- **Reviewers requested:** crivetimihai (code owner) and marekdano
- No explicit feedback comments visible yet
