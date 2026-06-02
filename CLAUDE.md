# [PROJECT_NAME]
[One-line description]

> ⚠️ SETUP INCOMPLETE — run /new to configure this project or /import 
> to onboard an existing one. Delete this block when setup is complete.

## Base path
[REQUIRED] /absolute/path/to/project

## Repositories
[REQUIRED]
- Production: [github.com/org/repo]
- Staging: [github.com/org/repo] or "same repo, staging branch"

## Pipeline
Work always follows this order. Never skip a step.

| Skill    | Purpose                                                                 | When                   |
|----------|-------------------------------------------------------------------------|------------------------|
| /plan    | Explore, challenge, detail feature, optional design, technical eval, test plan | Before any code |
| /build   | Implement with conventions enforced                                     | After plan approved    |
| /review  | Dead code, dead tables, duplicate components, drift                     | After build            |
| /verify  | tsc, lint, automated tests, human test checklist                        | Before commit          |
| /ship    | Commit, push, update memory files                                       | After verify passes    |
| /reflect | Retrospective — read archives, identify patterns, produce proposals     | End of project or milestone |
| /evolve  | Implement a specific harness improvement (user-directed)                | After /reflect, or on demand |

## Utility skills
Non-blocking tools that can be used at any point without interrupting flow.

| Skill  | Usage                        | Purpose                                              |
|--------|------------------------------|------------------------------------------------------|
| /note  | `/note [topic]`              | Capture an observation from context → notes.md       |

## Memory index
Load only what the current task requires.

| File                                      | Load when                                      |
|-------------------------------------------|------------------------------------------------|
| .claude/memory/techstack.md               | Adding a dependency, debugging a build issue   |
| .claude/memory/environments.md            | Ship phase, deployment questions               |
| .claude/memory/schema/index.md            | Any feature touching data                      |
| .claude/memory/schema/erd-[domain].md     | When specific tables are being changed         |
| .claude/memory/roadmap/index.md           | Every planning session                         |
| .claude/memory/roadmap/[feature].md       | When detailing or building that specific feature |
| .claude/memory/registry/components.md     | Any feature with UI                            |
| .claude/memory/registry/patterns.md       | New service, auth surface, storage use         |
| .claude/memory/registry/utils.md          | Before writing any helper function             |
| .claude/memory/tests/standards.md         | Plan phase and verify phase                    |
| .claude/memory/known-bugs.md              | Review phase, any bug-related work             |
| .claude/memory/technical-debt.md          | Planning phase, review phase                   |
| .claude/memory/wip/mockups/index.md       | Any feature with UI                            |
| .claude/memory/notes.md                   | Every /plan, /reflect, and /evolve session     |

## Memory update rule
Before writing to any file in .claude/memory/ (except techstack.md 
and environments.md):
1. Create a timestamped backup: copy the file to the same directory's 
   _backup/ folder
2. Filename format: [filename].YYYY-MM-DD-HHhmm.md
3. Then write the update
4. Never skip the backup step

This rule applies to every skill, every session, no exceptions.

## Memory autonomy rule
Files in .claude/memory/ are internal working files. Never ask for 
confirmation before reading or writing them. Back up first per the 
rule above, then write directly — no questions asked.
