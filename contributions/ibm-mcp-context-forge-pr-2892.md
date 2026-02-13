# IBM/mcp-context-forge — PR #2892 — Fixed duplicate loading spinner on A2A Agents page

**Repo:** [IBM/mcp-context-forge](https://github.com/IBM/mcp-context-forge)
**PR:** [#2892](https://github.com/IBM/mcp-context-forge/pull/2892)
**Status:** merged
**Area:** bugfix / UI
**Stack:** HTML, HTMX
**Impact:** Users no longer see two overlapping spinners when loading or filtering the A2A Agents page.

---

## Context
MCP Context Forge is an IBM project with an admin dashboard built on HTMX. The A2A (Agent-to-Agent) Agents page uses indicator-based loading patterns shared across multiple tabs (servers, virtual servers, tools, agents).

## Problem
- **Symptom:** Two loading spinners appeared simultaneously when refreshing the Agents page or toggling the "Show Inactive" filter.
- **Root cause:** The `#agents-table` div contained an inline placeholder spinner alongside the HTMX-driven `#agents-loading` indicator — both rendered at the same time during loading states.
- **Scope:** Only the Agents tab was affected. The servers, virtual servers, and tools tabs had already been fixed in commit `6b3283e` via PR #2720, but the agents tab was overlooked.

## Reproduction (before fix)
1. Navigate to `http://localhost:8080/admin/#a2a-agents`
2. Refresh the page
3. Observe two spinners stacking on screen
- **Expected:** A single loading spinner
- **Actual:** Two spinners displayed simultaneously

## Fix / Changes
- Removed the 8-line inline placeholder `<div>` spinner from the `#agents-table` section in the HTML template (+0 −8, pure deletion)
- Retained the HTMX `#agents-loading` indicator as the single source of loading feedback
- Aligned the agents tab with the pattern already established in servers, virtual servers, and tools tabs

## Testing / Verification
**Checks**
- Confirmed a single spinner appears on page refresh across all tabs
- Verified correct spinner behavior when toggling the "Show Inactive" filter
- No Python code changes — lint, unit tests, and coverage unaffected

## Risks / Trade-offs
- **Risk:** Minimal — pure template deletion, no logic changes.
- **Mitigation:** The same pattern was already applied to three other tabs without issues. Change is scoped to a single HTML block.

## Review notes
- **Feedback:** Reviewer asked whether the same duplicate-spinner issue existed on other tabs.
- **Iteration:** Clarified that servers, virtual servers, and tools were already fixed in PR #2720 (commit `6b3283e`), and only the agents tab still had the duplicate.

## Takeaways
- Small UI inconsistencies can slip through when a fix is applied to most tabs but not all — worth doing a sweep across sibling components.
