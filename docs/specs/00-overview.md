# Quill — Product Overview

> Status: Draft v0.2 — 2026-10-23

## 1. Vision

A desktop word processor built for novelists. It should feel like **Google Docs** to write
in — clean single toolbar, white pages on a grey canvas, comments in the margin — but be
organized like a **code editor (e.g. VS Code)**: you open a project folder, everything in
it (chapters, scenes, outlines, character sheets, notes) shows up in one file tree, and you
click between files and tabs freely. Built-in outlining tools (Snowflake Method and other
popular structures) are simply another kind of file in that tree.

## 2. Goals

| # | Goal | Spec |
|---|------|------|
| G1 | Google Docs-style rich-text editing on real pages | [01-editor.md](01-editor.md) |
| G2 | Import and export `.docx` without destroying tables, comments, tracked changes or formatting; export `.pdf` | [02-file-formats.md](02-file-formats.md) |
| G3 | Text-editor-style workspace: one file tree, tabs, instant switching between chapters and outlines | [03-workspace.md](03-workspace.md) |
| G4 | Outline novels with Snowflake Method and other structures, stored as files in the project | [04-outlining.md](04-outlining.md) |
| — | Technical architecture | [05-architecture.md](05-architecture.md) |
| — | Phased roadmap & acceptance criteria | [06-roadmap.md](06-roadmap.md) |

## 3. Non-goals

- AI writing or outlining assistance (decided: not wanted).
- Real-time collaboration / multi-user editing.
- Cloud sync (users can put the project folder in OneDrive/Dropbox/Google Drive themselves).
- Full Word parity (SmartArt, macros, equations, text boxes) — these are *preserved*, not edited (see 02).
- Mobile apps.

## 4. Target user

A single author writing long-form fiction (60k–150k+ words), on Windows first, who:
- Already has chapters as `.docx` files — sometimes with tables, comments and tracked
  changes from editors/beta readers — or needs to deliver `.docx` to agents/editors.
- Wants a standard manuscript-format PDF/DOCX for submissions.
- Plans stories with a structured method before or during drafting.

## 5. Core concepts

- **Project** — a folder on disk. Everything about a book lives inside it, visibly.
- **File** — anything in the project tree. Quill knows how to open several kinds:
  - **Documents** (`.docx`) — chapters, scenes, notes. Whether a file is a chapter or a
    scene is entirely the author's choice; Quill doesn't care.
  - **Planning files** (`.quill`) — Snowflake outlines, beat sheets, character sheets,
    scene lists. Human-readable JSON, opened in purpose-built editors.
  - **Plain text** (`.txt`, `.md`) — opened in a simple editor.
- **Manuscript order** — the ordered list of documents that make up the book, used for
  full-book export ("compile").

## 6. Key decisions

| ID | Decision | Choice | Rationale |
|----|----------|--------|-----------|
| D1 | Platform | Desktop app (Electron), Windows first | Real folder access, file watching, native dialogs, PDF printing. |
| D2 | Where things live | **Everything the author writes is a visible file in the project tree** — chapters as `.docx`, outlines/characters/scenes as `.quill` files. Only machine bookkeeping (open tabs, backups, crash recovery) goes in a hidden `.quill/` folder. | Outlines are as much "the book" as chapters; they should be browsable, switchable, renameable, and backed up the same way. |
| D3 | UI model | Google Docs for the editing surface; VS Code for the workspace (tree + tabs + quick open) | Author preference. |
| D4 | Pages in v1 | **Paginated page view is required in v1** | Author wants to see real formatted pages while writing. |
| D5 | DOCX fidelity | Tables, comments, tracked changes and images must survive round-trips in v1; unknown content is preserved opaquely, never dropped | Author's existing files rely on them. |
| D6 | Editor engine | TipTap (ProseMirror) | Mature, extensible, handles long documents; pagination approach decided in a Phase 0 spike (05 §3). |

## 7. Resolved questions (from v0.1)

| Question | Answer |
|----------|--------|
| AI assistance? | No. |
| One file per chapter or per scene? | Author's choice; both must work equally well. |
| Paginated view in v1? | Yes — Google Docs-style pages. First version can be modest in features otherwise. |
| Heavy `.docx` features? | Yes, tables/comments/tracked changes matter. |
| Name? | Quill (for now). |

## 8. Open questions

1. **Pageless mode** — Google Docs also offers a "pageless" continuous view. Want it as a toggle (later phase)?
2. **Suggesting mode** — v1 will display, accept and reject tracked changes from other people. Do you also want to *make* tracked changes yourself (Google Docs "Suggesting" mode) soon, or is that a later nicety?
