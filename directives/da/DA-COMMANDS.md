# Design Assistant (DA) — Command Reference

Commands are prefixed with `/da`. In a Claude Code session, type them as prompts.

---

## Project Management

### `/da init "[project-name]"`

Initialize a new project.

**First run (no `~/dev/.da/config.json` exists):**
1. Ask the user for a Notion parent page ID (where the "Project Design" root page will be created).
2. Create the full global Notion structure:
   - "Project Design" root page under the provided parent
   - "Templates" child page with 10 template pages (Project Page, Discussion, Decision, TODO, Risk, Point, Question, Part 🔩, Component 🧩, Software 💾) — populated with default content from DA-SYSTEM.md
   - "Projects" database with properties: Name (title), Status (select), GitHub URL (url), Created (date)
3. Write `~/dev/.da/config.json` with all resulting Notion IDs.
4. Proceed to create the first project entry (step below).

**Subsequent runs:**
1. Create a new entry in the Projects database (Status=Active, Created=today).
2. Create the project's Notion page (fetch Project Page Template, prompt for unfilled sections).
3. Create 4 child databases inside the project page: Discussions, Decisions, Action Items, System Items — with the properties defined in DA-SYSTEM.md.
4. Create the local directory structure:
   - `[project-path]/.da/`
   - `[project-path]/.da/discussions/`
   - `[project-path]/.da/config.json`
5. Register the project in `~/dev/.da/config.json → projects`. Write `last_project` = new project slug to global config on disk. Set session-local `active_project` to the new project slug. Do NOT write `active_project` to disk.

The project path is derived as `~/dev/Projects/[slug]/` where slug is the project name lowercased with spaces replaced by hyphens.

---

### `/da open "[project-name]"`

Switch to a named project. Partial name match accepted (case-insensitive).

1. Look up the project in `global config → projects`.
2. Write `last_project` = current session `active_project` (if set) to global config on disk.
3. Set session-local `active_project` to the matched project slug. Do NOT write `active_project` to disk.
4. Confirm: display new active project name and Notion project page link.

---

### `/da open`

Reopen the last active project (`last_project` in global config). Equivalent to `/da open [last_project]`.

---

### `/da close`

Deactivate the current project in this session.

1. Require an active project (error if none).
2. If a discussion is currently open, block and tell the user to conclude or abandon it first.
3. Write `last_project` = current `active_project` slug to global config on disk.
4. Clear `active_project` from session context.
5. Confirm: display the name of the project that was closed.

---

### `/da projects`

List all projects registered in the global config. Mark the active one with `*`.

Output format:
```
  desk-lamp — Desk Lamp
* robot-arm  — Robot Arm   ← active
```

---

### `/da config`

Display current state:
- Active project name and Notion project page link
- Current open discussion (title, DISC-NNN, date opened) or "No open discussion"
- Record counters (discussions, decisions, TODOs, risks, points, questions, parts, components, software)

---

## Discussion Lifecycle

### `/da new "[title]"`

Open a new discussion.

1. Check that no discussion is currently open (error if one is).
2. Fetch the global Discussion Template from Notion.
3. Apply the `[later:]` rule: prompt only for non-`[later:]` sections (Summary). Skip all others.
4. Create the Notion page in the active project's Discussions database (Status=Open, Date Opened=today, ID=DISC-[NNN]).
5. Increment `discussion_counter`. Assign ID `DISC-[NNN]`. Write updated per-project config.
6. Hold `current_discussion` in session context: `{ notion_page_id, title, opened, disc_number }`.
7. Create the local transcript file at `[project-path]/.da/discussions/DISC-[NNN]-[slug].md`:
   ```
   # DISC-NNN — [title]
   **Opened:** [date]
   **Status:** Open

   ---
   ```
8. Confirm: display DISC-NNN and Notion page link.

---

### `/da new about [ID]`

Open a new discussion anchored to an existing record.

1. Check that no discussion is currently open (error if one is).
2. Look up the record by ID in Notion (any type: DEC-NNN, TODO-NNN, RISK-NNN, PNT-NNN, QST-NNN, DISC-NNN, PART-NNN, COMP-NNN, SFW-NNN).
3. Fetch and display the record's full content in the session — the discussion starts with full context.
4. Auto-generate the title: `[ID] — [record title]` (no prompt).
5. Fetch the global Discussion Template from Notion.
6. Apply the `[later:]` rule: prompt only for non-`[later:]` sections (Summary). Skip all others.
7. Create the Notion page in the Discussions database (Status=Open, Date Opened=today, ID=DISC-[NNN]).
8. Inject a **Regarding** section at the top of the discussion page (above Summary) with a Notion page mention/link to the source record:
   ```
   ## Regarding
   → [ID] — [record title]
   ```
   This section is not part of the standard Discussion Template — injected only for `about` discussions.
