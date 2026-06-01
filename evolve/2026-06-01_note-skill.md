## New skill: /note — Quick Capture

From bstack evolution 2026-06-01.

Adds a non-blocking `/note [topic]` utility skill. Reads running conversation context, synthesizes a well-formed note, appends to `.claude/memory/notes.md`, confirms with one line, and returns. No questions, no back-and-forth.

Notes accumulate across sessions and are read during `/plan`, `/reflect`, and `/evolve` to surface patterns, risks, and ideas.

---

### 1 — Create .claude/skills/note/SKILL.md

Create this file (create the directory too):

```
---
name: note
description: "Quick capture skill for BStack. Invoked as /note [topic] — reads the current conversation context, synthesizes a well-formed note, and appends it to .claude/memory/notes.md. Non-blocking: no questions asked, one-line confirmation, returns immediately. Notes surface during /plan, /reflect, and /evolve."
---

# /note — Quick Capture

You are capturing a note from a live session. The user has flagged something worth preserving. Your job is to synthesize it into a well-formed, self-contained note and append it — then stop. No questions. No back-and-forth. Return immediately.

## On invocation

The user typed `/note [topic]`. The topic is your raw material.

Read the recent conversation context to understand what the topic refers to — what decision, friction point, idea, risk, or observation is being flagged. Do not ask for clarification. Derive it from context.

## Synthesize the note

Write a note in this exact structure:

```
### YYYY-MM-DD — [topic title, 5-10 words]
**Context:** [1-2 sentences on what was happening when this was noted]
**Observation:** [The core insight, problem, or idea — 2-4 sentences. Specific enough to be useful cold, months later.]
**Relevant to:** [comma-separated subset of: plan / reflect / evolve / build / review]
**Tags:** [comma-separated keywords]
```

Notes must stand alone. Avoid vague observations like "this was complicated." Name the exact thing and why it matters.

## Append to notes.md

Append the note to `.claude/memory/notes.md`.

If the file does not exist, create it with this header first:

```
# Project Notes

Live session captures. Read during /plan, /reflect, and /evolve to surface patterns, risks, and ideas before they're lost.

---
```

The memory backup rule does NOT apply to notes.md — it is append-only and never overwritten.

## Confirm and stop

Reply with exactly one line: `Noted: [topic title]`

Do not summarize the note in your reply. Do not ask follow-up questions. Do not continue any prior conversation thread — return control to the user immediately.

## What /note never does

- Never asks clarifying questions before writing
- Never produces multi-line output (one confirmation line only)
- Never overwrites or edits existing notes
- Never reads the full notes.md (append only — do not load the whole file)
- Never skips writing the note to file (even if the topic seems obvious)
```

---

### 2 — Update CLAUDE.md

**In the Pipeline section**, add after the `/evolve` row:

```
## Utility skills
Non-blocking tools that can be used at any point without interrupting flow.

| Skill  | Usage                        | Purpose                                              |
|--------|------------------------------|------------------------------------------------------|
| /note  | `/note [topic]`              | Capture an observation from context → notes.md       |
```

**In the Memory index table**, add a row:

```
| .claude/memory/notes.md                   | Every /plan, /reflect, and /evolve session     |
```

---

### 3 — Update /reflect and /plan skills (optional improvement)

In each, add to the evidence-gathering section:

```
Read .claude/memory/notes.md — live session captures that may surface 
patterns, risks, or ideas relevant to this session.
```

---

### Note on non-blocking behaviour

Skills in Claude Code run in the main conversation thread — true parallel execution is not available for skills. The non-blocking feel comes from design constraints: inline args, zero questions, one-line response. This is the same pattern as `/btw`-style commands. The skill is as instant as the model can make it.
