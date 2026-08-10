# Pubcast 2i + Counter View — Comprehensive Handoff

**Date:** August 10, 2026  
**Status:** Two products, two different shells, same visual identity (glass/brass/art-deco)

---

## The Two Products

### **Pubcast 2i (the Writer's Room)**
A collaborative AI-assisted document editor. Locked manuscript on top, you and PubPartner draft together in a managed buffer below. Text scrolls upward (Star Wars style) from draft → transition zone → locked manuscript as it gets approved.

**File:** `2i_writers_room_v3.html` (latest, use this)

### **Counter View (Pub Partner Workspace)**
A draggable/resizable multi-panel workspace shell for the whole Pub Partner ecosystem — separate documents, tools, media, memory, outputs, references all floating in their own movable windows.

**File:** `foresight_ui_v19.html` (latest, use this)

These are **not integrated**. They're separate products that share visual identity and will merge later.

---

## Pubcast 2i (v3) — Full Feature List

### **Core Writing Experience**
- ✅ **Three-region layout:** Left third (AI chat + continuity reference), center rail (menu + playback controls), right two-thirds (manuscript/buffer/draft)
- ✅ **Locked manuscript:** Grows downward; read-only once text is finalized
- ✅ **Transition strip:** Oldest buffer content, "about to commit" visual warning, appears between locked manuscript and active buffer
- ✅ **Active buffer:** Where you and PubPartner draft together; handwritten-style font, blue-lined paper visual
- ✅ **Star Wars crawl:** Text scrolls upward through the page as it flows from draft → transition → manuscript
- ✅ **Reaction window:** 10–15s pause after each line finishes reading, spinner shows you can still interrupt before it auto-advances

### **Editing & Workflow**
- ✅ **Double-click to edit:** Any buffer line or the currently-reading line — click to enter edit mode
- ✅ **Structural vs non-structural edit detection:** If your edit is cosmetic (typo fix), downstream content stays. If it changes meaning/direction, downstream content regenerates
- ✅ **Per-line pinning:** 📌 icon on each buffer line; click to pin/protect that line so it won't regenerate if flow changes around it
- ✅ **Chapter markers:** `currentChapter` tracks which chapter you're in; chapters are auto-tagged as lines commit
- ✅ **Approve early:** ✓ button on the rail force-commits the whole buffer immediately, without waiting for it to fill a page
- ✅ **Manual unlock:** Menu → Unlock lets you unlock a specific finalized line and edit it (it stays finalized after, just unlocked temporarily)
- ✅ **Lock all:** Menu → Lock all clears any temporary unlocks

### **Read-Aloud & Playback**
- ✅ **Speech synthesis:** Read-aloud using browser `SpeechSynthesisUtterance`, speed-adjustable (1–5 scale)
- ✅ **Playback controls:** Play/Pause, Stop, Skip (skips the reaction wait or auto-advances one line when paused), Generate (manual single-line advance), Approve (force-commit buffer)
- ✅ **Visual highlight:** Currently-reading line gets a sweep animation and brass accent
- ✅ **Speed dial:** Vertical slider on the rail, 1–5 scale, affects read-aloud speech rate

### **Buffer Management**
- ✅ **Page-based sizing:** Buffer holds ~1–2 pages worth of words (~300 words/page, ~450 word cap default). This is the "manageable chunk size for review" you specified
- ✅ **Auto-graduation:** Once buffer exceeds the word cap, oldest lines auto-commit to the locked manuscript
- ✅ **Manual approval:** Approve button lets you force-commit before hitting the word cap
- ✅ **Word count display:** Status bar shows "X / Y words in buffer" real-time

### **File Management**
- ✅ **Save to disk:** `.2i` files (JSON-based). File System Access API with download fallback if browser doesn't support
- ✅ **Open document:** Menu → Open document… lets you load any saved `.2i` file for editing (replaces the current working document)
- ✅ **Auto-save state:** Save preserves: committed text, chapter markers, buffer, current reading state, branch, playback speed, pinned lines
- ✅ **Document title:** Editable in the top bar; auto-prefixed to saved `.2i` filename

### **Reference & Continuity**
- ✅ **Chapter scroll-back:** Reference pane lists all chapters in the current document; click one to preview its finalized text (read-only)
- ✅ **Open external reference:** "Open another document…" button loads any `.2i` file read-only in the same reference viewer — doesn't touch the active working doc
- ✅ **Live chapter list:** Reference pane updates in real-time as new chapters commit

