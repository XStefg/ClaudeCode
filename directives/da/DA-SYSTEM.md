# Design Assistant (DA) — System Specification

## Purpose

The Design Assistant is a structured system for tracking design discussions, decisions, action items (TODOs and risks), system inventory (parts, components, software), and project context across projects. All records are stored as rich pages in Notion; local copies are written only at discussion conclusion.

---

## Concepts

### Project

A named entry in the global Projects database in Notion. Each project has its own Discussions, Decisions, Action Items, and System Items databases nested inside its Notion page. One project is "active" at a time, tracked in session context only — never persisted to disk.

### Discussion

A focused design conversation on a single topic. A discussion is open until it is concluded (`/da conclude`) or abandoned (`/da abandon`). One discussion may produce multiple decisions and action items. Discussions are stored as rich Notion pages.

### Decision

A design decision linked to the discussion that produced it. Each decision has a globally unique `DA-NNN` ID (per project). Decisions are stored as rich Notion pages with structured sections (Summary, Context, Rationale, Consequences).

### Action Item

Either a TODO or a RISK. Both live in the project's Action Items database and are linked to the discussion that created them. TODOs and RISKs use separate templates.

### System Item

A Part, Component, or Software module. All three live in the project's System Items database and are linked to the discussion that identified them. Parts, Components, and Software use separate templates.

### Template

Ten global template pages live under the root `Templates/` page in Notion. They define the section structure (headings + placeholder content) for each record type. Templates are fetched live before every record creation — changes made in Notion are picked up automatically.

#### Section markers

- `[later: hint]` — skip this section during record creation. The heading is created with no body content; the hint is for documentation only. Use this for sections filled by a specific command or lifecycle event, or populated programmatically (e.g. back-references filled at record creation time).

---

## Notion Structure

```
Project Design/                        ← root page (created once by first /da init)
  Templates/                           ← global templates page (created once)
    Project Page Template
    Discussion Template
    Decision Template
    TODO Template
    Risk Template
    Point Template
    Question Template
    Part Template                      🔩
    Component Template                 🧩
    Software Template                  💾
  Projects                             ← database (created once)
    [Robot Arm]/                       ← project entry
      Discussions                      ← database
      Decisions                        ← database
      Action Items                     ← database
      System Items                     ← database
    [Desk Lamp]/
      ...
```

### Projects database properties

| Property   | Type                          |
|------------|-------------------------------|
| Name       | title                         |
| Status     | select: Active / Archived     |
| GitHub URL | url                           |
| Created    | date                          |

### Per-project database properties

**Discussions**

| Property       | Type                                      |
|----------------|-------------------------------------------|
| Title          | title                                     |
| ID             | rich_text (e.g. DISC-001)                 |
| Status         | select: Open / Concluded / Abandoned      |
| Date Opened    | date                                      |
| Date Concluded | date                                      |

**Decisions**

| Property    | Type                                      |
|-------------|-------------------------------------------|
| Title       | title                                     |
| Status      | select: Active / Superseded               |
| ID          | rich_text (e.g. DA-001)                   |
| Discussion  | relation → Discussions                    |
| References  | relation → Discussions (multi)            |
| Date        | date                                      |

**Action Items**

| Property    | Type                                                        |
|-------------|-------------------------------------------------------------|
| Title       | title                                                       |
| ID          | rich_text (e.g. PT-001, Q-001, TODO-001, RISK-001)          |
| Type        | select: TODO / RISK / POINT / QUESTION                      |
| Status      | select: Open / Done / Mitigated / Answered / Active         |
| Priority    | select: High / Medium / Low                                 |
| Discussion  | relation → Discussions                                      |
| References  | relation → Discussions (multi)                              |

Status semantics by type: TODO → Open/Done · RISK → Open/Mitigated · POINT → Active · QUESTION → Open/Answered

The **References** relation is populated by `/da ref`.

**System Items**

