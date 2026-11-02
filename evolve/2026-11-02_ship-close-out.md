# /ship close-out gaps — 2026-11-02

## Problem

Found while closing the `contabilidade` feature in embffl-admin:

1. **Roadmap status missed.** The ship session log said the roadmap row was
   marked complete, but the commit still had it as in progress, linking to the
   wip file that had just been archived. Nothing checked the row before
   archiving, and the skill never said to update the File column.
2. **Audit log always left dirty.** The `END` line was appended after the commit
   and push, so every ship left `.claude/audit.log` modified and uncommitted.

## Changes to `.claude/skills/ship/SKILL.md`

- **Roadmap index:** also point the File column at `roadmap/[feature-slug].md`.
- **Step 2:** starts by re-reading `roadmap/index.md`. The row must show the
  status just set and its File column must not point at `wip/`, or the wip is
  not archived. The check compares against the status set, not always complete,
  because a feature can ship partly done.
- **Audit `END`:** moved from the old Step 5 to the end of Step 2, before the
  commit. Step 5 is removed.
- **Step 3:** the staged set now names the audit log.
- **Step 4:** on a failed push, say the commit is local only, give the push
  command, and do not report the task as closed.

## Trade-off

`END` is now logged just before the commit, so ship durations no longer include
the commit, the push or waiting for confirmations. Accepted over a second commit
on every ship just for the log line. If a push fails, the log shows `END` before
the work reaches the remote, which is why Step 4 now covers that case.

## Apply to existing projects

Run `/update` in each project. It replaces `.claude/skills/ship/SKILL.md` and
changes nothing else in the project.

A project with `.claude/audit.log` still modified from an earlier ship can commit
that line along with the update.

Projects with roadmap rows that still show in progress after being shipped are
not corrected by this change. Check for rows linking to a `wip/` file that no
longer exists.