9. Link the record:
   - Add the new discussion to the record's **References** relation in Notion.
   - Append a linked bullet to the appropriate section of the discussion page based on record type (RISK → Risks, QST-NNN → Open Questions, TODO → TODOs, PNT-NNN → Key Points, DEC-NNN → Decisions). For PART-NNN, COMP-NNN, and SFW-NNN, skip this step — the discussion template has no sections for system items.
10. Increment `discussion_counter`. Assign ID `DISC-[NNN]`. Write updated per-project config.
11. Hold `current_discussion` in session context: `{ notion_page_id, title, opened, disc_number }`.
12. Create the local transcript file:
    ```
    # DISC-NNN — [title]
    **Opened:** [date]
    **Status:** Open

    ---
    **[date]** Opened about [ID] — [record title]
    ```
13. Confirm: display DISC-NNN, Notion page link, and the linked record.

---

### `/da status`

Fetch the current discussion page from Notion and show a full summary:
- DISC-NNN title, date opened
- Each record type listed separately (fetch live from Notion, not session context):
  - Points (PNT-NNN — title)
  - Questions (QST-NNN — title)
  - Decisions (DEC-NNN — title)
  - TODOs (TODO-NNN — title)
  - Risks (RISK-NNN — title)
  - Parts (PART-NNN — title)
  - Components (COMP-NNN — title)
  - Software (SFW-NNN — title)
- Show "none" for any type with no linked records.

If no discussion is open, say so.

---

### `/da point "[text]"`

Create a Point action item linked to the current discussion.

1. Require an open discussion (error if none).
2. Fetch the global Point Template from Notion.
3. Prompt for each non-`[later:]` section (Description).
4. Increment `point_counter`. Assign ID `PNT-[NNN]`.
5. Create the Notion page in the Action Items database with:
   - Type = POINT, Status = Active, ID = PNT-[NNN]
   - Discussion relation = current discussion page
   - Fill the `Raised in` section with a Notion page mention/link to the current discussion.
6. Append a linked bullet to the `Key Points` section of the current discussion page: `- PNT-NNN — [title]`.
7. Append to the local transcript: `**[date]** PNT-NNN — [title]`
8. Write updated per-project config.
9. Confirm: display PNT-NNN and Notion page link.

---

### `/da question "[text]"`

Create a Question action item linked to the current discussion.

1. Require an open discussion (error if none).
2. Fetch the global Question Template from Notion.
3. Prompt for each non-`[later:]` section (Question).
4. Increment `question_counter`. Assign ID `QST-[NNN]`.
5. Create the Notion page in the Action Items database with:
   - Type = QUESTION, Status = Open, ID = QST-[NNN]
   - Discussion relation = current discussion page
   - Fill the `Raised in` section with a Notion page mention/link to the current discussion.
6. Append a linked bullet to the `Open Questions` section of the current discussion page: `- QST-NNN — [title]`.
7. Append to the local transcript: `**[date]** QST-NNN — [title]`
8. Write updated per-project config.
9. Confirm: display QST-NNN and Notion page link.

---

### `/da decision "[title]"`

Create a decision linked to the current discussion.

1. Require an open discussion (error if none).
2. Fetch the global Decision Template from Notion.
3. Prompt for each non-`[later:]` section (Details).
4. Increment `decision_counter`. Assign ID `DEC-[NNN]`.
5. Create the Notion page in the Decisions database with:
   - ID property = `DEC-[NNN]`
   - Discussion relation = current discussion page
   - Status = Active
   - Date = today
   - Fill the `Discussion` section with a Notion page mention/link to the current discussion.
6. Append a linked bullet to the `Decisions` section of the current discussion page: `- DEC-NNN — [title]`.
7. Append to the local transcript: `**[date]** DEC-NNN — [title]`
8. Write updated per-project config.
9. Confirm: display DEC-NNN and Notion page link.

---

### `/da todo "[text]"`

