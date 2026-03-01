# IBM/mcp-context-forge — PR #3209 — Prevented browser crashes on large teams by switching to search-only non-member loading

**Repo:** [IBM/mcp-context-forge](https://github.com/IBM/mcp-context-forge)
**PR:** [#3209](https://github.com/IBM/mcp-context-forge/pull/3209)
**Issue:** [#3085](https://github.com/IBM/mcp-context-forge/issues/3085)
**Status:** confirmed
**Area:** bugfix / UI
**Stack:** Python, JavaScript, HTML, HTMX
**Impact:** The Manage Members modal no longer crashes browsers when the system has 20K+ users, by replacing eager infinite-scroll loading with search-first server-side filtering.

---

## Context
MCP Context Forge's Teams page includes a "Manage Members" modal for adding and removing team members. The modal has two sections: current members and non-members (users available to add).

## Problem
- **Symptom:** Opening the Manage Members modal on a system with 20K+ users caused browsers to crash or become unresponsive.
- **Root cause:** The non-members section loaded all non-members eagerly via infinite scroll, accumulating DOM nodes without limit. With large user pools, this overwhelmed the browser.
- **Scope:** Only the Manage Members modal was affected. The issue became critical at scale (20K+ users) but degraded performance starting around 1K+ users.

## Reproduction (before fix)
1. Set up a system with 20K+ users
2. Navigate to admin Teams page
3. Open "Manage Members" for any team
4. Browser crashes or becomes unresponsive
- **Expected:** Modal opens responsively regardless of user count
- **Actual:** Browser crash or severe lag due to unbounded DOM node accumulation

## Fix / Changes
- Non-members section no longer auto-loads on modal open — users must search by name/email first, with results capped at 50
- Current members section gained its own search bar with server-side filtering via new `search` parameter on `get_team_members()`
- Replaced monolithic client-side search function `serverSideUserSearch` with two focused server-side functions: `serverSideMemberSearch` and `serverSideNonMemberSearch`
- Files changed: `admin.py` (split unified search into two per-section inputs), `team_management_service.py` (added `search` parameter and `_escape_like()`), `admin.js` (replaced unified search with two focused functions), `test_admin.py` (updated 13 tests, added `test_admin_team_non_members_partial_html_empty_search`)

## Testing / Verification
**Checks**
- `make lint` passes
- `make test` passes
- `make coverage` >= 80%

## Review notes
- **Feedback:** Maintainer recreated this PR from my original [#3103](https://github.com/IBM/mcp-context-forge/pull/3103) during repository maintenance, crediting me as co-author.
