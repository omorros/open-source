# IBM/mcp-context-forge — PR #3785 — Added forbidden-pattern check to ToolUpdate.validate_description

**Repo:** [IBM/mcp-context-forge](https://github.com/IBM/mcp-context-forge)
**PR:** [#3785](https://github.com/IBM/mcp-context-forge/pull/3785)
**Issue:** [#3773](https://github.com/IBM/mcp-context-forge/issues/3773)
**Status:** merged
**Area:** bugfix / Security
**Stack:** Python, Pydantic
**Impact:** Tool descriptions can no longer bypass shell-metacharacter validation by being injected post-creation via the update endpoint.

---

## Context
MCP Context Forge validates tool descriptions on creation via `ToolCreate.validate_description`, rejecting shell metacharacters (`&&`, `;`, `||`, `$(`, `|`, `>`, `<`). The update path (`ToolUpdate`) lacked the same check.

## Problem
- **Symptom:** A tool created with a clean description could later be updated with shell metacharacters that `ToolCreate` would have rejected.
- **Root cause:** `ToolUpdate.validate_description` was missing the forbidden-pattern check that `ToolCreate.validate_description` enforced.
- **Scope:** Only tool description updates were affected. Creation-time validation was correct.

## Reproduction (before fix)
1. Create a tool with a valid description
2. Update the tool description to include `; rm -rf /` or `$(malicious)`
3. Update succeeds without rejection
- **Expected:** Validation error rejecting the forbidden pattern
- **Actual:** Update accepted the dangerous description

## Fix / Changes
- Added forbidden-pattern validation to `ToolUpdate.validate_description`, mirroring the logic in `ToolCreate`
- Aligned with `VALIDATION_STRICT` setting: in strict mode, forbidden patterns raise a validation error; in non-strict mode, a warning is logged instead (matching `ToolCreate` behavior which allows Markdown syntax like `> blockquote`)
- Added parametrized tests rejecting seven forbidden patterns on `ToolUpdate`
- Expanded test coverage to mirror `TestToolCreateDescriptionValidationStrict`, including strict-mode rejection, non-strict warnings, logging verification, and boundary conditions (empty strings, patterns at start/end, maximum length descriptions)

## Testing / Verification
**Checks**
- `make lint` passes
- `make test` passes — all 75 tests in the validation test file passing
- `make coverage` >= 80%

## Review notes
- **Feedback:** Reviewer confirmed "forbidden-pattern check correctly mirrors ToolCreate with VALIDATION_STRICT support. Tests comprehensive."
- **Iteration:** Added `VALIDATION_STRICT` alignment after review — original code hardcoded exceptions, breaking symmetry with `ToolCreate`.
