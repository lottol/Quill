# 06 — Roadmap & Acceptance Criteria

Each phase ends with a usable tool. Estimates are rough, for one developer working with AI coding assistance.

## Phase 0 — Foundations & spikes (≈2–3 weeks)

- Single-package scaffold (Electron + Vite + React + TS), lint/format, Vitest, Playwright, CI.
- Typed IPC skeleton, settings store, workbench shell (empty Explorer, tab bar, status bar).
- `src/docx/` skeleton, fixture corpus, golden-test harness.
- **Pagination spike** (05 §3) → written decision.
- **DOCX passthrough spike**: prove a file with comments, tracked changes, tables and a text
  box survives import → export unchanged.

**Done when:** `npm run dev` opens the shell; CI runs unit + E2E smoke tests; both spikes have a go/no-go decision recorded in `docs/decisions/`.

## Phase 1 — v1: write and outline in one workspace (≈8–12 weeks)

**Workspace**
- Open Folder / New Project templates; Explorer with all supported types, filter, context menu, drag-and-drop, external-change watching.
- Tabs with preview/pinned behavior, MRU Ctrl+Tab, Quick Open (Ctrl+P), split editor, preserved per-tab state.
- Autosave, atomic save, backups + restore, crash recovery.

**Documents**
- Paginated Google Docs-style page view with headers/footers and page numbers.
- v1 feature tier from 01 §3: formatting, styles, lists, images, tables, comments, view/accept/reject tracked changes, find & replace, spell check, word counts.
- DOCX Tier A/B/C support (02 §2).
- PDF export of a document; Standard Manuscript Format preset.
- Compile manuscript to `.docx` and `.pdf`.

**Planning**
- `.quill` file format, schemas, plain-text fallback.
- Snowflake editor, steps 1–10, including "create documents from scene list".
- Scene list (table view) with document links; character files.
- Export planning files to `.docx`/`.pdf`.

**Acceptance criteria**
- [ ] Open a project with 30 chapters + Snowflake + 10 character files; switching between open tabs < 50 ms; Ctrl+P finds any file in < 100 ms.
- [ ] Snowflake outline and a chapter can be viewed side by side in split view and both edited.
- [ ] A `.docx` with tables, merged cells, threaded comments, and tracked changes from two authors opens, displays correctly, and after edits saves without losing any of them; Word and Google Docs show the same comments/changes.
- [ ] Import → save without edits passes all golden tests.
- [ ] Page view and exported PDF have identical page breaks for every fixture.
- [ ] Typing latency < 16 ms in a 60-page document.
- [ ] Killing the process mid-edit loses at most 5 s of typing.
- [ ] Complete all 10 Snowflake steps for a sample novel; step 10 creates linked chapter files.
- [ ] Renaming a chapter in Explorer keeps its scene link intact.

## Phase 2 — Richer writing & more methods (≈6–8 weeks)

- Suggesting mode (author-created tracked changes), footnotes as editable, table merge/split & column resize, ruler indents, super/subscript, small caps.
- Project-wide search & replace; headings in Quick Open.
- Beat sheets: Three-Act, Save the Cat!, Custom; beat timeline; assign scenes to beats.
- Corkboard view; focus mode; word targets and session goals; book-interior preset with mirror margins.
- Markdown export.

## Phase 3 — Polish (ongoing)

- Pageless mode, version history view, writing statistics.
- Hero's Journey, Seven-Point, Story Circle templates; location files; character images.
- macOS/Linux builds, code signing, auto-update.

## Risks

| Risk | Impact | Mitigation |
|------|--------|-----------|
| Pagination in the editor | Core v1 requirement; hardest piece | Phase 0 spike before committing; commercial option as fallback. |
| DOCX comments/tracked changes fidelity | Losing editor feedback is unacceptable | Passthrough-first design, golden tests, real-world fixtures, backups. |
| v1 scope growth (pages + tables + comments + Snowflake) | Delay | Keep the v1 tier strict; everything else goes to Phase 2. |
| Long-document performance | Laggy typing | Incremental layout, workers, CI perf budget on a 60-page fixture. |
| File locked by Word on Windows | Save failures | Detect EBUSY/EPERM, retry, clear UI, Save As fallback. |
| Copyright on method guidance text | Legal | Original paraphrased guidance; cite method names/authors only. |
