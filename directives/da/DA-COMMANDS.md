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
   - "Templates" child page with 7 template pages (Project Page, Discussion, Decision, TODO, Risk, Point, Question) — populated with default content from DA-SYSTEM.md
   - "Projects" database with properties: Name (title), Status (select), GitHub URL (url), Created (date)
3. Write `~/dev/.da/config.json` with all resulting Notion IDs.
4. Proceed to create the first project entry (step below).

**Subsequent runs:**
1. Create a new entry in the Projects database (Status=Active, Created=today).
2. Create the project's Notion page (fetch Project Page Template, prompt for unfilled sections).
3. Create 3 child databases inside the project page: Discussions, Decisions, Action Items — with the properties defined in DA-SYSTEM.md.
4. Create the local directory structure:
   - `[project-path]/.da/`
   - `[project-path]/.da/discussions/`
   - `[project-path]/.da/config.json`
5. Register the project in `~/dev/.da/config.json → projects`. Set session-local `active_project` to the new project slug. Do NOT write `active_project` to disk.

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
- Record counters (discussions, decisions, TODOs, risks, points, questions)

---

## Discussion Lifecycle

### `/da new "[title]"`

Open a new discussion.

1. Check that no discussion is currently open (error if one is).
2. Fetch the global Discussion Template from Notion.
3. Apply the `[later:]` rule: prompt only for non-`[later:]` sections (Summary). Skip all others.
4. Create the Notion page in the active project's Discussions database (Status=Open, Date Opened=today).
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
2. Look up the record by ID in Notion (any type: DA-NNN, TODO-NNN, RISK-NNN, PT-NNN, Q-NNN, DISC-NNN).
3. Fetch and display the record's full content in the session — the discussion starts with full context.
4. Auto-generate the title: `[ID] — [record title]` (no prompt).
5. Fetch the global Discussion Template from Notion.
6. Apply the `[later:]` rule: prompt only for non-`[later:]` sections (Summary). Skip all others.
7. Create the Notion page in the Discussions database (Status=Open, Date Opened=today).
8. Inject a **Regarding** section at the top of the discussion page (above Summary) with a Notion page mention/link to the source record:
   ```
   ## Regarding
   → [ID] — [record title]
   ```
   This section is not part of the standard Discussion Template — injected only for `about` discussions.
9. Link the record:
   - Add the new discussion to the record's **References** relation in Notion.
   - Append a linked bullet to the appropriate section of the discussion page based on record type (RISK → Risks, Q-NNN → Open Questions, TODO → TODOs, PT-NNN → Key Points, DA-NNN → Decisions).
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

Show the current open discussion:
- DISC-NNN title, date opened
- Decisions linked so far (IDs + titles)
- Action items linked so far (IDs + titles)

If no discussion is open, say so.

---

### `/da point "[text]"`

Create a Point action item linked to the current discussion.

1. Require an open discussion (error if none).
2. Fetch the global Point Template from Notion.
3. Prompt for each non-`[later:]` section (Point, Context).
4. Increment `point_counter`. Assign ID `PT-[NNN]`.
5. Create the Notion page in the Action Items database with:
   - Type = POINT, Status = Active
   - Discussion relation = current discussion page
   - Fill the `Raised in` section with a Notion page mention/link to the current discussion.
6. Append a linked bullet to the `Key Points` section of the current discussion page: `- PT-NNN — [title]`.
7. Append to the local transcript: `**[date]** PT-NNN — [title]`
8. Write updated per-project config.
9. Confirm: display PT-NNN and Notion page link.

---

### `/da question "[text]"`

Create a Question action item linked to the current discussion.

1. Require an open discussion (error if none).
2. Fetch the global Question Template from Notion.
3. Prompt for each non-`[later:]` section (Question, Context).
4. Increment `question_counter`. Assign ID `Q-[NNN]`.
5. Create the Notion page in the Action Items database with:
   - Type = QUESTION, Status = Open
   - Discussion relation = current discussion page
   - Fill the `Raised in` section with a Notion page mention/link to the current discussion.
