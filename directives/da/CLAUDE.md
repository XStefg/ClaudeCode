# Design Assistant (DA) System

@DA-SYSTEM.md
@DA-COMMANDS.md

## Instructions

You are Design Assistant-aware in every session. Follow these rules at all times:

- Recognize and handle all `/da` commands as defined in `DA-COMMANDS.md`.
- Follow the rules in `DA-SYSTEM.md` strictly.

### State loading on every `/da` command

Before executing any `/da` command:
1. Read `~/dev/.da/config.json` — global state (root Notion IDs, project registry).
   - If this file does not exist, the workspace has not been initialized. Inform the user to run `/da init "[project-name]"` first.
2. Resolve the active project: use the session-local `active_project` if already set in session context; otherwise there is no active project — commands that require one must error and ask the user to run `/da open "[project-name]"` first.
3. Resolve the active project path from `projects[active_project]`.
4. Read `[project-path]/.da/config.json` — per-project state (Notion IDs, counters). Neither `current_discussion` nor `active_project` is stored on disk — both live in session context only.
   - Exception: skip steps 2–4 for `/da init` (first-run bootstrap) and `/da projects`.

### Notion operations

Use the `notionApi` MCP tool for **all** Notion reads and writes. Never construct Notion page URLs or IDs by hand — always retrieve them from the config files.

### Template fetch rule

Before creating any record (discussion, decision, TODO, risk, point, question, or new project page):
1. Fetch the relevant template page from the global Templates page (ID in global config: `notion_templates_page_id`).
2. Parse its sections (headings + placeholder text).
3. For each section whose placeholder is not yet filled in:
   - If the placeholder starts with `[later:` (case-insensitive), skip it — create the heading with no body content, without prompting.
   - Otherwise, prompt the user for the value — one section at a time.
4. The user may skip any non-`[later:]` section — create the page with the heading present but no body content for that section.
5. Create the Notion page with whatever content was provided.

This fetch-and-prompt behavior applies identically to all record types.

### Counter management

Counters (`discussion_counter`, `decision_counter`, `todo_counter`, `risk_counter`) live in the per-project `.da/config.json`. Increment the relevant counter **immediately after** successfully creating the Notion record, then write the updated config back to disk. IDs are assigned at record-creation time.

### `/da init` — first-run detection

If `~/dev/.da/config.json` does not exist:
- This is a first-run bootstrap. Ask the user for a **Notion parent page ID**.
- Create the full global Notion structure (root page, Templates page + 5 template pages, Projects database).
- Write `~/dev/.da/config.json` with the resulting IDs.
- Then proceed to create the first project entry as described in the command spec.

If `~/dev/.da/config.json` already exists:
- Skip all prompts. Reuse the existing global structure.
- Proceed directly to creating a new project entry.

### Active project management

- `active_project` is held in **session context only** — never stored on disk. Each session starts with no active project. The user must run `/da open` to set one.
- `/da open` updates session-local `active_project` and writes `last_project` to disk (for use as a default hint by subsequent sessions). It does NOT write `active_project` to disk.
- `last_project` lives only on disk — it is a cross-session convenience hint, updated whenever the user switches projects. `/da open` (no args) uses it as the default.
- Always write config changes back to disk immediately after making them.
