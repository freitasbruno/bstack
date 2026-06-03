# memory-viewer
Single-file HTML/JS browser for `.claude/memory/` files. Opens in Chrome/Edge with no server or build step.

## What it does
Opens `.claude/memory-viewer.html` in a Chromium browser. On first load, prompts the user to select their project's `.claude/memory/` folder via the File System Access API. On subsequent loads, restores the folder silently from IndexedDB (or guides re-grant with `startIn` if permission expired). Renders the file tree in a left sidebar with collapsible folders and pinned quick links. Clicking a file renders it in the main pane via the active module.

## Location
`.claude/memory-viewer.html` — single self-contained file, no dependencies except Google Fonts CDN for Inter and JetBrains Mono (graceful fallback to system fonts when offline).

## Architecture

### Module system
`ModuleRegistry` dispatches to the first matching module by file path. Dispatch order (first match wins):
1. `MockupRenderer` — `mockups/**` (stub, Phase 2)
2. `SchemaVisualizer` — `schema/**/*.md` (stub, Phase 2)
3. `MarkdownRenderer` — all `.md` files (active)

Module interface: `{ name, canHandle(fh, relPath), render(fh) → Promise<HTMLString> }`

### Core modules
| Module | Purpose |
|--------|---------|
| `DB` | IndexedDB IIFE — `save(handle)` / `load()` for folder persistence |
| `MD` | Inline markdown parser — no dependencies, handles all common syntax |
| `ModuleRegistry` | `register(module)` / `dispatch(fh, relPath)` |
| `FileTree` | Async recursive directory renderer with expand/collapse state |
| `QuickLinks` | Pinned sidebar links: Roadmap, WIP, Notes, Known Bugs, Tech Debt |
| `UI` | DOM helpers: open/content state, header path, breadcrumb, module pill, active states |
| `Core` | App init, permission restore, openFile/openDir/openByPath/openByRelPath, refresh |

### Markdown renderer supports
Headers h1–h6, bold, italic, strikethrough, inline code, fenced code blocks, tables (with column alignment), ordered/unordered lists, checkboxes (`- [ ]` / `- [x]`), blockquotes, horizontal rules, relative links (navigable), external links (new tab).

## Design system
Adapted from X.Plorers: warm cream background (`#f5f0e8`), paper grain overlay, rose accent (`#f43f5e`), Inter font, card surfaces, fadeUp animations.

## Known limitations
- Multiline blockquotes: inline formatting applied as a single joined string — not an issue for BStack files
- YAML frontmatter `---` fences render as `<hr>` — memory files don't use frontmatter
- Google Fonts CDN required for correct font rendering; offline falls back to system sans-serif

## Next modules (roadmap)
- `schema-visualizer` — Mermaid diagram rendering for `schema/**` files
- `mockup-renderer` — iframe rendering for `mockups/**/*.html` files
