# IBM/mcp-context-forge — PR #3021 — Surfaced toast notifications for user deletion errors

**Repo:** [IBM/mcp-context-forge](https://github.com/IBM/mcp-context-forge)
**PR:** [#3021](https://github.com/IBM/mcp-context-forge/pull/3021)
**Issue:** [#3015](https://github.com/IBM/mcp-context-forge/issues/3015)
**Status:** confirmed
**Area:** bugfix / UI
**Stack:** HTML, JavaScript, HTMX
**Impact:** Admins now see clear error messages when user deletion fails (self-deletion, last admin, DB error) instead of the error being silently discarded.

---

## Context
MCP Context Forge's admin dashboard allows administrators to delete users. The deletion uses HTMX for the request, with a global `htmx:beforeSwap` handler managing response processing.

## Problem
- **Symptom:** When deleting a user returned a 4xx error (self-deletion, last admin, or DB error), no feedback was shown to the administrator. The operation appeared to succeed silently.
- **Root cause:** The global `htmx:beforeSwap` handler prevented swaps for user delete responses on error status codes. The error HTML response was received but silently discarded — never displayed to the user.
- **Scope:** Only user deletion errors were affected. Successful deletions worked correctly.

## Reproduction (before fix)
1. Navigate to admin Users page
2. Attempt to delete the currently logged-in admin user
3. No error message appears — deletion appears to fail silently
- **Expected:** Error toast explaining why deletion failed
- **Actual:** No feedback

## Fix / Changes
- Attached `hx-on::after-request` handlers to delete buttons in both the Jinja template (`users_partial.html`) and the Python-generated card (`admin.py`)
- On non-2xx responses, the handler parses the error HTML, extracts plain text, and calls the existing `showErrorMessage()` function to display a toast
- Extracted inline handler into a named JS function in `admin.js` after reviewer feedback
- Files changed: `admin.py` (line 6357), `users_partial.html` (line 116), `admin.js` (new handler function)

## Testing / Verification
**Checks**
- Verified toast appears for self-deletion attempt
- Verified toast appears for last-admin deletion attempt
- Verified toast appears for simulated DB errors
- Confirmed successful deletions still work normally

## Review notes
- **Feedback:** Both reviewers approved — one suggested extracting the inline handler into a named function (implemented). Maintainer called it a "clean" and "important UX fix."
