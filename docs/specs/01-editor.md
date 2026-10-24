# 01 — Editor (Documents)

The document editor is modeled on **Google Docs**: a simple, uncluttered writing surface
on real pages. v1 deliberately covers core word processing well; richer features are
listed per phase so the architecture leaves room for them.

## 1. Layout

```
┌───────────────────────────────────────────────────────────────────────┐
│ Menu bar  (File  Edit  View  Insert  Format  Tools  Help)             │
│ Single-row toolbar: undo redo │ zoom │ style ▾ │ font ▾ │ size │      │
│   B I U  color highlight │ link comment │ align │ spacing │ lists     │
├────────────┬──────────────────────────────────────────┬───────────────┤
│ Explorer   │ [01 Arrival.docx] [Snowflake.quill •]    │ Comments      │
│ (file tree │ ┌ruler─────────────────────────────────┐ │ (margin       │
│  — see 03) │ │  ┌──────────────┐                    │ │  threads,     │
│            │ │  │   page 1     │   grey canvas      │ │  aligned to   │
│ Doc outline│ │  │              │                    │ │  their text)  │
│ (headings) │ │  └──────────────┘                    │ │               │
│            │ │  ┌──────────────┐                    │ │               │
│            │ │  │   page 2     │                    │ │               │
├────────────┴──────────────────────────────────────────┴───────────────┤
│ Status: page 3 of 12 · 2,431 words (doc) · 61,204 (project) · Saved ✓ │
└───────────────────────────────────────────────────────────────────────┘
```

- Left sidebar has two sections: **Explorer** (project file tree) and **Document outline**
  (headings in the current document, click to jump — like Google Docs' outline).
- Right side: **comments margin**, shown when the document has comments (collapsible).
- All panels collapsible/resizable; layout persists per project.
- **Focus mode** (Phase 2): hide all chrome except the pages.

## 2. Page view (v1)

- White pages with drop shadow on a grey canvas, centered; page size and margins come from
  the document (default Letter, 1" margins; A4 by locale).
- Text flows across pages; paragraphs split across page boundaries at line level.
- Honors page breaks, section breaks (with per-section page size/orientation/margins),
  keep-with-next, keep-lines-together, widow/orphan control.
- Headers and footers rendered on every page, including page numbers. Editable in v1 by
  double-clicking the header/footer area (text + page number field + different first page).
- Horizontal ruler showing margins and paragraph indents (drag to adjust indents — Phase 2).
- Zoom 50–200% (toolbar + Ctrl+scroll). "Fit width" option.
- Typing must stay smooth: < 16 ms per keystroke in a 15,000-word / ~60-page document.
- **What you see is what you export**: PDF output pagination must match the page view
  (same line breaks and page breaks).

Later: pageless mode toggle (Open Question 1), two-pages-side-by-side view.

## 3. Feature tiers

### v1 — core word processing

| Area | Features |
|------|----------|
| Character | Bold, italic, underline, strikethrough, font family, font size, text color, highlight, clear formatting (Ctrl+\\) |
| Paragraph | Align left/center/right/justify; line spacing (single, 1.15, 1.5, double, custom); space before/after; first-line indent; left indent (increase/decrease) |
| Styles | Normal text, Title, Subtitle, Heading 1–3, plus any styles imported from the `.docx`. "Update style to match" like Google Docs. |
| Lists | Bulleted and numbered, multi-level (Tab/Shift+Tab) |
| Insert | Page break, scene break (configurable `#` / `* * *` / ornament), link, image (inline, from file or paste), table, special characters |
| Tables | Insert table, add/remove rows/columns, edit cell text, basic cell borders/shading preserved from imported files |
| Comments | Add comment on a selection (Ctrl+Alt+M), reply, edit, resolve/reopen, delete; comments sidebar aligned to text |
| Tracked changes | Display insertions/deletions/formatting changes from imported files with author colors; accept/reject individually or all; "show/hide changes" toggle |
| Editing | Unlimited undo/redo per open tab (kept across tab switches); paste with formatting / Ctrl+Shift+V plain; find & replace in document (match case, whole word) |
| Typography | Smart quotes, `--` → em dash, `...` → ellipsis (toggleable) |
| Proofing | Spell check with a project dictionary (character/place names) |
| Counts | Word count for document, selection, and project in the status bar; Tools → Word count dialog (words, characters, pages) |

### Phase 2

Suggesting mode (author creates tracked changes — Open Question 2), footnotes, find & replace
across project, regex search, ruler drag-to-indent, hanging indents, small caps,
super/subscript, table cell merge/split and column resizing, document word targets,
session word goals, focus mode, typewriter scrolling, keyboard shortcut customization.

### Phase 3 and later

Pageless mode, version history view (built on backups), writing statistics dashboard,
columns, image text wrapping, dictionary/thesaurus, read-aloud.

## 4. Keyboard shortcuts

Use Google Docs' defaults (which mostly match Word): Ctrl+B/I/U, Ctrl+Shift+L/E/R/J for
alignment, Ctrl+Alt+1..3 headings, Ctrl+Alt+0 normal text, Ctrl+Enter page break,
Ctrl+Alt+M comment, Ctrl+K link, Ctrl+F / Ctrl+H find & replace, Ctrl+Shift+C word count,
Ctrl+P print/export PDF. Workspace shortcuts are in 03 §3.

## 5. Style presets (project level)

Applied via Format → Document style preset, and usable at export time:
- **Standard Manuscript Format** — 12pt Times New Roman, double spaced, 0.5" first-line
  indent, 1" margins, header "Surname / TITLE / page".
- **Book interior** — 6"×9", serif 11pt, justified, mirror margins (Phase 2).
- **Comfortable screen** — larger font, 1.5 spacing, for drafting.