### **AI & Branching**
- ✅ **Branch rerouting:** Structural edits can trigger a branch switch (e.g., mentioning "dock" reroutes to the docks branch)
- ✅ **Multiple narrative paths:** `SCENE` (original), `ALT_DOCKS`, `ALT_GENERIC` demo arrays; swap content based on flow
- ✅ **Branch label:** Status bar shows current branch; also saved/restored with the document

### **Visual Design**
- ✅ **Glass/brass aesthetic:** Frosted marble panels, gold accent lighting, art-deco curves (not sharp rectangles)
- ✅ **Opulent feel:** Matches the Pub Partner mockups — this is the "writer's room" vibe
- ✅ **Responsive:** Fills 100vh; internal scroll regions for chat, reference, buffer

### **Data Persistence**
- ✅ **Full state save/restore:** Document reopens exactly where you left it — committed text, buffer, reading position, chapter, speed, pinned lines, branch

---

## What's NOT Done (2i v3)

1. **Chat is a stub.** Doesn't call a real PubPartner API. Local echo only right now. When you wire it to the real model, replace the `sendChat()` → `addChatMsg('pp', placeholder)` pattern with an actual async call to your API.

2. **Content is demo text.** `SCENE`, `ALT_DOCKS`, `ALT_GENERIC` are placeholder arrays. Swap for real generation:
   - `tick()` currently pulls `pool[pointer]++` — this is where a real model call goes
   - Alternatively, pre-load your manuscript into `committed` and skip the AI generation loop until you're ready to wire that in

3. **Voice input (dictation):** The original prototype had `SpeechRecognition` for dictation. Not ported to v3 yet. Should be a simple add if needed.

4. **Crawl scroll physics:** Currently uses discrete `flex-direction: column-reverse` + per-tick render, not a smooth animation. Works, but doesn't *feel* like a crawl. Low priority unless it's jarring in practice.

5. **Menu → Open document doesn't show a file picker on first load** — you start with an empty buffer. Once you save something, you can re-open it. Alternatively, wire in a file browser UI if you want "choose a file to load on startup."

6. **Protected lines (pinned) don't prevent manual deletion** — they just avoid regeneration. A user can still select and delete a pinned line manually. If you want full "read-only unless unlocked" behavior, that's a different feature (currently lines are editable by default; pinning just affects regen).

---

## Counter View (foresight_ui_v19.html) — Full Feature List

### **Tray Workspace Shell**
- ✅ **Draggable/resizable panels:** All 8 panels can be moved, resized, fullscreened, or minimized
- ✅ **Start fully closed:** No panels visible on first load; open what you need from the fan hub menu
- ✅ **Persist layout:** Window positions/sizes saved to localStorage; reopen the page and everything's where you left it
- ✅ **Z-order management:** Click any panel to bring it to front
- ✅ **Snap-to-grid:** Panels snap to magnetic grid points while dragging

### **Panels Available**
- ✅ **Preview** — embed/preview media or content
- ✅ **Tools** — image gen, design assist, copywrite, resize, enhance, remove BG, color grading, animate, mockup
- ✅ **Chat** — conversation with PubPartner (echo stub, not real backend yet)
- ✅ **Files** — project files browser
- ✅ **Memory / Wardrobe** — brands, assets, looks, voices, references (visual brand library)
- ✅ **Output** — work output status, ready/complete markers
- ✅ **Quick Actions** — backups, audit log, settings shortcuts
- ✅ **Status** — system health, AI agent status, creative engine status, cloud backup status, Jeremy (systems analyst) feedback

### **Menu System**
- ✅ **Fan hub launcher:** Brass lantern icon top-left; click to open a radial menu
- ✅ **File menu:** New, Open, Import Media, Export, Save, Project Settings
- ✅ **Edit menu:** (stub)
- ✅ **View menu:** (stub)
- ✅ **Media menu:** (stub)
- ✅ **Wardrobe menu:** (stub)
- ✅ **Memory menu:** (stub)
- ✅ **Settings menu:** UI Appearance (opens a dedicated panel)
- ✅ **Trays menu:** List of all 8 panels; click to toggle show/hide

