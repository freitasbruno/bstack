# BStack

A skill-based harness for [Claude Code](https://claude.com/claude-code) that turns
"ask Claude to build a feature" into a disciplined, repeatable pipeline with a
persistent project memory.

BStack gives every project:

- **A fixed delivery pipeline** — `/plan` → `/build` → `/review` → `/verify` → `/ship`.
  Each step is a Claude Code skill with hard rules, entry checks and a defined hand-off.
- **A memory layer** in `.claude/memory/` — roadmap, schema, component registry,
  patterns, known bugs, tech debt, test standards — that Claude reads selectively
  and keeps up to date as features ship.
- **A self-improvement loop** — `/reflect` mines past work for recurring friction,
  `/evolve` turns that into concrete changes to the skills themselves.
- **One upstream for all projects** — `/update` pulls the latest skills from this
  repo into any project without touching its memory or `CLAUDE.md`.

---

## Contents

- [How it works](#how-it-works)
- [Getting started](#getting-started)
- [The pipeline](#the-pipeline)
- [Utility and setup skills](#utility-and-setup-skills)
- [The memory layer](#the-memory-layer)
- [Rules every skill follows](#rules-every-skill-follows)
- [Keeping projects up to date](#keeping-projects-up-to-date)
- [Evolving the harness](#evolving-the-harness)
- [Repository layout](#repository-layout)
- [Assumptions and limitations](#assumptions-and-limitations)

---

## How it works

Claude Code loads skills from `.claude/skills/<name>/SKILL.md` and project
instructions from `CLAUDE.md`. BStack is a set of those skills plus a template
`CLAUDE.md` and memory skeleton.

Work on a feature moves through a single **wip file**
(`.claude/memory/wip/<feature-slug>.md`). `/plan` creates it, every later step reads
it, checks its `Status`, appends to its session log and advances it:

```
planning ──/build──▶ building ──▶ reviewing ──/verify──▶ shipping ──/ship──▶ archived
                         ▲                         │
                         └──── any check fails ────┘
```

Because the plan, decisions and progress live in a file rather than in the chat,
a feature can be paused and resumed across sessions — `/build` reads the session
log and picks up where it left off.

Every lifecycle skill also writes `START`/`END` lines to `.claude/audit.log`, which
`/reflect` later uses as evidence.

---

## Getting started

### Prerequisites

- [Claude Code](https://docs.claude.com/en/docs/claude-code) (CLI, desktop or IDE extension)
- `git`
- Windows + PowerShell for `/update` as currently written (see
  [limitations](#assumptions-and-limitations))
- For the default `/verify` gate: a Node/TypeScript project with `npx tsc`,
  `npm run lint` and `npm test`

### New project

1. Create the project folder and copy this repo's `.claude/skills/`,
   `.claude/memory-viewer.html` and `CLAUDE.md` into it:

   ```bash
   git clone --depth 1 https://github.com/freitasbruno/bstack /tmp/bstack
   mkdir -p my-project/.claude
   cp -r /tmp/bstack/.claude/skills /tmp/bstack/.claude/memory-viewer.html my-project/.claude/
   cp /tmp/bstack/CLAUDE.md my-project/
   ```

2. Open Claude Code in `my-project/` and run **`/new`**.
   It walks you through identity, stack, environments, roadmap, schema and
   design system one question at a time, then scaffolds the memory tree,
   writes `.claude/settings.json` permissions and fills in `CLAUDE.md`.

3. Run **`/plan`** to start your first feature.

### Existing project

Copy the same files into the existing repo and run **`/import`** instead of
`/new`. It reads the codebase and/or any documents you point it at (vision docs,
user stories, ERDs, design files), asks about gaps one at a time, and populates
the memory layer. It never modifies code. It is safe to re-run to merge in new
information, and it reports findings (dead code, duplicate components, missing
tests, security surfaces) into `known-bugs.md` and `technical-debt.md`.

> The `CLAUDE.md` in this repo is a placeholder with a
> "SETUP INCOMPLETE" banner. `/new` or `/import` replaces it.

---

## The pipeline

Never skip a step. Each skill checks the wip status and refuses to run out of order.

| Skill | Role | What it does | Never does |
|-------|------|--------------|------------|
| **`/plan`** | Senior technical partner | Situates the feature in the roadmap, challenges assumptions and edge cases one question at a time, locks requirements (what it does *and doesn't* do), optionally produces a UI brief/mockup, evaluates DB/components/patterns, defines a test plan, then writes the wip file. | Write the wip file without explicit approval. |
| **`/build`** | Disciplined implementer | Builds exactly the approved plan in a fixed order: migrations → types → services → hooks → components → routes → tests. Checks the component registry before creating anything. Runs `tsc` after each logical unit and logs progress to the wip. | Make architectural decisions (it stops and asks), fix unrelated bugs, refactor opportunistically, run git. |
| **`/review`** | Critical reviewer | Audits correctness against requirements, dead code and dead tables, component duplication, convention drift, and known bugs. Fixes trivial issues directly; flags anything with consequences. Produces a minimal manual test checklist. | Rubber-stamp. |
| **`/verify`** | CI gate | Runs TypeScript, lint (zero warnings), tests, a smell scan (`console.log`, `localhost:`, merge markers) and a TODO/FIXME scan. Any failure sends the work back to `/build`. | Change anything. Ever. |
| **`/ship`** | Release + memory keeper | Rewrites the feature's roadmap file to describe what *actually* shipped, updates roadmap/schema/registries/known-bugs, archives the wip, shows staged files, proposes a conventional commit message, commits and pushes. | Auto-stage everything or commit without confirmation. |

---

## Utility and setup skills

| Skill | When | What it does |
|-------|------|--------------|
| **`/new`** | Once, new project | Guided setup conversation → full memory layer, scaffolded folders, permissions, `CLAUDE.md`. |
| **`/import`** | Once (re-runnable), existing project | Reads code and documents → populates memory. Read-only on code. |
| **`/status`** | Any time | Lists in-progress wip files with their last log entry and renders the roadmap table. No commentary. |
| **`/note [topic]`** | Any time, mid-flow | Turns the current context into a dated, self-contained note appended to `notes.md`. One-line confirmation, no questions. Notes surface in `/plan`, `/reflect` and `/evolve`. |
| **`/reflect`** | End of project or milestone | Reads archived wips, audit log, bugs, debt and roadmap. Names patterns (one incident is noise, two a signal, three a pattern), confirms the diagnosis with you, then writes prioritised proposals. Implements nothing. |
| **`/evolve`** | After `/reflect`, or on demand | Implements one specific harness change: shows the diff, gets confirmation, edits the skill files, and writes a dated record to `evolve/` describing what consumer projects need to apply. |
| **`/update`** | Whenever upstream changes | Pulls the latest skills from this repo into the current project (see below). |

---

## The memory layer

Everything Claude needs to know about a project lives in `.claude/memory/`.
`CLAUDE.md` contains an index telling Claude which file to load for which task,
so it only reads what the current step needs.

| File | Holds | Loaded when |
|------|-------|-------------|
| `techstack.md` | Stack decisions and dependencies | Adding a dependency, debugging builds |
| `environments.md` | Repos, branches, environments, deploy targets | Shipping, deployment questions |
| `schema/index.md`, `schema/erd-<domain>.md` | Tables grouped by domain, Mermaid ERDs | Any feature touching data |
| `roadmap/index.md`, `roadmap/<feature>.md` | Feature list + status; per-feature "as built" description | Every planning session |
| `registry/components.md` | Reusable UI components | Any feature with UI |
| `registry/patterns.md` | Service, auth and storage patterns | New service or auth surface |
| `registry/utils.md` | Helper functions | Before writing any helper |
| `tests/standards.md` | Harness-wide testing standards (fixed) | Plan and verify |
| `known-bugs.md`, `technical-debt.md` | Open bugs and debt | Planning, review |
| `mockups/index.md` | Registered mockups and design briefs | Any feature with UI |
| `notes.md` | Append-only `/note` captures | `/plan`, `/reflect`, `/evolve` |
| `harness.md` | Log of `/update` runs | Before `/update` |
| `wip/<feature>.md` | The in-flight feature (status, requirements, decisions, test plan, session log) | Every pipeline step |
| `wip/_archive/` | Completed wips | `/reflect` |

### Memory viewer

`.claude/memory-viewer.html` is a single-file browser for the memory tree. Open it
in a Chromium-based browser and pick your project's `.claude/memory` folder (it uses
the File System Access API, so nothing is uploaded anywhere).

---

## Rules every skill follows

These are written into the template `CLAUDE.md` and enforced by the skills:

- **Back up before writing memory.** Before changing any memory file (except
  `techstack.md`, `environments.md` and the append-only `notes.md`), copy it to
  `_backup/<file>.YYYY-MM-DD-HHhmm.md` in the same folder. Backups and wip archives
  are gitignored.
- **Memory is autonomous.** Claude never asks permission to read or write
  `.claude/memory/`. `/new` writes matching allow rules into `.claude/settings.json`
  so these operations don't trigger prompts.
- **Native file access only.** Skills use Claude Code's Read/Edit/Write tools for
  files and never shell out for reads or timestamps (shell substitutions like
  `$(date …)` trip Claude Code's command-injection detector). Shell is reserved
  for git, npm/npx, creating directories and copying skill files.
- **Mockups are never left in chat.** Any brief, wireframe or sketch is saved under
  `wip/mockups/<feature>/` and registered in the mockups index.

---

## Keeping projects up to date

This repo is the upstream for every BStack project. In a project, run **`/update`**:

1. Shallow-clones `github.com/freitasbruno/bstack` to a temp folder.
2. Compares every file under `.claude/skills/` (and `memory-viewer.html`) by hash.
3. Lists only what changed and asks once before overwriting.
4. Logs the date, commit and changed files in `.claude/memory/harness.md`.

It **never** touches `CLAUDE.md`, memory files or settings. A project can keep its
own local skills alongside the BStack ones; `/update` only overwrites files that
exist upstream.

To change behaviour for all projects: edit `.claude/skills/` here, push, then run
`/update` in each project.

---

## Evolving the harness

```
work on features ──▶ /note friction as it happens
        │
        ▼
   /reflect  ──▶ patterns + prioritised proposals
        │
        ▼
   /evolve   ──▶ skill edits here + evolve/YYYY-MM-DD_<change>.md
        │
        ▼
   push ──▶ /update in each project
```

The `evolve/` folder is the changelog of the harness: each file records the
problem, the root cause, which skills changed and any manual steps a consumer
project needs (for example, permission rules to add to its settings).

---

## Repository layout

```
bstack/
├── CLAUDE.md                  # Placeholder project instructions (replaced by /new or /import)
├── README.md
├── LICENSE                    # MIT
├── evolve/                    # Dated records of harness changes
└── .claude/
    ├── skills/                # The harness: one folder per skill, each with SKILL.md
    │   ├── new/  import/  plan/  build/  review/  verify/  ship/
    │   └── reflect/  evolve/  note/  status/  update/
    ├── memory/                # BStack's own memory (it is built with itself)
    ├── memory-viewer.html     # Standalone memory browser
    ├── settings.json          # Permission rules for memory/audit operations
    └── audit.log              # Lifecycle log
```

`.claude/settings.local.json` is gitignored: it holds per-machine permissions.

---

## Assumptions and limitations

- **Opinionated stack conventions.** `/review` and `/build` check for things like
  Supabase RLS policies, signed storage URLs, dark-mode classes and a
  migrations → types → services → hooks → components order. They fit a
  TypeScript/React/Supabase app best; adapt the skills for other stacks.
- **`/verify` assumes npm scripts.** It runs `npx tsc --noEmit`, `npm run lint` and
  `npm test`, and scans `src/`.
- **`/update` is PowerShell.** Its clone/compare/copy steps use PowerShell cmdlets.
  On macOS/Linux, Claude can usually translate them on the fly, but they are not
  written for bash yet.
- **Upstream URL is hardcoded.** `/update`, `/new` and the generated `harness.md`
  point at `github.com/freitasbruno/bstack`. If you fork, change that URL in
  `.claude/skills/update/SKILL.md` and `.claude/skills/new/SKILL.md`.

---

## License

[MIT](LICENSE)