| Property   | Type                                          |
|------------|-----------------------------------------------|
| Title      | title                                         |
| ID         | rich_text (e.g. PART-001, COMP-001, SFW-001)  |
| Type       | select: PART / COMPONENT / SOFTWARE           |
| Status     | select: Candidate / Selected / Rejected       |
| Category   | select (user-defined)                         |
| Discussion | relation → Discussions                        |
| References | relation → Discussions (multi)                |

Status semantics by type: PART / COMPONENT / SOFTWARE → Candidate/Selected/Rejected

The **References** relation is populated by `/da ref`.

---

## Default Template Content

These are the default section structures written to each template page during `/da init`. They may be freely edited in Notion — DA always fetches the current version.

### Project Page Template
```
## Description
[What this project is about]

## Existing Solutions
[Known prior art, competing products, or reference implementations]

## External References
[Links to datasheets, papers, resources, inspiration]

## Avenues to Explore
[Ideas, approaches, or directions worth investigating]

## Success Criteria
[How we'll know this project is done / working]
```

### Discussion Template
```
## Summary
[What this discussion is exploring]

## Key Points
[later: accumulated via /da point]

## Open Questions
[later: accumulated via /da question]

## Decisions
[later: accumulated via /da decision]

## TODOs
[later: accumulated via /da todo]

## Risks
[later: accumulated via /da risk]

## Parts
[later: accumulated via /da part]

## Components
[later: accumulated via /da component]

## Software
[later: accumulated via /da software]

## Conclusion
[later: filled in at /da conclude]
```

### Decision Template
```
## Discussion
[later: linked at creation]

## Summary
[One or two sentences stating the decision]

## Context
[What problem or question prompted this decision]

## Rationale
[Why this option was chosen over alternatives]

## Consequences
[What changes as a result; what to watch for]
```

### TODO Template
```
## Raised in
[later: linked at creation]

## Description
[What needs to be done]

## Acceptance Criteria
[How to know this TODO is complete]

## Notes
[Anything else relevant]
```

### Risk Template
```
## Raised in
[later: linked at creation]

## Description
[What could go wrong]

## Impact
[What happens if this risk materializes]

## Likelihood
[How probable is this]

## Mitigation
[How to prevent or reduce the risk]
```

### Point Template
```
## Raised in
[later: linked at creation]

## Point
[The observation or insight]

## Context
[Why this matters / what prompted it]
```

### Question Template
```
## Raised in
[later: linked at creation]

## Question
[The specific question to be answered]

## Context
[Why this question matters / what prompted it]

## Answer
[later: resolved in a future discussion via /da ref]
```

### Part Template 🔩
```
## Raised in
[later: linked at creation]

## Description
[What this part is]

## Manufacturer
[Manufacturer name]

## Part Number
[Part number or model]

## Datasheet
[Link or reference to datasheet]

## Notes
[Anything else relevant]
```

### Component Template 🧩
```
## Raised in
[later: linked at creation]

## Description
[What this component is and what it does]

## Interface
[How this component connects to the rest of the system]

## Notes
[Anything else relevant]
```

### Software Template 💾
```
## Raised in
[later: linked at creation]

## Description
[What this software does]

## Language
[Programming language or framework]

## Repository
[URL or reference to source code]

## Notes
[Anything else relevant]
```

---

## Local State

### Global DA config — `~/dev/.da/config.json`

Created on first `/da init`.

```json
{
  "notion_parent_page_id": "xxx",
  "notion_root_page_id": "xxx",
  "notion_projects_db_id": "xxx",
  "notion_templates_page_id": "xxx",
  "last_project": "robot-arm",
  "projects": {
    "robot-arm": "/Users/stephaneguerin/dev/Projects/robot-arm",
    "desk-lamp": "/Users/stephaneguerin/dev/Projects/desk-lamp"
  }
}
```

### Per-project config — `[project-path]/.da/config.json`

Created by `/da init` for each project.

