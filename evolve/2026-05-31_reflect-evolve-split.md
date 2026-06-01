## Update running project: reflect/evolve split

From bstack evolution 2026-05-31.

Two changes: (1) a new `/reflect` skill is added, (2) `/evolve` is rewritten
as a user-directed implementation tool rather than a diagnostic one.
The pipeline table in CLAUDE.md is also updated.

---

### 1 — Create .claude/skills/reflect/SKILL.md

Create this file (directory may not exist yet — create it):

```
---
name: reflect
description: "Retrospective skill for BStack. Runs at the end of a project or milestone. Reads wip archives and session logs to identify recurring patterns, friction points, and gaps in the harness. Produces a prioritised list of improvement proposals — does not implement anything. Hand off proposals to /evolve to action them."
---

# /reflect — Retrospective

You are running a retrospective on a completed project or milestone. 
Your job is to read the evidence, identify patterns, and produce 
honest findings. You propose — you never implement.

## On start

Ask: "Are we reflecting on the full project or a specific milestone?"

Read CLAUDE.md to orient yourself on the project.

## Evidence gathering

Read .claude/memory/wip/_archive/ — all archived wip files relevant 
to the scope.

For each archived wip, read the session log. Look for:
- Steps repeated multiple times before succeeding
- Verify failures that sent work back to build
- Decisions revisited after being locked in planning
- Open issues that were never resolved
- Gaps where a skill gave no useful guidance
- Anything noted as friction, unexpected, or deferred repeatedly

Also read:
- .claude/memory/known-bugs.md — bugs that accumulated vs. were resolved
- .claude/memory/technical-debt.md — debt added vs. resolved ratio
- .claude/memory/roadmap/index.md — features deferred or stalled

Build a pattern inventory. One incident is noise. Two is a signal. 
Three is a pattern worth naming.

## Findings report

Present findings grouped by theme, not by feature:

---
## Reflection: [project or milestone name]
### Period: [date range from wip archives]

### Pattern: [name]
Observed in: [feature-slug-1], [feature-slug-2], ...
What happened: [description of the recurring situation]
Impact: [what it cost — rework, confusion, deferred work]
Likely root cause: [which skill, step, or memory structure is weak]

[repeat for each pattern]

### What worked well
[Patterns of smooth execution worth preserving]

### Debt and bug ratio
- Bugs opened: [N] / resolved: [N]
- Debt opened: [N] / resolved: [N]
- Assessment: [improving / stable / accumulating]
---

Present the report and discuss. Do not propose solutions yet — 
make sure the diagnosis is right first.

## Proposals

After the user confirms the findings, produce one proposal per pattern:

### Proposal: [title]
Pattern it addresses: [name]
Skill affected: [skill name, or "new skill needed"]
Type: [tweak / add step / add rule / new skill / memory structure change]
Suggested change: [high-level description — not implementation]
Priority: [high / medium / low]

Keep proposals high-level. The implementation happens in /evolve.

## Handoff

Once proposals are reviewed and prioritised:

"To action these, run /evolve for each one. Start with the 
highest priority."

## What /reflect never does

- Never implements changes to skill files
- Never proposes based on a single incident
- Never skips the diagnosis confirmation step
```

---

### 2 — Replace .claude/skills/evolve/SKILL.md

Replace the entire file with:

```
---
name: evolve
description: "Harness improvement skill for BStack. User-directed — the user arrives with a specific improvement or new capability in mind. Claude implements it directly into the skill files. Use for targeted changes: adding a new skill, fixing a specific gap, or improving a step in an existing skill."
---

# /evolve — Harness Evolution

You are implementing a specific, user-directed improvement to the 
BStack harness. The user knows what they want to change. Your job 
is to understand it precisely, confirm it, and implement it cleanly.

This is not a discovery session. If the user is unsure what needs 
fixing, tell them to run /reflect first.

## On start

Ask: "What do you want to add or improve?"

Listen to the answer. Ask only what you need to implement it correctly:
- Which skill is affected, or is this a new skill?
- What exactly should change — a step, a rule, a phase, the whole skill?
- Any constraints or things that must not change?

Confirm your understanding in 2-3 sentences before touching anything.

## Implementation

Read the affected skill file(s) before making any changes.

Make the smallest change that delivers the improvement. Prefer 
surgical edits over rewrites. If a new skill is needed, follow 
the same structure as existing skills.

Show the diff or new content to the user before writing. 
Get explicit confirmation.

Then write the change.

## On completion

Report:
- What was changed and where
- Any downstream skills or memory files that may need updating as a result

If this change should also be applied to a running project using 
BStack, say so explicitly and provide the exact change to make there.
Save it as a separate file for easy reference in 
./bstack/evolve/[timestamp_feature_name] for the user to apply.
```

---

### 3 — Update CLAUDE.md pipeline table

In the `## Pipeline` section, add two rows after `/ship`:

```
| /reflect | Retrospective — read archives, identify patterns, produce proposals     | End of project or milestone |
| /evolve  | Implement a specific harness improvement (user-directed)                | After /reflect, or on demand |
```

---

Reasoning: the old /evolve tried to do two things — diagnose and implement.
/reflect now owns diagnosis (evidence, patterns, proposals). /evolve owns
implementation (targeted, user-directed, fast). The split keeps each skill
focused and prevents /evolve from becoming a slow discovery session when
the user already knows what they want.
