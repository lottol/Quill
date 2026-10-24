# 02 — File Formats: DOCX import/export, PDF export, `.quill` files

## 1. Summary

| Format | Open/import | Save/export | Notes |
|--------|-------------|-------------|-------|
| `.docx` | ✅ v1 | ✅ v1 | Documents are edited in place as `.docx`. |
| `.pdf` | — | ✅ v1 | Single document, outline file, or compiled manuscript. |
| `.quill` | ✅ v1 | ✅ v1 | Quill's planning files (outlines, characters, scenes). See §5. |
| `.txt` / `.md` | ✅ v1 (plain editor) | ✅ v1 | Notes; also Markdown export of documents (Phase 2). |
| `.odt`, `.rtf`, `.epub` | — | Later | |

## 2. DOCX — fidelity tiers

The rule: **Quill never silently destroys content in a `.docx`.** Every OOXML feature falls
into one of three tiers.

### Tier A — Editable (v1)

Round-trips losslessly *and* can be edited in Quill:

- Paragraphs/runs: bold, italic, underline, strike, font, size, color, highlight.
- Paragraph properties: alignment, indents, spacing, line spacing, keep-with-next,
  keep-together, page-break-before, widow control.
- Styles (`styles.xml`) with `basedOn` inheritance and document defaults.
- Lists (`numbering.xml`), multi-level.
- Page breaks; sections with page size, orientation, margins.
- Headers/footers: text and page-number fields, different-first-page.
- Hyperlinks.
- Inline images (PNG/JPEG/GIF; others preserved as Tier B).
- **Tables**: rows, columns, cell text and paragraphs inside cells, borders, shading,
  column widths, merged cells (merged cells *displayed* in v1; merge/split editing Phase 2).
- **Comments** (`comments.xml`, plus `commentsExtended.xml` for replies/resolved state):
  author, date, anchored range, replies, resolved flag.
- **Tracked changes** (`w:ins`, `w:del`, `w:rPrChange`, `w:pPrChange`): displayed with
  author/date, accept/reject. Creating new tracked changes is Phase 2.
- Core properties (title, author).

### Tier B — Preserved and displayed, not editable (v1)

Round-trip untouched; shown as a non-editable placeholder or best-effort rendering:

- Footnotes/endnotes (become Tier A in Phase 2).
- Floating/anchored images and shapes, text boxes, WordArt, SmartArt, charts, equations,
  embedded objects, fields other than page numbers (TOC, cross-references), content
  controls, columns.

Placeholder example: a grey inline chip *"[Text box — open in Word to edit]"*, selectable
and movable/deletable as a unit.

### Tier C — Opaque passthrough (v1)

Everything else in the package (custom XML parts, theme, settings, unknown elements and
attributes on known elements, `mc:AlternateContent`) is stored verbatim and re-emitted on save.

### Fidelity guard

If a file contains Tier B content, the tab shows a dismissible info bar:
*"This file has text boxes and equations Quill can't edit. They'll be kept as-is."*
There is no "this will be removed" case anymore — if an element can't be preserved for some
reason (e.g. the user deletes a paragraph containing a Tier B anchor), normal undo applies
and the pre-save backup (03 §4) is the safety net.

## 3. DOCX import/export requirements

- Open via Explorer, File → Open, or drag-and-drop.
- Malformed, password-protected, or `.doc` (legacy binary) files: clear error message, never crash.
  (Legacy `.doc` support is out of scope; suggest re-saving as `.docx`.)
- Parsing and serialization run in a worker thread.
- Saved files open without repair prompts in Word 2016+, Word for Mac, Google Docs, and LibreOffice.
- Comments and tracked changes created/resolved in Quill appear correctly in Word and Google Docs.
- Saves are atomic and backed up (03 §4).
- Author name for new comments comes from Settings (prompted on first comment).

## 4. PDF export

### 4.1 Scope

- Export current document (File → Download/Export → PDF, or Ctrl+P).
- Export a planning file (e.g. Snowflake outline as a formatted synopsis).
- Compile manuscript — documents in manuscript order into one PDF (03 §6).

### 4.2 Options

- Page size: Letter, A4, A5, 5"×8", 5.5"×8.5", 6"×9", custom — default *as in document*.
- Margins (mirror margins for print books — Phase 2).
- Format preset: *As formatted*, *Standard Manuscript Format*, *Book interior* (Phase 2).
- Include comments: none / as margin notes (Phase 2).
- Tracked changes: export as final (default) / show markup (Phase 2).
- Chapter starts on new page; on odd page (book interior).

### 4.3 Requirements

- **Uses the same layout engine as the page view** (see 05 §3) so the PDF matches the screen.
- Selectable, searchable text; fonts embedded.
- 100,000-word compiled manuscript exports in < 60 s.

## 5. `.quill` planning files

- UTF-8 JSON, pretty-printed (2-space indent) so they diff well in git and are readable in Notepad.
- Every file has a header:
  ```json
  { "quill": "1.0", "type": "snowflake", "id": "c0f1…", "title": "Snowflake", ... }
  ```
  `type` selects the editor: `snowflake`, `beats` (Save the Cat, Three-Act, etc.),
  `character`, `scenes`, `location` (Phase 3).
- Validated with zod on load. Unknown fields are preserved on save (forward compatibility).
- If a `.quill` file is invalid JSON (e.g. hand-edited badly), open it in the plain-text
  editor with an error banner instead of failing.
- Cross-file references (scene → document, outline → character) use paths relative to the
  project root plus the target's stable `id`, so renames can be re-linked.
- Windows file association: double-clicking a `.quill` file opens Quill with its project.

## 6. Test corpus

`tests/fixtures/docx/`: Word-authored manuscript; Google Docs export; LibreOffice export;
Scrivener compile output; files with tables, merged cells, comments with replies, resolved
comments, tracked insertions/deletions/format changes from multiple authors, footnotes,
text boxes, equations; malformed zip.

Golden tests:
1. Import → export (no edits) → re-import yields an identical document model.
2. Import → export (no edits): every Tier B/C XML part is byte-equivalent after XML canonicalization.
3. Exported files pass OOXML schema validation and open in LibreOffice headless without errors (CI).
