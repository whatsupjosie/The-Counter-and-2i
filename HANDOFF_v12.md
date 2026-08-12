# Pubcast 2i — Handoff (v12: Lock In replaces Pin)

**Date:** August 12, 2026
**File:** `2i_writers_room_v12.html` — built on `2i_writers_room_v11.html`. No changes to layout, no changes to the core buffer/word-cap/reaction-window engine. This pass replaces one interaction: Pin (click a 📌 icon on a single buffer line) becomes **Lock In** (highlight text in the draft area, right-click, choose Lock in).

Verified: `node --check` passes, brace depth balanced at 0, every `getElementById` call resolves, no duplicate ids, no leftover references to the old pin mechanism anywhere — code, comments, or user-facing modal copy.

---

## What changed

**The concept is the same as Pin, the interaction is different:**
- Old: click a small pin icon on one buffer line at a time.
- New: highlight one or more lines' worth of text in the draft area, right-click, choose **Lock in** from a small menu (or **Unlock**, if everything highlighted is already locked).
- Snapped to whole lines — the app's data model is one buffer entry per line, so a selection touching any part of a line locks/unlocks that entire line, not an arbitrary character range within it.

**The visual signal changed too, on purpose:**
- Old: gold border/glow on the line, text stays cursive.
- New: the locked line's font switches from the handwritten Kalam cursive to the same Cormorant Garamond serif committed manuscript text uses — while the line is still sitting in the grey draft-area background, still logically in `buffer`, not yet `committed`. The font change *is* the "this is locked" signal. When the line naturally graduates out of the draft area later (same word-cap mechanism as before, unchanged), nothing visibly changes — it already read as finished.

**What a lock still does, unchanged from Pin:**
- Protects the line from being wiped out when an earlier line's structural edit triggers a downstream regeneration — locked lines survive the cut and keep their place, exactly like pinned lines did.
- Doesn't force the line into the manuscript early. It still graduates the normal way, whenever the draft area's word count crosses the cap.
- Doesn't block the user from manually editing it themselves (only blocks *automatic* AI regeneration).

**Renamed throughout** (state, save format, comments, user-facing copy): `pinnedLines` → `lockedLines`, `.pinned` CSS → `.locked`, on-disk field `pinned` → `locked`.

**Backward compatible:** opening a `.2i` file saved by v10/v11 (field `pinned`) or v9 (field `bufferedProtected`) still restores locked lines correctly — the load path checks `locked`, then falls back to `pinned`, then `bufferedProtected`, in that order.

---

## Not changed this pass (confirmed working as-is, no edits needed)

- The reaction window (`REACTION_MS`, ~12s after a line is read, before it enters the draft area) — unchanged.
- The word-cap crossing mechanism (`BUFFER_WORD_CAP`, `graduateIfOverCap()`) that decides when the oldest draft-area lines lock into the manuscript — unchanged. This is "crossing the buffer zone" and it already worked the way it needed to.
- Approve button — unchanged, still forces the whole current draft area to lock early.
- Find & Replace — unchanged functionally; only its user-facing copy was updated to say "lock-in" instead of "pin."
- Read/scroll speed control — unchanged; the existing speed slider already drives both TTS rate and the highlight-sweep pace.

---

## Suggested smoke test before shipping

1. **Lock a single line:** in the draft area (with content in it — generate a few lines or type some manually), double-click or drag-select part of one buffered line's text, right-click, confirm a small menu appears with "Lock in." Click it. Confirm that line's font switches to serif immediately, and it stays in the grey draft-area background (hasn't jumped to the manuscript).
2. **Lock a multi-line selection:** drag-select across two or three buffered lines (even partially into each), right-click, confirm the menu still says "Lock in" (not "Unlock," since not everything's locked yet), click it, confirm all touched lines switch to serif.
3. **Unlock:** highlight an already-locked line, right-click, confirm the menu now says "Unlock" instead of "Lock in." Click it, confirm the font reverts to cursive.
4. **Protection survives a structural edit:** lock a line, then edit an *earlier* buffer line with something structural (e.g. add "never"), confirm the locked line survives the downstream cut and keeps showing serif, while unlocked lines after the edit point get dropped as before.
5. **Locked lines still graduate normally:** keep adding content until the draft area crosses its word cap, confirm a locked line graduates into the committed manuscript along with everything else, with no visible font flicker (it was already serif).
6. **Right-click inside an active edit textarea:** click a buffered line to open its inline editor, then right-click inside the textarea — confirm the browser's normal cut/copy/paste menu appears, not the Lock In menu.
7. **Right-click with nothing selected:** right-click anywhere in the draft area without highlighting text first — confirm the normal browser context menu appears (Lock In only intercepts when there's an actual selection touching a draft-area line).
8. **Old save files still restore locks correctly:** if you have a `.2i` file saved by v10 or v11 with a pinned line in it, open it here and confirm that line still shows as locked (serif).

---

## Addendum: Chat/Reference panel close & expand

Added after the initial v12 handoff above — same file, no version bump, since this is additive to what shipped.

**What it does:** Chat and Reference panels each get their own ✕ close button in the header. Closing either one is purely a user click — nothing auto-closes, ever. Two toggle buttons live permanently in the rail (💬 for chat, 📁 for reference) as the way back — the rail is its own grid column and is never hidden, so there's no state where a closed panel becomes unreachable. Close one panel and the other fills the full column height automatically (plain flexbox, no extra logic needed). Close both and the manuscript actually reclaims that width — the shell's grid collapses that column to 0 instead of leaving dead space.

State machine verified by trace: close chat → close ref → reopen chat → close chat again → reopen ref, confirming no combination leaves the toggles unreachable.

### Smoke test
1. Click ✕ on the chat panel — confirm it disappears and the reference panel fills the left column's full height.
2. Click the 💬 rail button — confirm chat reopens, split returns to normal.
3. Close both panels — confirm the manuscript area visibly widens to fill the freed space, not just blank space where the panels were.
4. Reopen one via its rail toggle — confirm the shell layout returns to the normal three-column split, not still collapsed.
5. Confirm nothing closes or reopens on page load, on save, on document open, or on any other action — only clicking a ✕ or a rail toggle changes panel state.

---

## Addendum: self-audit findings, fixed

Went back through everything built tonight looking specifically for what was weak, not just re-confirming what already passed. Found two real things in the Lock In feature, both fixed here — no version bump, same file.

**Real bug — stale index race.** The Lock In context menu captures which buffer indices a selection touched at right-click time, and applies Lock in/Unlock to those same index numbers when the menu button is clicked. If playback was actively running while the menu stayed open, the reaction window firing and lines auto-graduating out of the buffer could shift what those indices pointed at before the click landed — Lock in could silently apply to the wrong line. Fixed by calling the same `pauseForEdit()` every other line-editing interaction in this app already uses, right when the menu opens — with playback paused, the buffer can't shift on its own, and any click elsewhere already closes the menu first via the existing document-level click listener.

**Polish gap.** The browser's native text-selection highlight stayed visible over the line after locking/unlocking it. Now clears automatically.

**Standing limitation, not new to this pass:** everything built tonight — v9 through this addendum — has been verified through static analysis only (syntax checks, ID cross-references, manual code tracing, isolated logic simulation in Node). None of it has run in an actual browser, because this environment doesn't have one. That's true of every version, not just this one.

### Smoke test addition
9. **Lock In during active playback:** start Play, let it read a couple lines into the draft area, right-click and highlight a line while it's still actively reading — confirm playback visibly pauses the moment the menu opens (matching what already happens when you click a line to edit it), and that Lock in applies to the correct line.