6. Append a linked bullet to the `Open Questions` section of the current discussion page: `- Q-NNN — [title]`.
7. Append to the local transcript: `**[date]** Q-NNN — [title]`
8. Write updated per-project config.
9. Confirm: display Q-NNN and Notion page link.

---

### `/da decision "[title]"`

Create a decision linked to the current discussion.

1. Require an open discussion (error if none).
2. Fetch the global Decision Template from Notion.
3. Prompt for each non-`[later:]` section (Summary, Context, Rationale, Consequences).
4. Increment `decision_counter`. Assign ID `DA-[NNN]`.
5. Create the Notion page in the Decisions database with:
   - ID property = `DA-[NNN]`
   - Discussion relation = current discussion page
   - Status = Active
   - Date = today
   - Fill the `Discussion` section with a Notion page mention/link to the current discussion.
6. Append a linked bullet to the `Decisions` section of the current discussion page: `- DA-NNN — [title]`.
7. Append to the local transcript: `**[date]** DA-NNN — [title]`
8. Write updated per-project config.
9. Confirm: display DA-NNN and Notion page link.

---

### `/da todo "[text]"`

Create a TODO action item linked to the current discussion.

1. Require an open discussion (error if none).
2. Fetch the global TODO Template from Notion.
3. Prompt for each non-`[later:]` section (Description, Acceptance Criteria, Notes).
4. Increment `todo_counter`. Assign ID `TODO-[NNN]`.
5. Create the Notion page in the Action Items database with:
   - Type = TODO, Status = Open
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
3. Prompt for each non-`[later:]` section (Description, Impact, Likelihood, Mitigation).
4. Increment `risk_counter`. Assign ID `RISK-[NNN]`.
5. Create the Notion page in the Action Items database with:
   - Type = RISK, Status = Open
   - Discussion relation = current discussion page
   - Fill the `Raised in` section with a Notion page mention/link to the current discussion.
6. Append a linked bullet to the `Risks` section of the current discussion page: `- RISK-NNN — [title]`.
7. Append to the local transcript: `**[date]** RISK-NNN — [title]`
8. Write updated per-project config.
9. Confirm: display RISK-NNN and Notion page link.

---

### `/da ref [id]`

Load a record into context and formally link it to the current discussion.

`[id]` may be any record ID: `DA-NNN`, `DISC-NNN`, `TODO-NNN`, `RISK-NNN`, `PT-NNN`, or `Q-NNN`.

1. Require an open discussion (error if none).
2. Look up the record in Notion (search by ID property or title).
3. Fetch and display the full page content in the session.
4. Add the current discussion to the record's **References** relation in Notion.
5. Confirm: display the linked record title and ID.

---

### `/da conclude`

Finalize the current discussion.

1. Require an open discussion.
2. Fetch the current discussion page from Notion.
3. Check the Conclusion section — if empty, prompt the user to fill it in (or skip).
4. Update the Notion page: Status=Concluded, Date Concluded=today.
5. Append to the local transcript and update the header:
   ```
   **[date]** *Concluded*
   [conclusion text]
   ```
   Update header line: `**Status:** Concluded`
6. Fetch all Action Items linked to the discussion where Type=QUESTION and Status=Open. If any exist, list them at the end of the output: `Open questions: Q-NNN — ..., Q-NNN — ...`
7. Clear `current_discussion` from session context.
8. Confirm: display DISC-NNN ID and local transcript file path.

---

### `/da abandon "[reason]"`

Close the current discussion without concluding.

1. Require an open discussion.
2. Append the reason to the discussion page body in Notion.
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

Fetch and display the full page content of any record — decision, todo, risk, point, question, or discussion. Search Notion by the ID value in the relevant database.

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

## Quick Reference

| Command | Action |
|---------|--------|
| `/da init "[name]"` | Bootstrap global structure (first run) or create new project |
| `/da open "[name]"` | Switch active project (partial name match) |
| `/da open` | Reopen last active project |
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
