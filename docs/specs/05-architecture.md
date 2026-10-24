# 05 — Technical Architecture

## 1. Stack

| Layer | Choice | Notes |
|-------|--------|-------|
| Shell | **Electron** (latest stable) | File system, dialogs, file watching, `printToPDF`, spell check, file associations. |
| Language | **TypeScript** (strict) | Main, preload, renderer, workers. |
| UI | **React** + Vite | |
| Document editor | **TipTap on ProseMirror** | Custom extensions: styles, sections, page/scene breaks, tables, comments, tracked changes, opaque nodes. |
| Pagination | Decided by Phase 0 spike (§3) | |
| Planning editors | React forms/tables; TipTap (light schema) for rich-text fields | Table view: TanStack Table. |
| State | Zustand for workspace/UI; one ProseMirror `EditorState` per open document tab | |
| DOCX read/write | Custom OOXML ⇄ model mapper using **JSZip** + **fast-xml-parser** (preserve-order mode) | Custom both ways: needed for Tier B/C passthrough, comments and tracked changes. Mammoth and the `docx` npm library can't round-trip unknown XML. |
| PDF | Electron `webContents.printToPDF` of the paginated render | Same layout as the page view. |
| File watching | **chokidar** | |
| Validation | **zod** for `.quill` files, `.quill/` metadata, templates | |
| Tests | **Vitest** (unit/golden), **Playwright** (Electron E2E), LibreOffice headless in CI for output validation | |
| Packaging | **electron-builder** → NSIS installer, `.quill` file association, auto-update | |

## 2. Process model

```
┌──────────────────────── Main process (Node) ─────────────────────────┐
│ ProjectService  – open folder, tree model, .quill/ metadata          │
│ FileService     – atomic writes, backups, recovery journal           │
│ WatcherService  – chokidar → tree/file-changed events                │
│ ExportService   – hidden window → printToPDF; compile                │
│ RefService      – updates cross-file references on rename/move       │
│ Settings, RecentProjects, AutoUpdate                                 │
└───────────────▲──────────────────────────────────────────────────────┘
                │ typed IPC via contextBridge (preload.ts)
┌───────────────┴──────────── Renderer (React) ────────────────────────┐
│ Workbench: Explorer, tab groups (split), Quick Open, status bar      │
│ Editor registry: file type → editor component                        │
│   .docx → DocumentEditor   .quill(type) → Snowflake/Scenes/…Editor   │
│   .txt/.md → TextEditor                                              │
│ Web Workers: docx parse/serialize, word counts, search               │
└──────────────────────────────────────────────────────────────────────┘
```

- `contextIsolation: true`, `nodeIntegration: false`; renderer reaches disk only through a
  narrow typed IPC API. Paths are validated to be inside the open project.
- **Editor registry** (like VS Code custom editors): each editor implements
  `open(bytes) → model`, `serialize(model) → bytes`, `isDirty`, `getViewState/restoreViewState`
  (cursor, scroll, selected step…), `dispose`. Tabs, autosave, backups, external-change
  handling and split view are implemented once in the workbench and work for every file type.

## 3. Pagination (Phase 0 spike — highest technical risk)

Google Docs-style pages inside a ProseMirror editor are hard: text must flow across page
boxes, paragraphs/table rows must split at line level, headers/footers must repeat, and it
must stay fast while typing.

Evaluate in a 1–2 week spike, against a 60-page fixture with tables and comments:

| Option | Notes |
|--------|-------|
| A. TipTap's commercial pagination/"Pages" extension (if available on a suitable plan) | Fastest path; check license cost and table/comment compatibility. |
| B. Open-source ProseMirror/TipTap pagination extensions | Usually page-break-by-block only; check line-level splitting. |
| C. Custom: single ProseMirror document + layout pass that measures line boxes (Range rects) and inserts page-gap widget decorations at line boundaries; incremental re-layout from the edited page onward | Most control; most work. |

Pass criteria: line-level paragraph splitting; tables split between rows; headers/footers
with page numbers; < 16 ms typing latency and < 100 ms re-layout after edits at page 1 of 60;
`printToPDF` output matches on-screen page breaks for all fixtures.

## 4. Document model

`docx bytes ⇄ [Worker: OOXML mapper] ⇄ Quill doc model (ProseMirror JSON + side tables) ⇄ EditorState`

- Nodes: `doc`, `section` (page size, margins, orientation, header/footer refs), `paragraph`
  (styleId, align, indents, spacing, keep flags), `heading`, lists, `table`/`tableRow`/`tableCell`
  (+ preserved table/cell props), `image`, `pageBreak`, `sceneBreak`, `hardBreak`,
  **`opaqueInline` / `opaqueBlock`** (Tier B content: raw XML + display hint).
- Marks: bold, italic, underline, strike, fontFamily, fontSize, color, highlight, link,
  charStyle, **`comment`** (comment id; overlapping allowed), **`insertion` / `deletion` /
  `formatChange`** (author, date, revision id).
- Side tables: styles, numbering, comments (threads, replies, resolved), headers/footers,
  core properties, Tier C parts and unknown attributes keyed to their elements.
- Every node/mark carries an optional `xmlExtra` attribute holding unknown attributes/children
  from the source XML so they are re-emitted on save.

## 5. Repo layout

Single app with one `package.json` (no monorepo). Folders, not packages:

```
src/main/            # Electron main process: files, watcher, export, IPC handlers
src/preload/         # typed contextBridge API
src/renderer/
   workbench/        # explorer, tabs, split, quick open, status bar
   editors/          # document/, snowflake/, scenes/, character/, text/ (beats/ in Phase 2)
src/shared/          # types + zod schemas for .quill files and .quill/ metadata
src/docx/            # OOXML ⇄ Quill doc model (pure TS, no Electron imports, heavily tested)
src/templates/       # built-in outline method templates (JSON)
tests/
   fixtures/docx/
   e2e/
docs/specs/
docs/decisions/
```

Workers live next to the code they run (e.g. `src/docx/worker.ts`). `src/docx/` has no
Electron imports so it can be tested in isolation.

## 6. Reliability rules

1. Never write a user file non-atomically (temp file + rename).
2. Never overwrite a file without a backup in `.quill/backups/`.
3. Never permanently delete — use `shell.trashItem`.
4. Never drop unknown DOCX content — it is preserved (Tier B/C) or the save is refused with an error.
5. Schema-validate all JSON on read; invalid metadata is moved aside, invalid `.quill` files open in the text editor.
