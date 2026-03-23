# README Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Redesign the OSS contributions README from a flat table into a portfolio-grade page with highlight cards, compact tables, and curated social proof.

**Architecture:** Replace README.md content entirely. Delete 7 orphaned case study files. No new files created beyond the README rewrite.

**Tech Stack:** Markdown (GitHub-flavored)

**Spec:** `docs/superpowers/specs/2026-03-23-readme-redesign-design.md`

---

### Task 1: Delete orphaned case study files

**Files:**
- Delete: `contributions/ibm-mcp-context-forge-pr-3179.md`
- Delete: `contributions/ibm-mcp-context-forge-pr-3200.md`
- Delete: `contributions/ibm-mcp-context-forge-pr-3201.md`
- Delete: `contributions/ibm-mcp-context-forge-pr-3204.md`
- Delete: `contributions/ibm-mcp-context-forge-pr-3208.md`
- Delete: `contributions/ibm-mcp-context-forge-pr-3209.md`
- Delete: `contributions/ibm-mcp-context-forge-pr-3215.md`

- [ ] **Step 1: Delete the 7 orphaned files**

```bash
cd "C:/Users/Oriol/OneDrive/Escritorio/repos/open source/open-source"
rm contributions/ibm-mcp-context-forge-pr-3179.md
rm contributions/ibm-mcp-context-forge-pr-3200.md
rm contributions/ibm-mcp-context-forge-pr-3201.md
rm contributions/ibm-mcp-context-forge-pr-3204.md
rm contributions/ibm-mcp-context-forge-pr-3208.md
rm contributions/ibm-mcp-context-forge-pr-3209.md
rm contributions/ibm-mcp-context-forge-pr-3215.md
```

- [ ] **Step 2: Verify only the expected 19 case study files + TEMPLATE remain**

```bash
ls contributions/
```

Expected: 19 `.md` files (15 IBM + 4 other) plus `TEMPLATE.md` = 20 files total.

- [ ] **Step 3: Commit**

```bash
git add -A contributions/
git commit -m "chore: remove 7 orphaned case study files from superseded PRs"
```

---

### Task 2: Rewrite README.md

**Files:**
- Modify: `README.md`

**Reference:** Use the spec's concrete highlight card format (Section 2) and the complete table rows (Section 3 and 4).

- [ ] **Step 1: Rewrite README.md with the new structure**

Replace the entire contents of `README.md` with the following:

