# marketcalls/openalgo — PR #900 — Added toast and inline error UI for Search API failures

**Repo:** [marketcalls/openalgo](https://github.com/marketcalls/openalgo)
**PR:** [#900](https://github.com/marketcalls/openalgo/pull/900)
**Status:** merged
**Area:** UX / error handling
**Stack:** React, TypeScript
**Impact:** Users now see clear error feedback when a symbol search fails, instead of a misleading "no results found" message.

---

## Context
OpenAlgo is an open-source algorithmic trading platform. The Search page (`frontend/src/pages/Search.tsx`) lets users look up trading symbols via an API call. This is a core workflow — users search before placing orders or configuring strategies.

## Problem
- **Symptom:** When the search API failed (network error, server error, expired session), users saw "No results found" with no indication that something went wrong.
- **Root cause:** The `catch` block only called `console.error()` and set results to an empty array — no user-facing feedback existed.
- **Scope:** Only the Search component was affected. The failure was silent for both HTTP errors (4xx/5xx) and network-level failures.

## Reproduction (before fix)
1. Open the Search page
2. Trigger an API failure (e.g., disconnect network, or let the session expire)
3. Enter a symbol and submit the search
- **Expected:** An error message indicating the search failed
- **Actual:** "No results found" displayed, indistinguishable from a valid empty result

## Fix / Changes
- **`frontend/src/pages/Search.tsx`** (+11 −1):
  - Added an `error` state variable via `useState`
  - Reset error state at the beginning of each fetch operation
  - Added toast notifications triggered on both HTTP errors and network failures
  - Replaced the "No results found" fallback with a red inline error message when the error state is set
- **Follow-up commit** (after reviewer feedback):
  - Added `'system'` category to toast calls
  - Differentiated 401/403 errors ("session expired") from 500 errors ("server issue")
  - Renamed variables to prevent shadowing

## Testing / Verification
**Checks**
- Simulated network failure — toast appears and inline error message displays
- Simulated HTTP 500 — toast shows "server issue" message
- Simulated HTTP 401 — toast shows "session expired" message
- Successful search still displays results normally
- Error state clears when a new search is initiated

## Review notes
- **Feedback:** Maintainer approved the core approach but requested two refinements: (1) add a toast category (`'system'`) and (2) differentiate HTTP status codes (401/403 vs 500) for more specific error messages.
- **Iteration:** Addressed both points in a follow-up commit — added the category, implemented status code branching, and cleaned up variable shadowing.