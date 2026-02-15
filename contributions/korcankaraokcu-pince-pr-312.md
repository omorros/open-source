# korcankaraokcu/PINCE — PR #312 — Fixed arrow-key scrolling and selection sync in hex dump viewer

**Repo:** [korcankaraokcu/PINCE](https://github.com/korcankaraokcu/PINCE)
**PR:** [#312](https://github.com/korcankaraokcu/PINCE/pull/312)
**Status:** merged
**Area:** bugfix
**Stack:** Python, PyQt (Qt framework)
**Impact:** Users can now seamlessly navigate the hex dump view with arrow keys — the view scrolls at boundaries and the selection highlight follows the cursor.

---

## Context
PINCE is a reverse-engineering tool built on GDB with a Qt-based GUI. The Memory Viewer window includes a hex dump panel (`QHexView`) and an ASCII panel (`QAsciiView`, which inherits from `QHexView`). Users click cells and navigate with arrow keys to inspect process memory.

## Problem
- **Symptom:** Arrow keys stopped working at the boundary rows of the hex dump view. Pressing Up at the top row did nothing; pressing Down at the bottom row moved the selection cell off-screen without scrolling.
- **Root cause:** `keyPressEvent` in `QHexView` had no logic to request scrolling when the cursor reached the first or last visible row.
- **Scope:** Both the hex and ASCII panels were affected (shared base class). Standard row-to-row navigation within the visible area worked correctly.

## Reproduction (before fix)
1. Launch PINCE and attach to any running process
2. Open the Memory Viewer via the "Memory View" button
3. Click any cell in the HexView panel
4. Press the Down arrow key repeatedly until the last visible row
5. Press Down once more
- **Expected:** The view scrolls down by one row and the selection follows
- **Actual:** The selection moves off-screen or the cursor locks at the boundary

## Fix / Changes
- **`GUI/TableViews/HexView.py`** — Added a `scroll_requested = pyqtSignal(int)` signal to `QHexView`. Modified `keyPressEvent()` to emit `scroll_requested(-1)` when pressing Up at row 0 and `scroll_requested(1)` when pressing Down at the final row. (+7 −1)
- **`PINCE.py`** — Added `hex_view_scroll_by_row(direction)` method that calculates the next memory address based on scroll direction and column count, updates selection boundaries (start, end, address_begin, address_end) by the row offset, and calls `hex_dump_address()` to navigate. Connected the signal from both `HexView_Hex` and `HexView_Ascii` to this handler. (+14)
- Because `QAsciiView` inherits from `QHexView`, both panels gained scrolling support automatically.

## Testing / Verification
**Checks**
- Attached to a running process in the Memory Viewer
- Verified Down arrow scrolls the view when reaching the bottom row
- Confirmed Up arrow scrolls the view when at the top row
- Validated that standard row-to-row navigation within the visible area still works
- Checked that both hex and ASCII panels behave consistently

## Review notes
- **Feedback:** Reviewer noticed the selection highlight stayed at the previous position during boundary scrolling instead of following the cursor.
- **Iteration:** Added a follow-up commit that updates the selection highlight to track the current cell when scrolling occurs.

