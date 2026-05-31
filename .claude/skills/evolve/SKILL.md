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
BStack, say so explicitly and provide the exact change to make there. Save it as a separate file for easy reference in ./bstack/evolve/[timestamp_feature_name ] for the user to apply.