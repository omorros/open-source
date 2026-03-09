# IBM/mcp-context-forge — PR #3544 — Fixed decode_auth crash on masked auth_value in admin_test_gateway

**Repo:** [IBM/mcp-context-forge](https://github.com/IBM/mcp-context-forge)
**PR:** [#3544](https://github.com/IBM/mcp-context-forge/pull/3544)
**Issue:** [#3539](https://github.com/IBM/mcp-context-forge/issues/3539)
**Status:** merged
**Area:** bugfix / backend
**Stack:** Python, SQLAlchemy, AES-GCM
**Impact:** admin_test_gateway no longer crashes when testing any gateway (with or without auth).

---

## Context
MCP Context Forge's admin API exposes an `admin_test_gateway` endpoint that lets administrators verify gateway connectivity. The endpoint retrieves the gateway's stored credentials, decrypts them via `decode_auth()`, and injects them into the test request headers.

## Problem
- **Symptom:** Calling `admin_test_gateway` crashed with `ValueError: Nonce must be between 8 and 128 bytes` for any gateway with stored auth credentials.
- **Root cause:** The endpoint used `get_first_gateway_by_url()`, which returns a `GatewayRead.masked()` object where `auth_value` is replaced with `"*****"`. Passing that masked string to `decode_auth()` base64-decoded it to a 3-byte payload, then AES-GCM failed extracting a 12-byte nonce from it.
- **Scope:** Every gateway with auth was affected. Gateways without auth were unaffected but still hit a redundant code path.

## Reproduction (before fix)
1. Create a gateway with basic or bearer auth via the admin UI
2. Call the `admin_test_gateway` endpoint for that gateway
3. Observe `ValueError: Nonce must be between 8 and 128 bytes` in server logs
- **Expected:** Gateway connectivity test succeeds or returns a meaningful HTTP error
- **Actual:** Server crashes with a cryptography error from AES-GCM nonce extraction

## Fix / Changes
- Replaced `gateway_service.get_first_gateway_by_url()` with a direct `select(DbGateway)` query to retrieve the unmasked `auth_value` from the database
- Replaced the catch-all `else` branch with `elif gateway.auth_type in ("basic", "bearer", "authheaders")` so gateways without auth skip decryption entirely
- Handled both `dict` and `str` `auth_value` formats from the raw DB object

## Testing / Verification
**Commands**
- `make lint` — no new issues
- `make test` — 1001 passed, 4 pre-existing failures (unrelated `TestPaginationSwapStyle`)
- `make coverage` — 97% on `mcpgateway/admin.py`

**Checks**
- Gateway with basic auth: test request succeeds, no crash
- Gateway with bearer auth: test request succeeds, no crash
- Gateway with no auth: test request skips decryption, no crash
- Maintainer added differential coverage tests for all three auth branches plus the enabled-filter regression

## Review notes
- **Feedback:** Maintainer noted the original `get_first_gateway_by_url()` filtered by `DbGateway.enabled` — the direct query initially dropped that filter, risking matches against disabled gateways with stale credentials.
- **Iteration:** Maintainer restored the `enabled` filter in the `where()` clause and added regression tests verifying the compiled SQL includes it. Also added 5 new tests covering dict/str auth_value, auth_type=None, header merge precedence, and the enabled filter.
