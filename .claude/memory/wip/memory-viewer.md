# memory-viewer
Single-file HTML/JS memory browser with inline markdown rendering and pluggable module architecture.

## Status
shipping

## Requirements
- Single `.claude/memory-viewer.html`, opens in Chrome/Edge (no server, no build step)
- File System Access API — folder picker on first open, permission persisted via IndexedDB
- Left sidebar: full file tree of `memory/` with collapsible folders
- Pinned quick links: roadmap/index.md, wip/ files, notes.md, known-bugs.md, technical-debt.md
- Main pane: rendered markdown (inline renderer, no CDN dependencies)
- Manual refresh button (no auto-watch)
- Pluggable module architecture: ModuleRegistry dispatches to renderer by file path/type
- Phase 1: MarkdownRenderer active; SchemaVisualizer and MockupRenderer exist as stubs

**Out of scope (Phase 1):**
- Mermaid rendering (SchemaVisualizer stub only)
- Iframe mockup rendering (MockupRenderer stub only)
- Auto-reload / file watching
- Editing

## Design reference
.claude/memory/mockups/memory-viewer/mockup.html

## Technical decisions

### Architecture
```
memory-viewer.html
├── <style>        — layout + markdown CSS (X.Plorers design system adapted)
├── <div id="app"> — header / sidebar / main pane
└── <script>
    ├── Core           — FileSystemAccess, IndexedDB persistence, state, events
    ├── ModuleRegistry — register() / dispatch(fileHandle) → module
    ├── FileTree       — render sidebar tree, expand/collapse
    ├── QuickLinks     — pinned navigation section
    ├── MarkdownRenderer  (active)  — inline renderer
    ├── SchemaVisualizer  (stub)    — falls through to MarkdownRenderer
    └── MockupRenderer    (stub)    — falls through to MarkdownRenderer
```

### Module interface
```js
{
  name: 'markdown',
  canHandle: (fileHandle) => Boolean,
  render: async (fileHandle) => HTMLString
}
```

### Module dispatch order (first match wins)
1. MockupRenderer    — `.claude/memory/mockups/**`
2. SchemaVisualizer  — `.claude/memory/schema/**`
3. MarkdownRenderer  — everything else (default)

### Inline markdown renderer handles
- Headers h1–h6, bold, italic, strikethrough
- Tables (critical — roadmap and registry use them)
- Fenced code blocks with language class
- Ordered and unordered lists
- Blockquotes
- Horizontal rules
- Relative links (navigable within granted folder)

### Persistence
IndexedDB stores the directory handle. On next session: `queryPermission()` → if granted, restore silently; if not, show folder picker.

### Design system
Adapted from X.Plorers: warm cream bg (#f5f0e8), paper grain overlay, rose accent (#f43f5e), Inter font, card surfaces, fadeUp animations.

## Test plan

### Automated
none — pure client-side HTML, no framework

### Human verification required
- [ ] Open memory-viewer.html in Chrome → folder picker appears
- [ ] Grant access to .claude/memory/ → file tree renders correctly
- [ ] Close and reopen → permission persists, no re-grant needed
- [ ] Click a .md file → markdown renders (headers, tables, code, lists)
- [ ] Click each quick link → correct file loads
- [ ] Click refresh → file re-reads from disk
- [ ] Click a schema file → renders as markdown (stub, no Mermaid)
- [ ] Navigate nested folders → tree expands/collapses correctly
- [ ] Relative link in a markdown file → navigates to that file

## Open issues
- `/update` skill should also fetch `memory-viewer.html` — flag for follow-up /evolve after ship
- CLAUDE.md memory index updated: `mockups/index.md` replaces `wip/mockups/index.md`

## Session log
- 2026-06-03 16:33 Plan approved
- 2026-06-03 16:45 Completed: .claude/memory-viewer.html — full implementation (Core, DB, ModuleRegistry, FileTree, QuickLinks, MarkdownRenderer, SchemaVisualizer stub, MockupRenderer stub)
- 2026-06-03 16:45 Pending: /review
- 2026-06-03 16:50 Review complete: fixed HTML injection in FileTree.makeRow, openDir dir-entries, UI.setBreadcrumb, and inline() link URL quoting. MemoryViewer added to component registry.
- 2026-06-03 16:55 Verify passed: no tsc/lint/tests (pure HTML, no build step). Smells: clean. TODOs: 0.
