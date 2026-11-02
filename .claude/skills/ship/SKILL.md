---
name: ship
description: "Commit and close skill for BStack projects. Commits the work, pushes to the correct repo, updates all memory files, reflects on the wip, archives it, and closes the task. Always runs after /verify passes."
---

# /ship — Ship It

Everything has been built, reviewed, and verified. Your job is to 
commit cleanly, update project memory so it reflects current reality, 
and close the wip.

## On start

1. Read CLAUDE.md
2. Read .claude/memory/wip/[feature-slug].md
   Append to .claude/audit.log using the Edit tool:
   `YYYY-MM-DD HH:mm | ship | [feature-slug] | START`
   (Substitute today's date and current time from your session context. Do not use shell date substitution.)
3. Confirm status is shipping — if not, stop and tell the user 
   which step is missing
4. Read .claude/memory/environments.md — confirm target repo and branch

## Step 1 — Reflect and update memory

Back up each file before updating it.

### Feature file
Read .claude/memory/roadmap/[feature-slug].md.

Rewrite it to describe the feature as it exists right now — not 
how it was planned, but what it currently is and does. 

Include:
- What the feature does in plain language
- Main components and where they live
- Data it reads and writes, which tables
- Non-obvious business logic or rules
- Known limitations or edge cases

Back up old version first, then overwrite.

### Roadmap index
Update roadmap/index.md — mark feature complete or update status,
and point its File column at roadmap/[feature-slug].md (the wip file
is about to move).

### Schema files
If feature changed the DB, update schema/index.md and relevant 
erd-[domain].md to reflect current state.

### Registry
Add any new reusable components, patterns, or utilities to:
- registry/components.md
- registry/patterns.md
- registry/utils.md

### Known bugs
- Add any TODOs or FIXMEs found during /verify
- Mark resolved any bugs fixed in this cycle

### Roadmap
Add any deferred decisions or new tasks identified during this 
cycle to roadmap/index.md with status not started.

## Step 2 — Check the roadmap, archive the wip, close the log

1. Re-read roadmap/index.md. The feature's row must show the status
   you just set, and its File column must not point at wip/. Fix it
   if not. Do not archive until both are true.
2. Add final session log entry with timestamp and completion note
3. Note any TODOs, FIXMEs, or deferred items added to roadmap
4. Move .claude/memory/wip/[feature-slug].md to 
   .claude/memory/wip/_archive/[feature-slug].YYYY-MM-DD-HHhmm.md
5. Append to .claude/audit.log using the Edit tool:
   `YYYY-MM-DD HH:mm | ship | [feature-slug] | END`
   (Substitute today's date and current time from your session context.)
   This comes before the commit so the line is included in it.

## Step 3 — Stage and commit

Show the user exactly what is staged:

git status
git diff --staged --stat

Do not auto-stage everything. Show changed files and ask:
"These are the files changed. Should all be staged, or do you 
want to exclude anything?"

The staged set will include updated memory files, the audit log and
the wip archive move — this is expected and correct.

On confirmation, stage and commit using conventional format:

<type>(<scope>): <short description>

[optional body — what changed and why]

Types: feat, fix, chore, refactor, style, docs, test, db
Rules: lowercase, no period, max 72 chars, imperative mood

Show proposed message and ask for confirmation before committing.

## Step 4 — Push

git push origin [branch]

Confirm push completed and show commit hash.
If the push fails, say the commit is local only and give the command
to push it. Do not report the task as closed until the push succeeds.

## Final report

Shipped: [feature-slug]
Commit:  [hash] — [message]
Pushed:  [repo] → [branch]

Memory updated:
- roadmap/[feature-slug].md — feature description current
- roadmap/index.md — status updated
- schema/ — [updated / not changed]
- registry/ — [N additions / not changed]
- known-bugs.md — [N added / N resolved / not changed]
- roadmap/index.md — [N new tasks from deferred items / not changed]

Task closed.
