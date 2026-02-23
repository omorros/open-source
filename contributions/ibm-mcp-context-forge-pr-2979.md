# IBM/mcp-context-forge — PR #2979 — Added json_prune plugin to strip unnecessary fields from tool outputs

**Repo:** [IBM/mcp-context-forge](https://github.com/IBM/mcp-context-forge)
**PR:** [#2979](https://github.com/IBM/mcp-context-forge/pull/2979)
**Issue:** [#2971](https://github.com/IBM/mcp-context-forge/issues/2971)
**Status:** confirmed
**Area:** feature / plugins
**Stack:** Python, orjson
**Impact:** LLM tool calls no longer waste tokens on extraneous API response fields (thumbnails, parsed URLs, engine metadata) thanks to whitelist-based pruning.

---

## Context
MCP Context Forge's plugin system allows post-processing of tool invocation results. REST APIs frequently return large JSON payloads with fields irrelevant to LLM reasoning, consuming tokens and potentially misleading models.

## Problem
- **Symptom:** Tool outputs from REST APIs included unnecessary fields (thumbnails, engines, parsed URLs) that inflated token usage and could mislead LLMs.
- **Root cause:** No mechanism existed to filter tool output fields before they reached the LLM. The full API response was passed through unchanged.
- **Scope:** All tools returning JSON responses. Plugin operates as a `tool_post_invoke` hook.

## Fix / Changes
- Created `json_prune` plugin with whitelist-only strategy using dot-notation paths (e.g., `results.title`)
- Implementation based on proposal by @alex9434 in issue #2971, adapted to project conventions
- New files: `plugins/json_prune/__init__.py`, `json_prune.py`, `plugin-manifest.yaml`, `README.md`
- Tests: 21 unit tests in `tests/unit/.../test_json_prune.py`
- Modified: `plugins/config.yaml` (registered at priority 149, disabled by default)
- Design: immutable payload handling consistent with `json_repair` pattern, no additional dependencies (uses existing `orjson`)

## Testing / Verification
**Checks**
- 21 unit tests covering recursive pruning, both result formats, and safety defaults — all pass
- Elegant `_get_pruned()` function handles nested field selection via dot-notation paths
- Support for both result formats validated

## Review notes
- **crivetimihai** (Feb 19): "Well-structured plugin with clean separation between recursive pruning algorithm and plugin dispatch logic." Noted strengths: elegant `_get_pruned()` function, comprehensive 21-test coverage, support for both result formats, appropriate safety defaults.
- **crivetimihai** (Feb 21): "Thanks @omorros — the json_prune plugin is a useful addition for reducing tool output payload sizes. Clean implementation."
- **Milestone:** Release 1.1.0
