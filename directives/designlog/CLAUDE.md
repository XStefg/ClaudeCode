# DesignLog (DL) System

@DL-SYSTEM.md
@DL-COMMANDS.md

## Instructions

You are DesignLog-aware in every project. Follow these rules at all times:

- Recognize and handle all `/dl` commands as defined in `DL-COMMANDS.md`.
- Follow the rules in `DL-SYSTEM.md` strictly.
- When a project has been initialized (`DesignLog.md` exists in the working directory), load it as the active log.
- When `/dl consensus` is issued, generate a properly formatted Decision Record, append it to the project's `DesignLog.md`, and increment the domain counter in the ID Counters table.
- When `/dl point` is issued, acknowledge and hold the point in context for the current discussion.
- Do not assign decision IDs before consensus is reached.
- `DesignLog.md` is append-only — records are never removed, only superseded.

### `/dl init` behavior
When `/dl init` is run in a project directory:
1. Create `DesignLog.md` from the template at `DL-TEMPLATE.md`.
2. Create or update the project's `CLAUDE.md`:
   - If no `CLAUDE.md` exists: create one with a DL section.
   - If one exists: append the DL section if not already present.
3. The DL section to add:

```markdown
## DesignLog
@DesignLog.md
```

4. Confirm to the user that the project is DL-ready.
