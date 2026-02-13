# marketcalls/openalgo — PR #899 — Added dynamic aria-labels to inline-edit inputs for accessibility

**Repo:** [marketcalls/openalgo](https://github.com/marketcalls/openalgo)
**PR:** [#899](https://github.com/marketcalls/openalgo/pull/899)
**Status:** merged
**Area:** accessibility
**Stack:** React, TypeScript
**Impact:** Screen reader users can now identify every inline-edit input field across admin pages, bringing the UI closer to WCAG 2.1 compliance.

---

## Context
OpenAlgo is an open-source algorithmic trading platform. Its admin dashboard includes inline-editing interfaces for market timings, freeze quantities, and holiday schedules. These pages let administrators modify values directly in table rows.

## Problem
- **Symptom:** Screen readers announced inline-edit inputs as unlabeled fields — users relying on assistive technology could not tell which value they were editing.
- **Root cause:** The `<Input>` components had no `aria-label` attributes and no associated `<label>` elements.
- **Scope:** Affected five input fields across three admin pages: MarketTimings (start/end time), FreezeQty (freeze quantity), and Holidays (special session start/end time).

## Reproduction (before fix)
1. Open any admin page with inline editing (e.g., Market Timings)
2. Click an edit button to activate inline editing on a row
3. Focus an input field using a screen reader (e.g., NVDA, VoiceOver)
- **Expected:** Screen reader announces the field purpose, e.g., "Start time for NSE"
- **Actual:** Screen reader announces a generic unlabeled input

## Fix / Changes
- **`MarketTimings.tsx`** — Added contextual `aria-label` attributes to start time and end time inputs (e.g., `aria-label={`Start time for ${exchange}`}`)
- **`FreezeQty.tsx`** — Added `aria-label` to the freeze quantity input (e.g., `aria-label={`Freeze quantity for ${symbol}`}`)
- **`Holidays.tsx`** — Added `aria-label` attributes to special session start and end time inputs

Labels are dynamic, incorporating the exchange or symbol name from the current row to provide full context.

## Testing / Verification
**Checks**
- Inspected rendered HTML to confirm `aria-label` attributes appear on all five inputs
- Verified labels are contextual (include exchange/symbol name, not generic text)
- Confirmed no XSS risk — interpolated values are React string attributes, not `dangerouslySetInnerHTML`, and the exchange/symbol names already appear elsewhere in the UI

## Risks / Trade-offs
- **Risk:** Minimal — additive change to JSX attributes, no logic or layout modifications.
- **Mitigation:** React's attribute handling automatically escapes interpolated strings, preventing HTML injection.

## Review notes
- **Feedback:** Maintainer described the implementation as "clean, correct, and complete" and noted coverage went beyond the original issue scope (MarketTimings) to include FreezeQty and Holidays.

## Takeaways
- Scanning sibling components for the same accessibility gap turned a single-file fix into a more complete contribution.
