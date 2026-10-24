# Quill

Desktop word processor for novelists: Google Docs-style paged editor, VS Code-style
workspace (file tree + tabs), `.docx` import/export, PDF export, Snowflake/beat-sheet
outlining stored as `.quill` files. Electron + TypeScript + React + TipTap.

**Specs live in [docs/specs/](docs/specs/README.md). Read the relevant spec before
implementing a feature.** If code and spec disagree, ask — don't silently diverge. If a
decision changes, update the spec in the same change.

## Core principle: the least code that fully works

The goal is production-ready code with the smallest reasonable footprint.

- Write the simplest code that meets the spec and its acceptance criteria. Nothing speculative.
- No abstractions until there are at least two real uses. Three similar lines beat a premature helper.
- No interfaces/base classes/factories/plugin systems "for later". The only planned extension
  points are the editor registry (05 §2) and outline templates (04 §7).
- Don't build features from a later phase than the one being worked on (see [06-roadmap.md](docs/specs/06-roadmap.md)).
- Prefer the platform and existing dependencies (Electron, Node, TipTap/ProseMirror, React)
  over new code; prefer a small focused library over writing a complex one ourselves —
  but don't add a dependency for something that's a few lines.
- Every new dependency needs a one-line justification in the PR/commit message. No duplicate-purpose libraries.
- Delete dead code, unused exports, and commented-out code. Don't leave TODO scaffolding.
- Before finishing a change, re-read the diff and remove anything not needed.

## Production-ready means

- **Never lose user data.** Follow the reliability rules in [05-architecture.md §6](docs/specs/05-architecture.md):
  atomic writes, backup before overwrite, trash instead of delete, never drop unknown DOCX content.
- Handle real failure cases (locked files, invalid JSON, malformed `.docx`, disk full) with
  clear user-facing messages. Don't add handling for impossible cases.
- No `any`, no non-null `!` without a reason, no swallowed errors. TypeScript `strict` on.
- Validate all data crossing a trust boundary (IPC, files on disk) with zod. Trust internal code.
- Electron security: `contextIsolation: true`, `nodeIntegration: false`, `sandbox: true`;
  renderer only uses the typed preload API; all paths validated to be inside the open project.
- Keep the UI responsive: DOCX parsing/serializing and project-wide work run off the UI thread.
- Meet the performance budgets in the specs (e.g. < 16 ms typing, < 50 ms tab switch).

## Code style

- Small, flat modules organized by feature (see layout below). Functions over classes unless state demands a class.
- Name things for what they are in the domain (document, chapter, scene, outline, beat).
- Comments explain *why*, not *what*. No boilerplate doc comments.
- Match the surrounding code's patterns before introducing new ones.
- Formatting/linting via Prettier + ESLint; don't hand-format.

## Layout

Single app, single `package.json` — no monorepo. Split into packages only if a real need appears.

```
src/main/        Electron main process (files, watcher, export, IPC handlers)
src/preload/     typed contextBridge API
src/renderer/    React UI: workbench/, editors/<type>/
src/shared/      types + zod schemas shared across processes
src/docx/        OOXML <-> document model (pure TS, no Electron imports)
tests/           fixtures/docx, e2e
docs/specs/      specs; docs/decisions/ for recorded decisions
```

## Testing

- Test behavior that matters: DOCX round-trip golden tests (02 §6), file safety, outline data,
  and E2E for each acceptance criterion. Don't write tests for trivial code or mock-heavy unit tests.
- Every bug fix gets a regression test.
- A change is done when `npm run typecheck`, `npm run lint`, and `npm test` pass.
  For UI changes, also run the app and check the change actually works.

## Workflow

- Work in small, reviewable commits that each leave the app working.
- Don't commit unless asked. Never commit secrets or large binaries (fixtures excepted).
- When something in the spec looks over-engineered for its phase, say so and propose the simpler version.
- Don't download any new technologies without approval
