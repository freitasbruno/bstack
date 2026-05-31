Prompt 2 — Update running project

## Update this project: audit log — time tracking per pipeline step

Apply the same five-skill change from the bstack evolution of 2026-05-30.

In this project, make the following changes. All changes follow the same
pattern: two lines added per skill, no existing lines removed or modified.

The audit file is `.claude/audit.log` at the project root.
---

### .claude/skills/ship/SKILL.md

In "On start", after:
"2. Read .claude/memory/wip/[feature-slug].md"

Add:


---

### .claude/skills/plan/SKILL.md

After the block:
"Check .claude/memory/wip/ — if a file already exists for this
feature, read it and resume from where it left off."

Add:
"Append to .claude/audit.log:
[YYYY-MM-DD HH:mm] | plan | [feature-slug] | START"

After step 5 of "On approval":
"5. Tell the user: "Plan approved. Run /build to start implementation.""

Add:
"6. Append to .claude/audit.log:
[YYYY-MM-DD HH:mm] | plan | [feature-slug] | END"

---

### .claude/skills/build/SKILL.md

In "On start", after:
"2. Read .claude/memory/wip/[feature-slug].md"

Add:
"3. Append to .claude/audit.log:
[YYYY-MM-DD HH:mm] | build | [feature-slug] | START
(Renumber subsequent steps accordingly.)"

In "On completion", after:
"5. Update session log with completion note"

Add:
"6. Append to .claude/audit.log:
[YYYY-MM-DD HH:mm] | build | [feature-slug] | END"

---

### .claude/skills/review/SKILL.md

In "On start", after:
"2. Read .claude/memory/wip/[feature-slug].md"

Add:
"   Append to .claude/audit.log:
   [YYYY-MM-DD HH:mm] | review | [feature-slug] | START"

In "On completion", after:
"4. Back up and update known-bugs.md if bugs added or resolved"

Add:
"5. Append to .claude/audit.log:
[YYYY-MM-DD HH:mm] | review | [feature-slug] | END"

---

### .claude/skills/verify/SKILL.md

In "On start", after:
"2. Read .claude/memory/wip/[feature-slug].md"
"   Append to .claude/audit.log:
   [YYYY-MM-DD HH:mm] | review | [feature-slug] | START"

In "On completion", after:
"4. Back up and update known-bugs.md if bugs added or resolved"

Add:
"5. Append to .claude/audit.log:
[YYYY-MM-DD HH:mm] | review | [feature-slug] | END"

---

### .claude/skills/verify/SKILL.md

In "On start", after:
"2. Read .claude/memory/wip/[feature-slug].md"

Add:
"   Append to .claude/audit.log:
   [YYYY-MM-DD HH:mm] | verify | [feature-slug] | START"

In "On full pass", after:
"2. Add verify pass note to session log with timestamp"

Add:
"3. Append to .claude/audit.log:
[YYYY-MM-DD HH:mm] | verify | [feature-slug] | END"

---

### .claude/skills/ship/SKILL.md

In "On start", after:
"2. Read .claude/memory/wip/[feature-slug].md"

Add:
"   Append to .claude/audit.log:
   [YYYY-MM-DD HH:mm] | ship | [feature-slug] | START"

After "## Step 4 — Archive the wip" (after the archive move step), add:

"## Step 5 — Audit log

Append to .claude/audit.log:
[YYYY-MM-DD HH:mm] | ship | [feature-slug] | END"

---

Reasoning: single append-only file accumulates a permanent cross-feature
time record. START with no matching END flags interrupted sessions.
Evidence: identified during /evolve session 2026-05-30 — no operational
audit trail existed in the harness.

---
Prompt 2 — Update running project

## Update this project: audit log — time tracking per pipeline step

Apply the same five-skill change from the bstack evolution of 2026-05-30.

In this project, make the following changes. All changes follow the same
pattern: two lines added per skill, no existing lines removed or modified.

The audit file is `.claude/audit.log` at the project root.
Format: `[YYYY-MM-DD HH:mm] | [skill] | [feature-slug] | START/END`

### Files to edit:
- .claude/skills/plan/SKILL.md
- .claude/skills/build/SKILL.md
- .claude/skills/review/SKILL.md
- .claude/skills/verify/SKILL.md
- .claude/skills/ship/SKILL.md

Apply exactly the changes described in the bstack update prompt above.
Confirm each file updated and show the added lines.

This update comes from bstack evolution. Apply and confirm.