# DesignLog — Command Reference

Commands are prefixed with `/dl`. In a Claude Code session, type them as prompts.

---

## Setup Commands

### `/dl init`
Initialize DesignLog in the current project directory.
- Creates `DesignLog.md` from the system template.
- Creates or updates the project's `CLAUDE.md` with a DL section so the log is auto-loaded.

Run once per project. Safe to re-run — it will not overwrite an existing `DesignLog.md`.

---

## Discussion Commands

### `/dl new [domain] "[title]"`
Start a new discussion.
- `[domain]` — domain code (SW, VI, EL, WF, HW, UX) or a custom code
- `[title]` — short description of the topic

**Example:**
```
/dl new SW "Choose between REST and GraphQL for the API"
```

---

### `/dl status`
Show the current open discussion (title, domain, points noted so far).

---

### `/dl point "[text]"`
Add an important point to the current discussion without closing it. Points are collected and included in the final consensus record.

**Example:**
```
/dl point "GraphQL adds complexity we don't need yet"
```

---

### `/dl consensus "[summary]"`
Record a decision. Closes the current discussion and appends a Decision Record to `DesignLog.md`.
- `[summary]` — one or two sentences stating the decision

**Example:**
```
/dl consensus "Use REST for v1. GraphQL to be reconsidered if query complexity warrants it."
```

---

### `/dl abandon "[reason]"`
Close the current discussion without recording a decision.

**Example:**
```
/dl abandon "Needs more research. Will revisit after prototyping."
```

---

## Log Commands

### `/dl log`
Display all Decision Records in `DesignLog.md`.

---

### `/dl log [domain]`
Display Decision Records filtered by domain.

**Example:**
```
/dl log SW
```

---

### `/dl show [id]`
Display a single Decision Record by ID.

**Example:**
```
/dl show DL-SW-001
```

---

### `/dl list`
List all decisions as a compact summary table (ID, title, date, status).

---

## Management Commands

### `/dl supersede [id] "[reason]"`
Mark an existing record as superseded by the *current open discussion's* upcoming consensus. Use before `/dl consensus`.

**Example:**
```
/dl supersede DL-SW-001 "Switching to GraphQL after v1 complexity grew"
```

---

### `/dl domain add [code] "[name]"`
Register a new domain.

**Example:**
```
/dl domain add ME "Mechanical"
```

---

### `/dl domain list`
List all registered domains and their codes.

---

## Quick Reference

| Command | Action |
|---------|--------|
| `/dl init` | Initialize DL in current project |
| `/dl new [domain] "[title]"` | Start discussion |
| `/dl point "[text]"` | Add a key point |
| `/dl status` | Show open discussion |
| `/dl consensus "[summary]"` | Record decision & close |
| `/dl abandon "[reason]"` | Close without recording |
| `/dl list` | List all decisions |
| `/dl log` | Show full log |
| `/dl log [domain]` | Filter log by domain |
| `/dl show [id]` | Show one record |
| `/dl supersede [id] "[reason]"` | Mark record superseded |
| `/dl domain add [code] "[name]"` | Add domain |
| `/dl domain list` | List domains |
