# DesignLog (DL) — System Specification

## Purpose

DesignLog is a structured design discussion system. It facilitates iterative design sessions across domains, captures decisions when consensus is reached, and maintains a single persistent log file per project.

---

## Concepts

### Domain
Each discussion belongs to one domain. Built-in domains:

| Code | Domain      |
|------|-------------|
| SW   | Software    |
| VI   | Visual      |
| EL   | Electronic  |
| WF   | Workflow    |
| HW   | Hardware    |
| UX   | UX/UI       |

New domains can be added as needed using `/dl domain add`.

### Discussion
A focused design conversation on a single topic within a domain. A discussion is open until a consensus is reached or it is explicitly closed/abandoned.

### Consensus
A decision or conclusion agreed upon within a discussion. A consensus produces a **Decision Record** that is appended to `DesignLog.md`.

### Decision Record
The atomic unit of DesignLog. Each record has a unique ID and captures the what, why, and key points of a decision.

**ID format:** `DL-[DOMAIN]-[###]` — e.g., `DL-SW-001`, `DL-VI-003`

Numbers are sequential per domain and never reused.

---

## Files

### Centralized (live in the directives directory — never copied)

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Loaded globally. Instructs Claude to follow the DL system. |
| `DL-SYSTEM.md` | This file. System rules and concepts. |
| `DL-COMMANDS.md` | Command reference. |
| `DL-TEMPLATE.md` | Template used by `/dl init` to create a project's `DesignLog.md`. |

### Per-project (created by `/dl init`)

| File | Purpose |
|------|---------|
| `DesignLog.md` | The live log of all decision records for this project. |
| `CLAUDE.md` *(updated)* | Gets a `@DesignLog.md` import so Claude loads the log automatically. |

### Setup

The directives `CLAUDE.md` is imported in `~/.claude/CLAUDE.md` so the DL system is always available globally. Run `/dl init` once in each new project directory to create its log and wire it up.

---

## Decision Record Format

```markdown
## DL-[DOMAIN]-[###] — [Title]

- **Date:** YYYY-MM-DD
- **Domain:** [Domain name]
- **Status:** [Active | Superseded by DL-XX-NNN | Abandoned]

### Decision
[One or two sentences stating the decision clearly.]

### Rationale
[Why this decision was made. Context that matters.]

### Key Points
- [Point 1]
- [Point 2]
- ...

### Supersedes
[DL-XX-NNN or —]
```

---

## Workflow

```
1. /dl new [domain] "[title]"    → opens a discussion
2. ... design conversation ...
3. /dl consensus "[summary]"     → records decision, closes discussion
   — or —
   /dl abandon "[reason]"        → discards discussion without logging
4. Repeat as needed
```

---

## Rules

1. One open discussion at a time (per session). Close or abandon before starting a new one.
2. A consensus is permanent — it can be **superseded** but never deleted.
3. IDs are assigned at consensus time, never before.
4. The `DesignLog.md` file is append-only. Records are never removed.
