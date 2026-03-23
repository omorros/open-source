# README Redesign — Design Spec

## Goal

Redesign the OSS contributions README from a flat table into a portfolio-grade page that works for both quick-scanning recruiters and deep-diving technical interviewers.

## Context

- This repo is linked from the user's main personal README — no personal intro needed here
- 15 merged PRs to IBM/mcp-context-forge + 4 PRs to other repos (19 total)
- Case study files live in `/contributions/` and must be preserved
- 7 orphaned case study files need deletion (#3179, #3200, #3201, #3204, #3208, #3209, #3215)
- `contributions/TEMPLATE.md` is kept — it's a working template for future contributions, not an orphan

## Files to delete

Orphaned case study files not referenced in the README (superseded PRs that were recreated by the maintainer):

- `contributions/ibm-mcp-context-forge-pr-3179.md`
- `contributions/ibm-mcp-context-forge-pr-3200.md`
- `contributions/ibm-mcp-context-forge-pr-3201.md`
- `contributions/ibm-mcp-context-forge-pr-3204.md`
- `contributions/ibm-mcp-context-forge-pr-3208.md`
- `contributions/ibm-mcp-context-forge-pr-3209.md`
- `contributions/ibm-mcp-context-forge-pr-3215.md`

## Structure

### Section 1: Header

```markdown
# Open Source Contributions

19 merged PRs across 4 repositories — security, backend, UI, and testing.
```

One line. Sets scale immediately.

### Section 2: Highlights (4 PRs)

Each highlight uses an `###` heading + 2-3 lines of prose + a footer line linking to the PR and case study. Social proof (P1 labels, reviewer quotes, GA milestone) is woven into the prose where it adds credibility.

**Concrete markdown format for each highlight:**

```markdown
### Security validation bypass in tool descriptions

`ToolUpdate` lacked the forbidden-pattern check that `ToolCreate` enforced, allowing shell metacharacters (`&&`, `;`, `$(...)`) to bypass creation-time validation via the update endpoint. Added the missing validation mirroring `VALIDATION_STRICT` behavior, plus parametrized tests covering seven forbidden patterns. Reviewer: *"Tests comprehensive."*

[PR #3785](https://github.com/IBM/mcp-context-forge/pull/3785) | [Case study](./contributions/ibm-mcp-context-forge-pr-3785.md)
```

**Selected highlights (in order):**

1. **PR #3785 — Security validation bypass in tool descriptions**
   - ToolUpdate lacked the forbidden-pattern check that ToolCreate enforced, allowing shell metacharacters to bypass creation-time validation via updates
   - Added validation + parametrized tests; reviewer confirmed "Tests comprehensive"
   - Merged Mar 22, 2026

2. **PR #3371 — Silent error masking in multi-worker RPC gateway**
   - Forwarded RPC calls returning non-2xx were masked as `{"result": {}}`, hiding real errors across the multi-worker session affinity mechanism
   - Propagated JSON-RPC errors, mapped non-JSON-RPC bodies, added null guards + 6 test cases
   - Merged Mar 22, 2026. Labeled MUST (P1) by maintainers

3. **PR #3610 — Browser crashes at scale (20K+ users)**
   - Manage Members modal loaded all non-members eagerly via infinite scroll, crashing browsers at scale
   - Replaced with search-first server-side filtering (50 result cap), added SQL injection prevention, found 3 additional bugs during review
   - Merged Mar 21, 2026. Tagged for Release 1.0.0 milestone. 2 reviewer approvals

4. **PR #3210 — Flaky E2E test elimination**
   - Playwright tests failed intermittently due to shared mutable login state and hard-coded 4s sleeps
   - Replaced with per-fixture JWT cookie injection + HTMX-aware DOM waits
   - Merged Feb 28, 2026

### Section 3: All IBM/mcp-context-forge contributions

Lead-in line: *"[MCP Context Forge](https://github.com/IBM/mcp-context-forge) is an IBM open-source MCP gateway, registry, and proxy. Full list of contributions:"*

This provides context for anyone who hasn't heard of the project.

Compact 2-column table listing all 15 PRs. Ordered by PR number descending (newest first), matching the current README. The 4 highlights are not visually differentiated — the table is a uniform reference list. No merge dates in the table rows.

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

Case study links are omitted from the table. All 15 case studies remain in `/contributions/` and are discoverable by browsing the directory. Only the 4 highlights link to their case studies directly.

### Section 4: Other contributions

Section header: `## Other contributions`

Same compact 2-column table. Case study links also omitted from the table (same rationale — discoverable in `/contributions/`).

| PR | Outcome |
|---|---|
| [DjangoCRM #388](https://github.com/DjangoCRM/django-crm/pull/388) | Added 19 unit tests for previously untested AJAX view |
| [PINCE #312](https://github.com/korcankaraokcu/PINCE/pull/312) | Fixed arrow-key scrolling and selection sync in hex viewer |
| [OpenAlgo #899](https://github.com/marketcalls/openalgo/pull/899) | Added dynamic aria-labels to inline-edit inputs |
| [OpenAlgo #900](https://github.com/marketcalls/openalgo/pull/900) | Replaced silent failures with toast + inline error UI |

## Design decisions

- **No personal intro** — this repo is linked from the main personal README
- **4 highlights** — tight and punchy; maximum impact per item
- **Case studies preserved but not linked from tables** — they stay in `/contributions/`, highlights link to them, table rows don't. Interviewers who want depth can browse the directory
- **One-line project context in Section 3 lead-in** — recruiters need to know what MCP Context Forge is
- **No "Area" column** — redundant; the one-line descriptions make the area obvious
- **Other OSS separate** — visually distinct since they're different repos and different scale of contribution
- **Reviewer quotes + labels woven into highlights** — social proof without a dedicated column
- **Table ordered by PR number descending** — newest first, matching current README convention
- **TEMPLATE.md preserved** — working template for future case studies
- **Header PR count updated dynamically** — if new contributions are added, update the count
