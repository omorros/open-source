# IBM/mcp-context-forge — PR #2959 — Fixed external plugin loading by wiring plugin_dirs config into sys.path

**Repo:** [IBM/mcp-context-forge](https://github.com/IBM/mcp-context-forge)
**PR:** [#2959](https://github.com/IBM/mcp-context-forge/pull/2959)
**Issue:** [#2935](https://github.com/IBM/mcp-context-forge/issues/2935)
**Status:** confirmed
**Area:** bugfix / plugins
**Stack:** Python
**Impact:** External plugins in custom directories now load correctly via the `plugin_dirs` configuration field.

---

## Context
MCP Context Forge supports a plugin system with a `PluginManager` that discovers and loads plugins. The `plugins/config.yaml` file includes a `plugin_dirs` field for specifying custom directories, but this field was never actually used by the plugin manager.

## Problem
- **Symptom:** External plugins placed in directories listed under `plugin_dirs` in `config.yaml` failed to load with `ImportError`.
- **Root cause:** `PluginManager.initialize()` never added configured `plugin_dirs` to `sys.path` before calling `importlib.import_module()`. The config field existed but was completely ignored.
- **Scope:** Only external plugins in custom directories were affected. Built-in plugins in the default directory loaded normally.

## Reproduction (before fix)
1. Add an external plugin directory to `plugin_dirs` in `plugins/config.yaml`
2. Place a valid plugin in that directory
3. Start the gateway
4. Plugin fails to load with `ImportError`
- **Expected:** The plugin loads successfully from the configured directory
- **Actual:** `ImportError` because the directory was never added to `sys.path`

## Fix / Changes
- Added `plugin_dirs` entries to `sys.path` during `PluginManager.initialize()` with `os.path.isdir` validation
- Added cleanup in both `shutdown()` and `reset()` to remove added paths and prevent path leakage
- Only `mcpgateway/plugins/framework/manager.py` was changed (production code)

## Testing / Verification
**Checks**
- `pytest tests/unit/mcpgateway/plugins/framework/test_manager.py -v` — 13/13 passed

## Review notes
- **Feedback:** Two maintainers approved — described the fix as "clean and minimal" and praised the test quality for covering both adding and removing plugins.
