# IBM/mcp-context-forge — PR #3402 — Preserved visibility selection when editing entities

**Repo:** [IBM/mcp-context-forge](https://github.com/IBM/mcp-context-forge)
**PR:** [#3402](https://github.com/IBM/mcp-context-forge/pull/3402)
**Issue:** [#3391](https://github.com/IBM/mcp-context-forge/issues/3391)
**Status:** merged
**Area:** bugfix / UI
**Stack:** JavaScript, FormData API
**Impact:** Editing any entity (server, tool, gateway, resource, prompt) no longer silently resets its visibility to "private".

---

## Context
MCP Context Forge's admin UI has create and edit forms for six entity types (servers, tools, gateways, resources, prompts, A2A agents). Each entity carries a `visibility` field ("public" or "private"). The create handlers already appended the selected visibility to `FormData` before submission, but the edit handlers did not.

## Problem
- **Symptom:** Every time an admin edited an entity and saved, the visibility silently reverted to "private" regardless of the user's selection.
- **Root cause:** The five edit form submit handlers (`handleEditServerFormSubmit`, `handleEditToolFormSubmit`, `handleEditGatewayFormSubmit`, `handleEditResFormSubmit`, `handleEditPromptFormSubmit`) did not append the `visibility` field to `FormData`. The backend defaulted to "private" when the field was absent.
- **Scope:** All entity edit forms were affected. Create forms worked correctly because they already included the `visibility` field.

## Reproduction (before fix)
1. Navigate to any entity page (e.g., `http://localhost:8080/admin/#servers`)
2. Create an entity with visibility set to "public"
3. Open the edit form for that entity
4. Change any field and click Save (without touching visibility)
5. Reopen the entity
- **Expected:** Visibility remains "public"
- **Actual:** Visibility has been silently reset to "private"

## Fix / Changes
- Added `formData.set("visibility", ...)` to all five edit form submit handlers in `admin.js`, matching the pattern already used in create handlers
- Used `set()` instead of `append()` to prevent duplicate visibility entries in `FormData`
- Added `tests/js/admin-edit-visibility.test.js` with unit tests validating that each edit handler includes exactly one visibility entry

## Testing / Verification
**Checks**
- New unit tests pass for all six edit handlers, confirming visibility is included in `FormData`
- `make lint` passes
- `make test` passes
- Maintainer independently validated with curl API tests and Playwright E2E tests

## Review notes
- **Feedback:** Maintainer confirmed: "Without this, every edit silently overwrites visibility to 'private'. Minimal and correct change."
- **Iteration:** Maintainer replaced `append()` with `set()` across all handlers to prevent duplicates, identified a gap in the A2A agent edit handler, and strengthened test coverage across all six edit handlers.
