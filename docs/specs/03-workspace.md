# 03 — Workspace: Projects, File Tree & Switching

The workspace works like a code editor (VS Code is the reference): open a folder, see every
file in one Explorer tree, open files in tabs, jump between them with the mouse or keyboard.
Chapters, scenes, outlines, character sheets and notes are all just files and behave identically.

## 1. Opening a project

- **File → Open Folder…** opens any folder as a project. No import or conversion step.
- **File → New Project…** creates a folder from a template:
  - *Empty*
  - *Novel* — `Manuscript/`, `Planning/`, `Characters/`, `Notes/`
  - *Novel — Snowflake* — the above plus `Planning/Snowflake.quill` and `Planning/Scenes.quill`
- Start screen lists recent projects (last opened, word count) and pinned projects.
- Reopening the app restores the last project, open tabs, active tab, and each tab's
  cursor/scroll position.

Example project:

```
The Lighthouse/
├── Manuscript/
│   ├── 01 Arrival.docx
│   ├── 02 The Keeper.docx
│   └── 03 Storm.docx
├── Planning/
│   ├── Snowflake.quill
│   ├── Save the Cat.quill
│   └── Scenes.quill
├── Characters/
│   ├── Mara Quinn.quill
│   └── Elias Vane.quill
├── Notes/
│   ├── Research.docx
│   └── ideas.md
└── .quill/                  ← hidden; app bookkeeping only (§5)
```

Templates are a starting point only — the author can organize folders however they like,
use one file per chapter or one per scene, and nest folders (e.g. `Part 1/Chapter 3/scene files`).

## 2. Explorer (file tree)

- Shows all files Quill can open (`.docx`, `.quill`, `.txt`, `.md`) and folders.
  "Show all files" toggle reveals everything else (greyed out, opens with the OS default app).
- Icons by type: document, Snowflake, beat sheet, character, scene list, text.
- Document rows can show word count and an optional status chip
  (*Idea, Outlined, Drafting, Draft done, Revised, Final*) and color label (toggle in View).
- **Ordering**: alphabetical by default, as in a text editor (numeric-aware, so `2` < `10`).
  Optional per-folder **custom order** via drag-and-drop, stored in `.quill/order.json`.
  Novelists who prefer numbered file names need nothing extra.
- Context menu: New document, New outline/beat sheet/character/scene list, New folder,
  Rename (F2), Duplicate, Copy path, Reveal in Explorer, Include/exclude from manuscript,
  Delete → OS Recycle Bin (never permanent).
- Drag-and-drop to move files/folders; moving/renaming updates references in `.quill` files.
- Changes made outside Quill (Explorer, Word, git) appear within 1 s.
- Filter box at the top of the tree (type to filter by name).

## 3. Tabs and switching

Behavior mirrors VS Code:

| Action | Behavior |
|--------|----------|
| Single-click a file | Opens in a **preview tab** (italic title); the next single-click replaces it. |
| Double-click, or start editing | Pins the tab. |
| Ctrl+P | **Quick Open**: fuzzy search across all file names in the project (and headings in documents, Phase 2). |
| Ctrl+Tab / Ctrl+Shift+Tab | Cycle tabs in most-recently-used order (hold Ctrl to see the list). |
| Ctrl+1..9 | Jump to tab N. |
| Ctrl+W / middle-click | Close tab. Ctrl+Shift+T reopens the last closed tab. |
| Ctrl+PgDn / Ctrl+PgUp | Next/previous tab. |
| Alt+↓ / Alt+↑ | Next/previous document in manuscript order. |
| Ctrl+B | Toggle sidebar. |
| Ctrl+\\ | **Split editor** — e.g. Snowflake outline on the left, chapter on the right (v1). |
| Drag tab | Reorder, or drop onto the other split group. |

Performance and state:
- Switching between open tabs < 50 ms. Each open tab keeps its full state in memory:
  content, cursor/selection, scroll position, undo history.
- Opening a not-yet-open file < 300 ms for a typical 5,000-word chapter.
- Recently closed files are kept in a cache (default 10) for instant reopen.
- Dirty indicator (•) on tabs with unsaved changes; switching never prompts to save (§4).
- The same file can be open in both split groups; edits stay in sync.

## 4. Saving

- **Autosave** 2 s after typing stops, and on tab switch, window blur, and app close.
  Ctrl+S saves immediately.
- Status bar: *Saving… / Saved ✓ / Save failed ⚠*.
- Saves are atomic (write temp file in the same folder, then rename).
- Before overwriting a `.docx`, the previous version is copied to `.quill/backups/`
  (keep last 20 per file, configurable). File → Restore previous version… lists them.
- On save failure (file locked by Word, disk full, permissions): keep content in memory,
  persistent warning, retry every 10 s, offer Save As.
- **Crash recovery**: unsaved state journaled to `.quill/recovery/` every 5 s; offered on next launch.
- **External changes**: if the file changes on disk and the tab has no unsaved edits, reload
  silently (keep cursor near the same spot). If it has unsaved edits: banner with
  *Keep mine / Load from disk*.

## 5. Hidden `.quill/` folder — bookkeeping only

Nothing the author writes lives here. Deleting it loses nothing but convenience.

```
.quill/
├── project.json     # manuscript inclusion list, statuses, labels, settings
├── order.json       # custom ordering per folder (if used)
├── session.json     # open tabs, cursors, layout (per machine)
├── dictionary.txt   # project spell-check words
├── backups/         # previous versions of saved files
├── recovery/        # crash-recovery journal
└── .gitignore       # ignores session, backups, recovery
```

All JSON carries a `schemaVersion` for migrations; invalid files are moved aside and reset,
never crash the app.

## 6. Manuscript & compile

- **Manuscript** = documents marked "included", in tree order (folder by folder, respecting
  custom order). By default every `.docx` under `Manuscript/` (if present) is included;
  otherwise every `.docx` in the project. Toggle per file/folder via context menu.
- **File → Compile Manuscript…** produces one `.docx` or `.pdf`:
  format preset, chapter headings (from file name / first heading / none), chapter numbering
  (*Chapter 1*, *Chapter One*, *1*), separator between files (page break or scene break —
  useful for one-file-per-scene projects), title page, comments/tracked changes stripped
  or kept.
- Compile never modifies source files.

## 7. Project search (Phase 2)

Search & replace across all documents and planning files, results grouped by file with
match previews (Ctrl+Shift+F), like a code editor.
