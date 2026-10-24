# 04 — Outlining

Outlining tools help plan the novel using established methods. **Every outline is a file in
the project tree** (`.quill`, see 02 §5) — opened in a tab, switchable with Ctrl+P/Ctrl+Tab,
viewable side by side with a chapter in split view, renameable, movable, and backed up like
any other file.

## 1. Principles

- **Files, not a separate mode.** There is no "outline mode"; there are outline files.
- **Method-driven, data-defined.** Each method is a template (steps, fields, guidance text).
  New methods can be added without code changes (§7).
- **Linked to the manuscript.** Scenes link to documents; characters are referenced from outlines.
- **Iterative.** Expansion steps (Snowflake) show the previous step beside the current one.
- **Non-blocking.** Steps can be done in any order, skipped, or left partial.

## 2. Planning file types

| Type | Example file | Editor | Phase |
|------|--------------|--------|-------|
| `snowflake` | `Planning/Snowflake.quill` | Step-by-step guided editor (§3) | 1 |
| `scenes` | `Planning/Scenes.quill` | Spreadsheet table + corkboard (§5) | 1 |
| `character` | `Characters/Mara Quinn.quill` | Character sheet form (§6) | 1 |
| `beats` | `Planning/Save the Cat.quill` | Beat sheet + timeline (§4) | 2 |
| `location` | `World/Harbor Town.quill` | Simple form | 3 |

New-file menu: **New → Outline → Snowflake / Save the Cat / Three-Act / …**, **New → Character**, **New → Scene list**.

Every planning file has a **"Export as document"** action → `.docx` or `.pdf`
(e.g. a formatted synopsis or character bible).

## 3. Snowflake Method (v1 flagship)

Based on Randy Ingermanson's 10-step method. The Snowflake editor shows a step navigator on
the left (with per-step completion: empty / started / done) and the current step on the right.

| Step | Name | Input UI | Guidance / constraints |
|------|------|----------|------------------------|
| 1 | One-sentence summary | Single text field | Live word counter, target ≈ 15 words; tip to avoid character names. |
| 2 | One-paragraph summary | Five labeled fields: Setup, Disaster 1, Disaster 2, Disaster 3, Ending | Step 1 shown above. Toggle to read as one paragraph. |
| 3 | Character summaries | List of characters; per character: Storyline (one sentence), Motivation, Goal, Conflict, Epiphany, Storyline (one paragraph) | Each entry **is a character file** in `Characters/` — creating one here creates the file; editing here or in the character file edits the same data. |
| 4 | One-page synopsis | Each step-2 sentence expanded into a paragraph | Side by side: sentence → paragraph. |
| 5 | Character synopses | One page per major character, the story from their POV | Stored in the character file. Step-3 data shown for reference. |
| 6 | Four-page synopsis | Each step-4 paragraph expanded to ~a page | Side by side with step 4. Rich text (bold/italic). |
| 7 | Character charts | Full character details (see §6) | Opens/embeds the character files. |
| 8 | Scene list | Links to (or creates) a `scenes` file | "Seed from synopsis": one scene row per step-6 paragraph. |
| 9 | Scene descriptions | Per-scene narrative description and scene/sequel fields | Stored in the scenes file. Optional step. |
| 10 | First draft | **Create documents from scene list** | Dialog: one document per scene or per chapter (grouped by the scene's Chapter column), target folder, naming pattern (`{chapter:00} {title}`). Creates `.docx` files with a heading and links them to their scenes. |

The Snowflake file links to its characters and scene list by path + id.

## 4. Beat-sheet methods (Phase 2)

Same engine: an ordered list of beats, each with name, guidance, recommended position
(% through the book), and fields; scenes can be assigned to beats.

| Method | Beats |
|--------|-------|
| Three-Act Structure | Setup, Inciting Incident, Plot Point 1 (~25%), Rising Action, Midpoint (~50%), Plot Point 2 (~75%), Climax, Resolution |
| Save the Cat! (Snyder; novel adaptation by Brody) | Opening Image, Theme Stated, Setup, Catalyst, Debate, Break into Two, B Story, Fun and Games, Midpoint, Bad Guys Close In, All Is Lost, Dark Night of the Soul, Break into Three, Finale, Final Image |
| Hero's Journey (Vogler) — Phase 3 | Ordinary World, Call to Adventure, Refusal of the Call, Meeting the Mentor, Crossing the Threshold, Tests/Allies/Enemies, Approach to the Inmost Cave, Ordeal, Reward, The Road Back, Resurrection, Return with the Elixir |
| Seven-Point Structure (Wells) — Phase 3 | Hook, Plot Turn 1, Pinch Point 1, Midpoint, Pinch Point 2, Plot Turn 2, Resolution |
| Story Circle (Harmon) — Phase 3 | You, Need, Go, Search, Find, Take, Return, Change |
| Custom | User-defined beats |

**Beat timeline**: a bar representing the book's target length with beat markers at their
recommended %; assigned scenes are placed by actual position computed from linked
documents' word counts — e.g. *"Midpoint lands at 63% (recommended ~50%)"*.

A project can contain any number of outline files, including several methods at once.

## 5. Scene list (`scenes`)

- **Table view**: rows = scenes; columns: #, Title, Summary, POV, Characters, Location,
  Chapter, Plotline(s), Beat(s), Linked document, Status, Est. words, Actual words, Notes.
  Sortable, filterable, column show/hide, drag rows to reorder.
- **Corkboard view** (Phase 2): index cards in a grid, colored by POV/plotline/status, drag to reorder.
- **Linking**: each scene can link to a document. Clicking the link opens the document in a tab
  (or the other split group). Actual words and status update from the linked document.
- Unlinked scenes and manuscript documents not linked to any scene are flagged.
- "Reorder manuscript to match scenes" action (optional, never automatic).
- Plotlines are defined in the scenes file (name + color).

## 6. Character files (`character`)

One file per character so they appear in the tree and can be opened in tabs.

Fields: name, aliases, role (protagonist/antagonist/supporting/…), one-sentence storyline,
motivation, goal, conflict, epiphany, one-paragraph storyline, character synopsis
(Snowflake step 5), physical description, background, personality, relationships
(links to other character files), voice/speech notes, arc, custom fields, free notes.
Image (Phase 2).

Character names and aliases are automatically added to the project spell-check dictionary.

## 7. Method template format

Built-in templates ship with the app; user templates live in `%APPDATA%/Quill/templates/`
and can be shared as files.

```json
{
  "id": "save-the-cat",
  "schemaVersion": 1,
  "name": "Save the Cat! Beat Sheet",
  "kind": "beats",
  "steps": [
    {
      "id": "catalyst",
      "name": "Catalyst",
      "position": { "percent": 10 },
      "guidance": "The event that knocks the hero's life off course...",
      "fields": [{ "id": "summary", "label": "What happens", "type": "longtext" }]
    }
  ]
}
```

Field types: `text`, `longtext`, `richtext`, `number`, `select`, `characterRef`, `sceneRef`,
`list` (repeatable group). Snowflake uses `kind: "steps"` with `sourceStep` references for
the side-by-side expansion views. A new outline file stores its template id and a copy of
the template version it was created from, so later template changes never break existing files.

Guidance text must be original paraphrase (not quoted from the methods' books).