```markdown
# Open Source Contributions

19 merged PRs across 4 repositories — security, backend, UI, and testing.

## Highlights

### Security validation bypass in tool descriptions

`ToolUpdate` lacked the forbidden-pattern check that `ToolCreate` enforced, allowing shell metacharacters (`&&`, `;`, `$(...)`) to bypass creation-time validation via the update endpoint. Added the missing validation mirroring `VALIDATION_STRICT` behavior, plus parametrized tests covering seven forbidden patterns. Reviewer: *"Tests comprehensive."*

[PR #3785](https://github.com/IBM/mcp-context-forge/pull/3785) | [Case study](./contributions/ibm-mcp-context-forge-pr-3785.md)

### Silent error masking in multi-worker RPC gateway

Forwarded RPC calls returning non-2xx status codes were silently masked as `{"result": {}}`, hiding real errors across the multi-worker session affinity mechanism. Propagated JSON-RPC errors directly, mapped non-JSON-RPC bodies to structured errors, and added null/type guards plus 6 new test cases. Labeled **MUST (P1)** by maintainers.

[PR #3371](https://github.com/IBM/mcp-context-forge/pull/3371) | [Case study](./contributions/ibm-mcp-context-forge-pr-3371.md)

### Browser crashes at scale (20K+ users)

The Manage Members modal loaded all non-members eagerly via infinite scroll, accumulating DOM nodes without limit and crashing browsers at 20K+ users. Replaced with search-first server-side filtering capped at 50 results, added SQL injection prevention via parameterized `ILIKE`, and discovered 3 additional bugs during review (stale cache leakage, encoding mismatch, off-screen state loss). Tagged for **Release 1.0.0** milestone with 2 reviewer approvals.

[PR #3610](https://github.com/IBM/mcp-context-forge/pull/3610) | [Case study](./contributions/ibm-mcp-context-forge-pr-3610.md)

### Flaky E2E test elimination

Playwright tests failed intermittently due to shared mutable login state causing multi-worker routing conflicts, compounded by hard-coded 4-second sleeps instead of readiness checks. Replaced with per-fixture JWT cookie injection and HTMX-aware DOM waits (`wait_for_function("() => !document.querySelector('.htmx-request')")`).

[PR #3210](https://github.com/IBM/mcp-context-forge/pull/3210) | [Case study](./contributions/ibm-mcp-context-forge-pr-3210.md)

## IBM/mcp-context-forge

[MCP Context Forge](https://github.com/IBM/mcp-context-forge) is an IBM open-source MCP gateway, registry, and proxy. Full list of contributions:

| PR | Outcome |
|---|---|
| [#3785](https://github.com/IBM/mcp-context-forge/pull/3785) | Added forbidden-pattern check to `ToolUpdate.validate_description` |
| [#3708](https://github.com/IBM/mcp-context-forge/pull/3708) | Removed unused `PaginationParams`, `ObservabilityQueryParams`, and `PerformanceHistoryParams` schemas |
| [#3647](https://github.com/IBM/mcp-context-forge/pull/3647) | Persisted admin table filters across HTMX pagination and partial refresh |
| [#3610](https://github.com/IBM/mcp-context-forge/pull/3610) | Fixed browser crashes on large teams by switching to search-only non-member loading |
| [#3544](https://github.com/IBM/mcp-context-forge/pull/3544) | Fixed `decode_auth` crash on masked credentials in gateway test endpoint |
| [#3402](https://github.com/IBM/mcp-context-forge/pull/3402) | Preserved visibility selection when editing entities |
| [#3371](https://github.com/IBM/mcp-context-forge/pull/3371) | Fixed forwarded RPC non-2xx responses masked as success |
| [#3370](https://github.com/IBM/mcp-context-forge/pull/3370) | Eliminated Playwright agents modal test flake by removing legacy hidden table |
| [#3210](https://github.com/IBM/mcp-context-forge/pull/3210) | Eliminated Playwright flakiness via JWT-first auth and HTMX-aware waits |
| [#3206](https://github.com/IBM/mcp-context-forge/pull/3206) | Fixed pagination controls vanishing after filtering (Alpine.js reinit) |
| [#3205](https://github.com/IBM/mcp-context-forge/pull/3205) | Surfaced toast notifications for user deletion errors |
| [#3185](https://github.com/IBM/mcp-context-forge/pull/3185) | Widened `ServerCapabilities` fields to `Dict[str, Any]` to match MCP SDK |
| [#2950](https://github.com/IBM/mcp-context-forge/pull/2950) | Standardized loading indicators across 4 admin pages |
| [#2937](https://github.com/IBM/mcp-context-forge/pull/2937) | Prevented modal overflow that hid Save for large teams |
| [#2892](https://github.com/IBM/mcp-context-forge/pull/2892) | Removed duplicate loading spinner on A2A Agents |

## Other contributions

| PR | Outcome |
|---|---|
| [DjangoCRM #388](https://github.com/DjangoCRM/django-crm/pull/388) | Added 19 unit tests for previously untested AJAX view |
| [PINCE #312](https://github.com/korcankaraokcu/PINCE/pull/312) | Fixed arrow-key scrolling and selection sync in hex viewer |
| [OpenAlgo #899](https://github.com/marketcalls/openalgo/pull/899) | Added dynamic aria-labels to inline-edit inputs |
| [OpenAlgo #900](https://github.com/marketcalls/openalgo/pull/900) | Replaced silent failures with toast + inline error UI |
```

- [ ] **Step 2: Verify the README renders correctly**

Open `README.md` and visually confirm:
- Header + subtitle line renders
- 4 highlight cards have `###` headings, prose, and footer links
- IBM table has 15 rows, all PR links point to correct URLs
- Other contributions table has 4 rows
- No broken markdown (unclosed backticks, malformed tables, etc.)

- [ ] **Step 3: Verify all case study links resolve to existing files**

```bash
cd "C:/Users/Oriol/OneDrive/Escritorio/repos/open source/open-source"
for f in contributions/ibm-mcp-context-forge-pr-3785.md contributions/ibm-mcp-context-forge-pr-3371.md contributions/ibm-mcp-context-forge-pr-3610.md contributions/ibm-mcp-context-forge-pr-3210.md; do
  [ -f "$f" ] && echo "OK: $f" || echo "MISSING: $f"
done
```

Expected: All 4 show `OK`.

- [ ] **Step 4: Commit**

```bash
git add README.md
git commit -m "docs: redesign README with highlight cards and compact tables"
```
