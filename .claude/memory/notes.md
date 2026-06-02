# Project Notes

Live session captures. Read during /plan, /reflect, and /evolve to surface patterns, risks, and ideas before they're lost.

---

### 2026-06-02 — Audit writes must use real system timestamp
**Context:** During a /evolve session adding settings.json permission scoping, a note was raised about audit logging behaviour.
**Observation:** When writing entries to .claude/audit.log, Claude must fetch the real system timestamp (via a shell call or the currentDate context) rather than deriving or assuming the time. Hardcoded or inferred timestamps produce misleading audit trails that are useless for debugging session timelines. This applies to any skill that writes to audit.log.
**Relevant to:** build, evolve, ship
**Tags:** audit.log, timestamp, logging, correctness

### 2026-06-02 — Add mockups folder to default plan-stage structure
**Context:** Raised while working on a separate running project that needed a place to store UI mockups produced during the /plan skill.
**Observation:** The default BStack project structure has no designated location for plan-stage mockups (wireframes, ASCII layouts, design sketches). These currently have nowhere to live, so they either get embedded in wip files or lost. A `wip/mockups/` or `.claude/mockups/` folder should be part of the scaffolded structure created by /new, and /plan should reference it when design work is produced.
**Relevant to:** plan, evolve
**Tags:** structure, mockups, scaffolding, /new, /plan