Create a TODO action item linked to the current discussion.

1. Require an open discussion (error if none).
2. Fetch the global TODO Template from Notion.
3. Prompt for each non-`[later:]` section (Description).
4. Increment `todo_counter`. Assign ID `TODO-[NNN]`.
5. Create the Notion page in the Action Items database with:
   - Type = TODO, Status = Open, ID = TODO-[NNN]
   - Discussion relation = current discussion page
   - Fill the `Raised in` section with a Notion page mention/link to the current discussion.
6. Append a linked bullet to the `TODOs` section of the current discussion page: `- TODO-NNN — [title]`.
7. Append to the local transcript: `**[date]** TODO-NNN — [title]`
8. Write updated per-project config.
9. Confirm: display TODO-NNN and Notion page link.

---

### `/da risk "[text]"`

Create a RISK action item linked to the current discussion.

1. Require an open discussion (error if none).
2. Fetch the global Risk Template from Notion.
3. Prompt for each non-`[later:]` section (Description).
4. Increment `risk_counter`. Assign ID `RISK-[NNN]`.
5. Create the Notion page in the Action Items database with:
   - Type = RISK, Status = Open, ID = RISK-[NNN]
   - Discussion relation = current discussion page
   - Fill the `Raised in` section with a Notion page mention/link to the current discussion.
6. Append a linked bullet to the `Risks` section of the current discussion page: `- RISK-NNN — [title]`.
7. Append to the local transcript: `**[date]** RISK-NNN — [title]`
8. Write updated per-project config.
9. Confirm: display RISK-NNN and Notion page link.

---

### `/da part "[name]"` 🔩

Create a Part system item linked to the current discussion.

1. Require an open discussion (error if none).
2. Fetch the global Part Template from Notion.
3. Prompt for each non-`[later:]` section (Description, Part Number, Where to buy, Notes).
4. Increment `part_counter`. Assign ID `PART-[NNN]`.
5. Create the Notion page in the System Items database with:
   - Type = PART, Status = Candidate, ID = PART-[NNN]
   - Discussion relation = current discussion page
   - Fill the `Raised in` section with a Notion page mention/link to the current discussion.
6. Append to the local transcript: `**[date]** PART-NNN — [title]`
7. Write updated per-project config.
8. Confirm: display PART-NNN and Notion page link.

---

### `/da component "[name]"` 🧩

Create a Component system item linked to the current discussion.

1. Require an open discussion (error if none).
2. Fetch the global Component Template from Notion.
3. Prompt for each non-`[later:]` section (Description, Notes).
4. Increment `component_counter`. Assign ID `COMP-[NNN]`.
5. Create the Notion page in the System Items database with:
   - Type = COMPONENT, Status = Candidate, ID = COMP-[NNN]
   - Discussion relation = current discussion page
   - Fill the `Raised in` section with a Notion page mention/link to the current discussion.
6. Append to the local transcript: `**[date]** COMP-NNN — [title]`
7. Write updated per-project config.
8. Confirm: display COMP-NNN and Notion page link.

---

### `/da software "[name]"` 💾

Create a Software system item linked to the current discussion.

1. Require an open discussion (error if none).
2. Fetch the global Software Template from Notion.
3. Prompt for each non-`[later:]` section (Description, Notes).
4. Increment `software_counter`. Assign ID `SFW-[NNN]`.
5. Create the Notion page in the System Items database with:
   - Type = SOFTWARE, Status = Candidate, ID = SFW-[NNN]
   - Discussion relation = current discussion page
   - Fill the `Raised in` section with a Notion page mention/link to the current discussion.
6. Append to the local transcript: `**[date]** SFW-NNN — [title]`
7. Write updated per-project config.
8. Confirm: display SFW-NNN and Notion page link.

---

### `/da ref [id]`

Load a record into context and formally link it to the current discussion.

`[id]` may be any record ID: `DEC-NNN`, `DISC-NNN`, `TODO-NNN`, `RISK-NNN`, `PNT-NNN`, `QST-NNN`, `PART-NNN`, `COMP-NNN`, or `SFW-NNN`.

1. Require an open discussion (error if none).
2. Look up the record in Notion (search by ID property across Discussions, Decisions, Action Items, and System Items databases).
3. Fetch and display the full page content in the session.
4. Add the current discussion to the record's **References** relation in Notion.
5. Confirm: display the linked record title and ID.

---

