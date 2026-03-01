# IBM/mcp-context-forge — PR #3185 — Widened ServerCapabilities fields to Dict[str, Any] to match MCP SDK

**Repo:** [IBM/mcp-context-forge](https://github.com/IBM/mcp-context-forge)
**PR:** [#3185](https://github.com/IBM/mcp-context-forge/pull/3185)
**Issue:** [#3063](https://github.com/IBM/mcp-context-forge/issues/3063)
**Status:** merged
**Area:** bugfix / backend
**Stack:** Python, Pydantic
**Impact:** Plugin-enabled MCP servers no longer fail validation when returning non-boolean capability values, aligning the gateway with the MCP SDK's permissive approach.

---

## Context
MCP Context Forge validates gateway server capabilities against a Pydantic model (`ServerCapabilities`) during registration. The MCP SDK uses `extra: "allow"` on `ToolsCapability`, permitting arbitrary extra fields. However, the gateway schema was more restrictive.

## Problem
- **Symptom:** `PydanticGateway.model_validate()` raised validation errors when plugin-enabled servers returned non-boolean values in their capability responses.
- **Root cause:** The `ServerCapabilities` model typed `prompts`, `resources`, and `tools` fields as `Dict[str, bool]`, which rejected non-boolean values. Meanwhile, `logging`, `completions`, and `experimental` already used `Dict[str, Any]`.
- **Scope:** Only the `prompts`, `resources`, and `tools` capability fields were affected. The three other capability fields (`logging`, `completions`, `experimental`) already used `Dict[str, Any]`.

## Reproduction (before fix)
1. Register a gateway with a plugin-enabled MCP server that returns non-boolean capability values
2. `PydanticGateway.model_validate()` raises a validation error
- **Expected:** Registration succeeds
- **Actual:** Validation error

## Fix / Changes
- Single file change in `mcpgateway/common/models.py`
- Changed `prompts`, `resources`, and `tools` fields from `Dict[str, bool]` to `Dict[str, Any]` to align with the MCP SDK's permissive approach and the pattern already used by the other three capability fields
- Extended the same treatment to `ClientCapabilities.roots` for consistency
- Added test coverage verifying that strings, integers, and nested dictionaries are accepted

## Testing / Verification
**Checks**
- `make lint` — passed
- `make test` — all 1,055 tests passed with zero failures

## Review notes
- **Feedback:** Maintainer recreated this PR from my original [#3075](https://github.com/IBM/mcp-context-forge/pull/3075) during repository maintenance, crediting me as co-author. Merged Feb 26, 2026.
