# Memory-tree permission rules — 2026-06-09

## Problem

BStack skills repeatedly prompted for permission when writing to memory
files (wip, roadmap, backups, audit log). Every new feature slug and every
timestamped backup path is a fresh path — per-path "always allow" approvals
accumulated in settings.local.json but never covered new paths.

Root cause (confirmed): relative-path rules like `Edit(.claude/memory/**)`
do not match when the Write/Edit tool receives an absolute path such as
`D:/CLAUDE/bstack/.claude/memory/wip/feature.md`. Proof: settings.json
already contained both relative and absolute versions for audit.log,
indicating the gap had been hit and patched there but not for the memory
tree. Additionally, Bash `cp` backup commands had no glob coverage.

## Layout detected

Per-project memory, inside the project root:

  D:\CLAUDE\bstack\.claude\memory\   (Write/Edit/Read tool writes)
  D:\CLAUDE\bstack\.claude\memory\wip\   (plan: wip files)
  D:\CLAUDE\bstack\.claude\memory\**\_backup\   (backup copies)
  D:\CLAUDE\bstack\.claude\audit.log   (Bash echo >> audit.log)

Case A (variable paths per session) was the primary driver.

## Files changed

### 1. D:\CLAUDE\bstack\.claude\settings.json
   Backup: D:\CLAUDE\bstack\.claude\settings.json.2026-06-09.bak

   Added to permissions.allow (no existing rules removed):
   - Read(.claude/memory/**)
   - Read(D:/CLAUDE/bstack/.claude/memory/**)
   - Edit(D:/CLAUDE/bstack/.claude/memory/**)
   - Write(D:/CLAUDE/bstack/.claude/memory/**)
   - MultiEdit(D:/CLAUDE/bstack/.claude/memory/**)
   - Bash(cp *_backup*)
   - Bash(cp * .claude/memory/**)
   - Bash(cp * D:/CLAUDE/bstack/.claude/memory/**)
   - Bash(mkdir -p .claude/memory/**)
   - Bash(mkdir -p D:/CLAUDE/bstack/.claude/memory/**)

### 2. D:\CLAUDE\bstack\.claude\skills\new\SKILL.md

   Updated Phase 6 — Scaffold to create .claude/settings.json in every
   new project with both relative and absolute-path memory-tree rules.
   The absolute BASE_PATH is substituted from the path confirmed in Phase 1.

## Rules NOT changed

- defaultMode unchanged (not set)
- bypassPermissions not added
- All pre-existing allow entries untouched

## Apply to existing projects

For any existing BStack-scaffolded project (e.g. d:/CLAUDE/builog), add
these entries to its .claude/settings.json (or create one if absent).
Replace BASE_PATH with the project's absolute root:

```json
"Read(.claude/memory/**)",
"Edit(.claude/memory/**)",
"Write(.claude/memory/**)",
"MultiEdit(.claude/memory/**)",
"Read(BASE_PATH/.claude/memory/**)",
"Edit(BASE_PATH/.claude/memory/**)",
"Write(BASE_PATH/.claude/memory/**)",
"MultiEdit(BASE_PATH/.claude/memory/**)",
"Edit(.claude/audit.log)",
"Write(.claude/audit.log)",
"Edit(BASE_PATH/.claude/audit.log)",
"Write(BASE_PATH/.claude/audit.log)",
"Bash(echo * >> .claude/audit.log)",
"Bash(*audit.log*)",
"Bash(cp *_backup*)",
"Bash(cp * .claude/memory/**)",
"Bash(cp * BASE_PATH/.claude/memory/**)",
"Bash(mkdir -p .claude/memory/**)",
"Bash(mkdir -p BASE_PATH/.claude/memory/**)",
"PowerShell(Copy-Item *_backup*)",
"PowerShell(New-Item -ItemType Directory *)"
```
