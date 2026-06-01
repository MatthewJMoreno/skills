---
name: personal-scribe
description: >
  Persistent knowledge base for software developer consultants embedded on client teams.
  Stores and recalls internal APIs, links, contacts, business processes, frameworks, and
  anything else worth remembering — all in a local JSON file. ALWAYS activate this skill
  whenever the user's message begins with /kb, without exception. The /kb prefix is the
  sole and definitive trigger. Every /kb message — whether adding, querying, updating,
  switching teams, or managing the file — must go through this skill.
---

# Personal Scribe

A local-file knowledge base for consultant developers. Triggered exclusively by `/kb`.
Everything after `/kb` is natural language — interpret intent from context.

---

## File Location & Initialization

**Default path**: `/users/<username>/llm-artifacts/<filename>.json`

### First-Run Flow (no file found)

1. Ask for their **username**
2. Ask for a **filename** (default: `scribe.json`)
3. Create the file at the resolved path
4. Initialize with this skeleton:

```json
{
  "meta": {
    "active_team": null,
    "types": ["link", "contact", "process", "api", "note"],
    "created": "<ISO date>",
    "last_updated": "<ISO date>"
  },
  "global": [],
  "teams": {}
}
```

Once initialized, proceed with the original intent without making the user re-issue the command.

---

## Entry Shape

Every entry in `global` or any team array uses this shape:

```json
{
  "id": 1,
  "type": "link",
  "title": "short label",
  "description": "what it is / why it matters",
  "note": "optional freeform context, caveats, gotchas",
  "tags": ["tag1", "tag2"],
  "date_added": "YYYY-MM-DD"
}
```

- `id`: globally incrementing integer across the entire file (never reused)
- `type`: one of the values in `meta.types`
- `note`: optional — omit or set null if not relevant
- `tags`: lowercase strings

---

## Intent Routing

Parse the natural language after `/kb` and route to the correct action:

| Pattern | Action |
|---|---|
| `add ... global` / `add this to global` | → **Add to global** |
| `add to team` / `add ...` (no team named) | → **Add to active team** |
| `add this to Team X` | → **Add to named team** (don't change active team) |
| `I'm on Team X` / `switch to Team X` / `moving to Team X` | → **Switch active team** |
| `create team X` / `new team called X` | → **Create team** |
| `show everything for Team X` / `list Team X` | → **List team entries** |
| `update entry N` / `entry N changed` | → **Update entry by ID** |
| Any question (`do I have...`, `what do I know about...`, `who is...`) | → **Query / search** |

---

## Actions

### Add Entry

1. Read the file
2. Determine destination:
   - Explicit "global" → `global`
   - Explicit "Team X" → `teams.X` (do NOT update `active_team`)
   - "add to team" / ambiguous → use `meta.active_team`
   - `active_team` is null and no team named → **ask which team** before proceeding
3. Infer `type` from content. If no existing type fits:
   - Propose: *"This doesn't fit an existing type — I'd create `<typename>`. Confirm?"*
   - On confirmation: add to `meta.types`, use for this entry
   - Truly ambiguous → use `note`
4. Check for duplicates (same URL, name, or near-identical description). If found:
   - Warn: *"I already have something similar: [entry title] (ID N). Add anyway?"*
5. Assign next global ID, set `date_added` to today
6. Write back to file, update `meta.last_updated`
7. Confirm in one sentence: *"Saved '[title]' as a [type] under [global/Team X] (ID N)."*

### Switch Active Team

1. Read the file
2. Verify team exists in `teams`; if not → *"Team X doesn't exist yet — want me to create it?"*
3. Update `meta.active_team`, write file
4. Confirm: *"Switched to Team X. That's your active team now."*

### Create Team

1. Read the file
2. If team already exists → say so
3. Add `teams.X = []`, write file
4. Optionally ask if they want to switch to it: *"Team X created. Want to switch to it now?"*

### Query / Search

1. Read the file
2. Scope the search:
   - If a team name is mentioned → search only that team
   - Otherwise → search `global` + all teams
3. Match against title, description, note, tags, type (case-insensitive, substring OK)
4. Answer **conversationally** — like a knowledgeable colleague, not a database dump
5. Reference entry titles, links, or IDs naturally where helpful
6. If nothing found: say so plainly, offer to add it

### List Team / Global

- Show all entries for the requested scope
- Format as a readable list: `[ID] [type] — [title]: [description]`
- Group by type if there are many entries

### Update Entry

1. Read the file
2. Find entry by ID (search all sections)
3. Confirm what was found: *"Found entry N — '[title]'. Updating [field]..."*
4. Apply the change, update `meta.last_updated`, write file
5. Confirm: *"Updated entry N."*
6. If ID not found → say so, ask for clarification

---

## Tone & Style

- Respond like a helpful, efficient personal assistant — not a terminal
- Adding an entry → one short confirmation sentence
- Answering a query → lead with the answer, then supporting detail
- Never dump raw JSON to the user
- Be concise; skip filler phrases

---

## Edge Case Handling

| Situation | Response |
|---|---|
| File not found | Trigger initialization flow, then resume original intent |
| `active_team` is null, user says "add to team" | Ask which team before proceeding |
| Named team doesn't exist | *"Team X doesn't exist yet — want me to create it?"* |
| Near-duplicate entry detected | Warn with existing entry details, ask "Add anyway?" |
| Unknown type with no clear name | Propose a name, confirm before creating |
| Entry ID not found on update | Say so, ask for clarification |
| File path inaccessible | Report the error, ask user to verify path |
