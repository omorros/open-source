# IBM/mcp-context-forge — PR #2950 — Standardized loading indicators across admin pages

**Repo:** [IBM/mcp-context-forge](https://github.com/IBM/mcp-context-forge)
**PR:** [#2950](https://github.com/IBM/mcp-context-forge/pull/2950)
**Issue:** [#2946](https://github.com/IBM/mcp-context-forge/issues/2946)
**Status:** merged
**Area:** bugfix / UI
**Stack:** HTML, CSS, Tailwind
**Impact:** All admin pages now display the same indigo SVG spinner during loading, giving users a consistent and polished experience.

---

## Context
MCP Context Forge is an IBM project with an admin dashboard for managing gateways, agents, teams, and MCP servers. Several pages fetch data asynchronously and show a loading indicator while waiting for the API response. The Gateways and Tools pages already used a standardized indigo SVG spinner, but other pages used different patterns.

## Problem
- **Symptom:** Loading indicators varied across admin pages — some used a CSS border-based spinner, others showed plain "Loading..." text or an animate-pulse message with a note, and spacing was inconsistent.
- **Root cause:** Each page section was implemented independently with its own loading markup instead of following the common pattern established on the Gateways and Tools pages.
- **Scope:** MCP Registry, Teams, API Tokens, and System Logs pages were affected. Gateways and Tools already used the correct pattern.

## Reproduction (before fix)
1. Navigate to `http://localhost:8080/admin`
2. Hard-refresh the MCP Registry, Teams, API Tokens, and System Logs pages
3. Observe the different loading indicator styles on each page
- **Expected:** All pages display the same loading spinner style used on Gateways and Tools
- **Actual:** Each page showed a different loading pattern (CSS spinner, plain text, animate-pulse text with note message, inconsistent spacing)

### Before
![MCP Registry page showing a non-standard CSS border spinner instead of the indigo SVG spinner](../images/ibm-mcp-context-forge-pr-2950/before-inconsistent-loading.png)

### After
![Gateways page showing the standardized indigo SVG loading spinner](../images/ibm-mcp-context-forge-pr-2950/after-standardized-loading.png)

## Fix / Changes
- **MCP Registry:** Replaced the CSS border-based spinner with the standard indigo SVG spinner (`flex items-center justify-center p-4`, `h-8 w-8` SVG, `ml-2`, `text-indigo-600 dark:text-indigo-400`)
- **Teams:** Replaced the animate-pulse text and note message with the standard SVG spinner pattern
- **API Tokens:** Replaced the spinner and corrected spacing from `p-8`/`ml-3` to `p-4`/`ml-2` to match the standard layout
- **System Logs:** Replaced the plain "Loading..." text with the standard SVG spinner

## Testing / Verification
**Checks**
- Confirmed all four pages now display the identical indigo SVG spinner during loading
- Verified the spinner matches the existing pattern on Gateways and Tools pages
- Tested dark mode to ensure `text-indigo-400` applies correctly
- No Python code changes — lint, unit tests, and coverage unaffected

## Review notes
- **Feedback:** Reviewer approved the cleanup and additionally applied the same standardization to the Users section loading indicator for full consistency.
- **Iteration:** Rebased onto main with no conflicts before merge.
