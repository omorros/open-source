# IBM/mcp-context-forge — PR #3371 — Fixed forwarded RPC non-2xx responses being masked as success

**Repo:** [IBM/mcp-context-forge](https://github.com/IBM/mcp-context-forge)
**PR:** [#3371](https://github.com/IBM/mcp-context-forge/pull/3371)
**Issue:** [#3365](https://github.com/IBM/mcp-context-forge/issues/3365)
**Status:** merged
**Area:** bugfix / Backend
**Stack:** Python, FastAPI, JSON-RPC
**Impact:** Non-2xx HTTP responses from forwarded RPC calls now propagate actual error details instead of returning a generic `{"result": {}}`, restoring proper error visibility across the multi-worker MCP gateway.

---

## Context
MCP Context Forge's gateway uses session affinity to route MCP requests to the correct worker. When a request lands on the wrong worker, it is forwarded via `_execute_forwarded_request()` to the session-owning worker.

## Problem
- **Symptom:** Forwarded RPC calls that returned non-2xx status codes (e.g., 404, 500) appeared to succeed with an empty `{"result": {}}` response.
- **Root cause:** `_execute_forwarded_request()` caught non-2xx responses and returned a generic `{"result": {}}` regardless of the actual error body, masking the real error details.
- **Scope:** All forwarded RPC requests through the multi-worker session affinity mechanism were affected. Direct (non-forwarded) requests worked correctly.

## Reproduction (before fix)
1. Deploy MCP Context Forge with multiple workers
2. Send an RPC request that gets forwarded to another worker
3. Have the target worker return a non-2xx response (e.g., tool not found)
4. Observe the caller receives `{"result": {}}` instead of the actual error
- **Expected:** JSON-RPC error with actual error details propagated
- **Actual:** Generic `{"result": {}}` masking the real error

## Fix / Changes
- Modified error handling in `mcpgateway/services/mcp_session_pool.py` to inspect response bodies before returning generic errors on non-2xx status codes
- JSON-RPC errors are propagated directly when present in the response
- Non-JSON-RPC bodies (e.g., FastAPI `{"detail": "..."}`) are mapped to structured JSON-RPC errors
- Falls back to `response.text[:200]` for unparseable content
- Added guards against `None` or non-dict JSON parsing results

## Testing / Verification
**Checks**
- 246 session pool unit/e2e tests pass
- 65 affinity/forwarding-specific tests pass
- 67 Playwright tests passed
- 0 worker errors in Docker deployment
- Verified on 3-worker cluster with RPC, SSE, Streamable HTTP, and tool calls
- 6 new test cases covering: JSON-RPC error propagation, non-JSON-RPC detail mapping, unparseable bodies, and null body edge cases

## Review notes
- **Feedback:** Initial reviewer flagged missing `status_code` attributes in test fixtures and requested additional test coverage for new error mapping paths.
- **Iteration:** Applied test fixture corrections, added 6 comprehensive test cases. Maintainer rebased onto main, removed unrelated refactoring that conflicted with earlier commit (background task ownership improvements), kept only the scoped bug fix, and added robustness guards for type safety.
