# Native file access in skills — 2026-06-09

## Problem

Skills used shell commands for file access, producing two failure modes:
1. Compound commands (`$(date +'%Y-%m-%d %H:%M')` embedded in echo strings)
   trip Claude Code's injection/obfuscation detector and force manual
   approval even when the command is allowlisted.
2. Out-of-tree shell reads fail closed to a prompt.

Root cause: audit.log entries in every lifecycle skill used
`echo "$(date +'%Y-%m-%d %H:%M') | [skill] | [slug] | START" >> .claude/audit.log`
The `$(date ...)` subshell substitution inside a quoted string was the
compound pattern that triggered the detector. The update skill had similar
issues: `$tmp = "...-$(Get-Date -Format '...')"` and
`Set-Location $tmp; git rev-parse --short HEAD`.

## Skills converted

### plan, build, review, verify, ship (2 changes each — START and END)

Removed:
```
Run `echo "$(date +'%Y-%m-%d %H:%M') | [skill] | [feature-slug] | START" >> .claude/audit.log`
```

Replaced with:
```
Append to .claude/audit.log using the Edit tool:
`YYYY-MM-DD HH:mm | [skill] | [feature-slug] | START`
(Substitute today's date and current time from your session context. Do not use shell date substitution.)
```

### update (2 changes)

1. `$tmp = "$env:TEMP\bstack-update-$(Get-Date -Format 'yyyyMMddHHmmss')"` — compound `$(...)` in string
   Fixed: split into `$ts = Get-Date -Format 'yyyyMMddHHmmss'` then `$tmp = "$env:TEMP\bstack-update-$ts"`

2. `Set-Location $tmp; git rev-parse --short HEAD` — semicolon chaining
   Fixed: split into two separate commands on separate lines

### evolve (1 addition)

Added "File access rule" section to ## Implementation documenting:
- Read tool for reads and existence checks
- Write/Edit for writes and appends
- Audit.log entries: compose timestamp inline, no shell date substitution
- Legitimate shell usage: git, npm/npx, mkdir, Copy-Item for skill bootstrapping

## Bootstrap verification

- Read/Edit/Write globs for memory tree: ✓ present (from previous evolve run)
- `~/.claude` in additionalDirectories: ✗ NOT present
  → Reported only; not added here (bootstrap's responsibility, not evolve's)
  → No current BStack skill reads from `~/.claude`, so no live breakage

## Verification

1. Read tool existence check on a non-existent wip path — returned file-not-found
   without prompting. ✓

2. Edit tool append to .claude/audit.log at a fresh position — completed without
   prompting. ✓

## Files changed

- D:\CLAUDE\bstack\.claude\skills\plan\SKILL.md
- D:\CLAUDE\bstack\.claude\skills\build\SKILL.md
- D:\CLAUDE\bstack\.claude\skills\review\SKILL.md
- D:\CLAUDE\bstack\.claude\skills\verify\SKILL.md
- D:\CLAUDE\bstack\.claude\skills\ship\SKILL.md
- D:\CLAUDE\bstack\.claude\skills\update\SKILL.md
- D:\CLAUDE\bstack\.claude\skills\evolve\SKILL.md

Backups: D:\CLAUDE\bstack\.claude\skills\_backup\[skill].SKILL.2026-06-09.md

## Apply to existing projects

For any existing BStack-scaffolded project, the skill files should be
updated by running /update in that project. The new skill files will
overwrite the old ones with these changes included.