### `/da conclude`

Finalize the current discussion.

1. Require an open discussion.
2. Fetch the current discussion page from Notion.
3. Check the Conclusion section — if empty, synthesize a conclusion from the discussion content (summary, key points, decisions, TODOs, risks, open questions). Present it to the user and ask: "Does this conclusion work, or would you like to edit it?" Wait for confirmation or a revised version before proceeding. If the user provides a replacement, use it as-is. The user may also skip (leave blank).
4. Update the Notion page: Status=Concluded, Date Concluded=today.
5. Append to the local transcript and update the header:
   ```
   **[date]** *Concluded*
   [conclusion text]
   ```
   Update header line: `**Status:** Concluded`
6. Fetch all Action Items linked to the discussion where Type=QUESTION and Status=Open. If any exist, list them at the end of the output: `Open questions: QST-NNN — ..., QST-NNN — ...`
7. Clear `current_discussion` from session context.
8. Confirm: display DISC-NNN ID and local transcript file path.

---

### `/da abandon "[reason]"`

Close the current discussion without concluding.

- **If reason is provided**: use it as-is.
- **If reason is omitted**: prompt "Why are you abandoning this discussion?" and use the answer as the reason. No auto-generation — the user's answer is used directly.

1. Require an open discussion.
2. Update the Conclusion callout on the Notion page with the reason text.
3. Update Notion: Status=Abandoned.
4. Append to the local transcript and update the header:
   ```
   **[date]** *Abandoned — [reason]*
   ```
   Update header line: `**Status:** Abandoned`
5. Clear `current_discussion` from session context.
6. Confirm.

---

## Viewing / Querying

### `/da show [id]`

Fetch and display the full page content of any record — decision, todo, risk, point, question, discussion, part, component, or software. Search Notion by the ID value across all project databases (Discussions, Decisions, Action Items, System Items).

---

### `/da decisions`

List all decisions for the active project. Output: ID, title, status, date — one per line.

---

### `/da todos`

List all open TODO action items for the active project. Output: ID, title, priority, date.

---

### `/da risks`

List all open RISK action items for the active project. Output: ID, title, priority, date.

---

### `/da points`

List all POINT action items for the active project. Output: ID, title, date — one per line.

---

### `/da questions`

List all open QUESTION action items for the active project. Output: ID, title, date — one per line.

---

### `/da discussions`

List all discussions for the active project (open and concluded). Output: DISC-NNN, title, status, date opened.

---

### `/da parts`

List all Part system items for the active project. Output: ID, title, status, category — one per line.

---

### `/da components`

List all Component system items for the active project. Output: ID, title, status — one per line.

---

### `/da softwares`

List all Software system items for the active project. Output: ID, title, status — one per line.

---

## Quick Reference

| Command | Action |
|---------|--------|
| `/da init "[name]"` | Bootstrap global structure (first run) or create new project |
| `/da open "[name]"` | Switch active project (partial name match) |
| `/da open` | Reopen last active project |
| `/da close` | Deactivate the current project |
| `/da projects` | List all projects |
| `/da config` | Show active project state and counters |
| `/da new "[title]"` | Open a new discussion |
| `/da new about [ID]` | Open a discussion anchored to an existing record |
| `/da status` | Show current discussion and linked records |
| `/da point "[text]"` | Create a Point linked to current discussion |
| `/da question "[text]"` | Create a Question linked to current discussion |
| `/da decision "[title]"` | Create a decision linked to current discussion |
| `/da todo "[text]"` | Create a TODO linked to current discussion |
| `/da risk "[text]"` | Create a RISK linked to current discussion |
| `/da part "[name]"` | Create a Part linked to current discussion |
| `/da component "[name]"` | Create a Component linked to current discussion |
| `/da software "[name]"` | Create a Software item linked to current discussion |
| `/da ref [id]` | Load record into context + link to current discussion |
| `/da conclude` | Finalize discussion; write local transcript |
| `/da abandon "[reason]"` | Close discussion without concluding |
| `/da show [id]` | Display full record content |
| `/da decisions` | List all decisions |
| `/da todos` | List open TODOs |
| `/da risks` | List open RISKs |
| `/da points` | List all Points |
| `/da questions` | List open Questions |
| `/da discussions` | List all discussions |
| `/da parts` | List all Parts |
| `/da components` | List all Components |
| `/da softwares` | List all Software items |