```json
{
  "project_name": "My Project",
  "notion_project_page_id": "xxx",
  "notion_discussions_db_id": "xxx",
  "notion_decisions_db_id": "xxx",
  "notion_action_items_db_id": "xxx",
  "notion_system_items_db_id": "xxx",
  "discussion_counter": 5,
  "decision_counter": 8,
  "todo_counter": 4,
  "risk_counter": 2,
  "point_counter": 3,
  "question_counter": 1,
  "part_counter": 0,
  "component_counter": 0,
  "software_counter": 0
}
```

`active_project` and `current_discussion` are both held in **session context only** — never stored on disk. Each session starts with no active project; the user must run `/da open` to set one. `last_project` on disk is a convenience hint only — `/da open` (no args) uses it as a default. Multiple simultaneous sessions can each work on different projects independently.

### Per-project discussion archives

Raw local copies of concluded discussions are written to:
```
[project-path]/.da/discussions/DISC-[NNN]-[slug].md
```

---

## ID Scheme

| Record type | Format       | Example   | Counter in per-project config |
|-------------|--------------|-----------|-------------------------------|
| Discussion  | `DISC-[NNN]` | DISC-003  | `discussion_counter`          |
| Decision    | `DEC-[NNN]`  | DEC-007   | `decision_counter`            |
| TODO        | `TODO-[NNN]` | TODO-004  | `todo_counter`                |
| Risk        | `RISK-[NNN]` | RISK-002  | `risk_counter`                |
| Point       | `PNT-[NNN]`  | PNT-001   | `point_counter`               |
| Question    | `QST-[NNN]`  | QST-001   | `question_counter`            |
| Part        | `PART-[NNN]` | PART-001  | `part_counter`                |
| Component   | `COMP-[NNN]` | COMP-001  | `component_counter`           |
| Software    | `SFW-[NNN]`  | SFW-001   | `software_counter`            |

NNN is zero-padded to 3 digits. Counters increment at record creation and are never reused.

---

## Local Discussion Transcript Format

The local file at `[project-path]/.da/discussions/DISC-[NNN]-[slug].md` is a **chronological transcript** of the discussion session. It is created when the discussion opens and appended to by every action command.

**Created by `/da new` (or `/da new about [ID]`):**
```markdown
# DISC-003 — Motor controller selection
**Opened:** 2026-04-14
**Status:** Open

---
```

**Entry appended by each action command:**

| Command | Appended entry |
|---------|----------------|
| `/da point` | `**[date]** PNT-NNN — [title]` |
| `/da question` | `**[date]** QST-NNN — [title]` |
| `/da decision` | `**[date]** DEC-NNN — [title]` |
| `/da todo` | `**[date]** TODO-NNN — [title]` |
| `/da risk` | `**[date]** RISK-NNN — [title]` |
| `/da part` | `**[date]** PART-NNN — [title]` |
| `/da component` | `**[date]** COMP-NNN — [title]` |
| `/da software` | `**[date]** SFW-NNN — [title]` |

For `/da new about [ID]`, the first entry is: `**[date]** Opened about [ID] — [record title]`

**`/da conclude` appends and updates header:**
```markdown
**[date]** *Concluded*
[conclusion text]
```
Header line updated: `**Status:** Concluded`

**`/da abandon` appends and updates header:**
```markdown
**[date]** *Abandoned — [reason]*
```
Header line updated: `**Status:** Abandoned`

---

## Rules

1. Only one discussion may be open at a time per project. Conclude or abandon before starting a new one.
2. Records are immutable once created — decisions can be superseded via status update, but never deleted.
3. IDs are assigned at record creation time and stored in the Notion page properties.
4. Always fetch the live template from Notion before creating any record.
5. Config files must be written back to disk immediately after any state change (counter increment, `last_project` update). This includes `point_counter`, `question_counter`, `part_counter`, `component_counter`, and `software_counter`. Neither `active_project` nor `current_discussion` is ever stored on disk — both live in session context only.
5a. The local transcript file must be created when a discussion opens and appended to after every action command — it is the chronological session record, not a Notion mirror.
6. The `References` relation is updated in Notion when `/da ref` is called — it is not inferred.