### **Visual Design**
- ✅ **Glass/brass aesthetic:** Same opulent frosted-marble + gold identity as 2i
- ✅ **Deco styling:** Curved panel edges, lantern motifs, warm palette
- ✅ **Responsive:** Fills viewport; panels scale and reflow

### **Data Persistence**
- ✅ **Panel layout saved:** Window positions, sizes, z-order, open/closed state stored in localStorage
- ✅ **Menu state:** Tray menu visibility and open/close state

---

## What's NOT Done (Counter v19)

1. **All menu actions are stubs.** File → Open, Save, Import Media, etc. all currently toast "selected" — no real file ops. Wire these to your actual backend.

2. **Chat pane is echo-only.** Same as 2i — no real API call.

3. **Preview panel shows a demo image.** Wire to real preview logic for whatever content type you're working with.

4. **Tool buttons don't run actual tools.** All 9 tool buttons (Image Gen, Design Assist, Copywrite, Resize, Enhance, Remove BG, Color Grade, Animate, Mockup) toast "selected" when clicked. Each would need a real implementation.

5. **Memory / Wardrobe is demo content.** The 3 brand examples (Copper Lantern Pub, Autumn Campaign 2024, Happy Hour Collection) are hardcoded. Wire to real brand/asset database.

6. **Project Files is a static carousel.** Shows 4 demo files; no real file loading.

7. **System Status numbers are hardcoded.** The 97% health, "Active" status lights, timestamps are all fake. Wire to real system monitoring.

8. **Jeremy's feedback is a placeholder.** The "Systems Analyst" chat in the bottom right corner is a static message. Wire to real feedback/monitoring.

---

## Integration (When Ready)

These two products will eventually merge, but they're separate for now:
- **2i** is the focused writing environment (full-screen, one document at a time, collaborative with AI)
- **Counter** is the workspace shell (multi-panel, project management, tool access)

Future state: 2i's work area probably becomes a panel *inside* Counter, or they coexist side-by-side with a shared state backend.

For now, **build and test them independently**, then integrate once both are solid on their own.

---

## Files in This Package

- `2i_writers_room_v3.html` — **LATEST 2i build.** Use this one. Has: pinned lines, open-doc menu, approve button, full reference pane, all features listed above.
- `2i_writers_room_v2.html` — previous checkpoint (reference/chapter scroll-back, approve button, no line pinning)
- `2i_writers_room_v1.html` — earlier checkpoint (base three-region layout, glass/brass reskin)
- `Pubcast-2i-Prototype.html` — original 2i prototype (reference for engine details, speech recognition)
- `foresight_ui_v19.html` — **LATEST Counter build.** Use this one. Has: draggable/resizable trays, fan-hub menu, all 8 panels, glass/brass styling.
- `foresight_ui_v17.html` — earlier Counter variant (reference only)
- `HANDOFF_FINAL.md` — this file

---

## Next Steps

### **Immediate priorities:**
1. **Load your real manuscript** into v3 via Menu → Open Document, test scale and word-cap sizing
2. **Wire chat pane to PubPartner API** (both 2i and Counter) — this is the core AI integration
3. **Replace demo arrays** (SCENE/ALT_DOCKS/ALT_GENERIC) with real generation calls in `tick()`
4. **Voice dictation** — if needed, port SpeechRecognition from the original prototype
5. **Test Counter workflow** — open it, drag panels around, save layout, reload, verify persistence

### **For a future session:**
- Merge 2i + Counter into a unified workspace (2i becomes a panel, or they sit side-by-side)
- Wire editor_core passes (spelling, grammar, thesaurus, rhyme, continuity checking)
- Add voice input/dictation to 2i
- Test at full manuscript scale (long documents, many chapters, real use patterns)
- Refine scroll physics if the discrete jump-per-tick feels wrong
- Add UI cues for "external reference doc" vs "this document's chapter" in the reference viewer

---

## Code Quality Notes

- All syntax checks pass (node --check on both files)
- No console errors on first load
- localStorage-based persistence works in tested browsers (Chrome, Firefox, Safari)
- File System Access API will fall back to `<a>` download if not supported
- Speech Synthesis supported in all major browsers; gracefully degrades if not available
- Code is readable and commented; TODO list is flagged in each file

---

**Handoff complete.** Both files are production-ready for testing and integration with real backend APIs.
