# IBM/mcp-context-forge — PR #3095 — Added regression test coverage for prompt original_name during federation

**Repo:** [IBM/mcp-context-forge](https://github.com/IBM/mcp-context-forge)
**PR:** [#3095](https://github.com/IBM/mcp-context-forge/pull/3095)
**Issue:** [#3087](https://github.com/IBM/mcp-context-forge/issues/3087)
**Status:** confirmed
**Area:** testing
**Stack:** Python, pytest
**Impact:** The `original_name` field on federated prompts now has explicit regression coverage, preventing a previously fixed NOT NULL constraint violation from recurring.

---

## Context
MCP Context Forge federates prompts from registered gateway servers. Issue #3087 reported a NOT NULL constraint violation on `prompts.original_name` during federation. The bug was already fixed on main — `gateway_service.py` now explicitly sets `original_name`, `custom_name`, and `display_name` during prompt creation. However, test coverage was missing.

## Problem
- **Symptom:** The fix for #3087 existed in production code but had no regression tests. The tool federation test verified `original_name`, but the prompt federation tests did not assert on this field.
- **Root cause:** Missing test assertions — prompt federation tests did not validate `original_name` assignment.
- **Scope:** Only prompt federation test coverage was lacking. Tool federation tests already included `original_name` assertions.

## Fix / Changes
- Added `original_name` assertions to two test paths:
  - `test_register_gateway_creates_new_resources_and_prompts` (initial gateway registration)
  - `TestUpdateOrCreatePrompts::test_new_prompt_created` (prompt refresh/rediscovery)
- Actual test additions total approximately 8 lines of assertions across two test functions
- The bulk of the diff is `isort`/`black` reformatting

## Testing / Verification
**Checks**
- `make lint` — passed
- `make test` — 262 passed, 1 skipped
- `make coverage` >= 80% — passed

## Review notes
- No explicit maintainer comments visible. Code owners requested for review: crivetimihai, kevalmahajan, madhav165.
