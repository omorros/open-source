# IBM/mcp-context-forge — PR #3708 — Removed unused PaginationParams, ObservabilityQueryParams, and PerformanceHistoryParams schemas

**Repo:** [IBM/mcp-context-forge](https://github.com/IBM/mcp-context-forge)
**PR:** [#3708](https://github.com/IBM/mcp-context-forge/pull/3708)
**Issue:** [#3706](https://github.com/IBM/mcp-context-forge/issues/3706)
**Status:** merged
**Area:** chore / Cleanup
**Stack:** Python, Pydantic
**Impact:** Removed three dead-code Pydantic schema classes that no endpoint imported, eliminating a latent bug where `PaginationParams` hardcoded `le=500` which would silently conflict with the configurable `PAGINATION_MAX_PAGE_SIZE` setting.

---

## Context
MCP Context Forge's `mcpgateway/schemas.py` contained three Pydantic schema classes — `PaginationParams`, `ObservabilityQueryParams`, and `PerformanceHistoryParams` — that were no longer imported by any endpoint.

## Problem
- **Symptom:** Dead code in `schemas.py` with no live references.
- **Root cause:** The three classes were remnants from earlier implementations that had been replaced. `PaginationParams` in particular hardcoded `le=500`, which would silently conflict with the configurable `PAGINATION_MAX_PAGE_SIZE` setting if ever re-imported (related to bug #3469).
- **Scope:** No runtime impact — purely dead code. But a latent risk if any developer re-imported `PaginationParams`.

## Fix / Changes
- Removed `PaginationParams`, `ObservabilityQueryParams`, and `PerformanceHistoryParams` from `mcpgateway/schemas.py`
- Deleted corresponding `PerformanceHistoryParams` unit tests from `test_performance_schemas.py`
- Cleaned up unused `pytest` import and re-sorted imports

## Testing / Verification
**Checks**
- `make lint` passes
- `make test` passes
- `make coverage` >= 80%
- Verified all three classes had zero live references across the codebase

## Review notes
- **Feedback:** Reviewer confirmed "clean dead code removal" with verification of zero live references. Noted the auto-generated `classes.svg` diagram would self-correct on next regeneration.
