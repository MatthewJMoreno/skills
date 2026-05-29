---
name: todo-manager
description: >
  A persistent todo list manager that reads and writes tasks to a JSON file on disk.
  Use this skill whenever the user wants to manage tasks or a to-do list — including:
  adding new items, setting or changing priorities, checking what's left to do,
  marking tasks complete (with optional notes), or asking anything like "what's on
  my list?", "what do I still need to do?", "add a task", "mark X as done", or
  "show my todos". Trigger proactively even for casual phrasing like "don't let me
  forget to...", "I need to do X", or "can you remind me about Y".
---

# Todo Manager Skill

Manage a persistent todo list stored in a JSON file. Always read the file before any
operation, apply changes, then write it back.

## Data File

Default path: `/mnt/user-data/uploads/todos.json`
If not found there, use: `~/todos.json` (i.e. `/root/todos.json` or `/home/claude/todos.json`)

If the file doesn't exist yet, create it with an empty task list.

## JSON Schema

```json
{
  "tasks": [
    {
      "id": 1,
      "title": "Buy groceries",
      "priority": "medium",
      "status": "pending",
      "created_at": "2026-05-28T10:00:00",
      "completed_at": null,
      "notes": null
    }
  ],
  "next_id": 2
}
```

**Fields:**
- `id`: Auto-incrementing integer (use `next_id`, then increment it)
- `title`: Task description string
- `priority`: `"high"`, `"medium"`, or `"low"` (default: `"medium"`)
- `status`: `"pending"` or `"done"`
- `created_at`: ISO 8601 timestamp when task was added
- `completed_at`: ISO 8601 timestamp when marked done, else `null`
- `notes`: Optional string added when completing, else `null`

## Operations

### Add a task
1. Read the JSON file (or create if missing)
2. Build a new task object with the next available `id`, set `status: "pending"`, record `created_at`
3. Append to `tasks`, increment `next_id`
4. Write back to file
5. Confirm: "Added **{title}** (#{id}) with {priority} priority."

### Set / change priority
1. Read the file
2. Find the task by ID or by matching title (case-insensitive substring match; if ambiguous, ask)
3. Update `priority`
4. Write back
5. Confirm: "Updated **{title}** to {priority} priority."

### What's left on my list? / Show pending tasks
1. Read the file
2. Filter `status == "pending"`
3. Sort: high → medium → low, then by `id` ascending within each group
4. Display as a clean list (use the widget below if available, otherwise plain text)

**Display format (plain text fallback):**
```
📋 Your Todo List (3 tasks)

🔴 HIGH
  #2 · Call the dentist

🟡 MEDIUM
  #1 · Buy groceries
  #4 · Reply to emails

🟢 LOW
  #3 · Reorganize bookshelf
```

### Mark task as complete
1. Read the file
2. Find the task by ID or title match
3. Set `status: "done"`, record `completed_at`, store optional `notes`
4. Write back
5. Confirm: "✅ Marked **{title}** as done." (include notes if provided)

## Edge Cases

- **File not found**: Create a fresh `todos.json` at `~/todos.json`, inform the user
- **No tasks**: "Your todo list is empty! Add something with 'add task: ...' "
- **Ambiguous title match**: Show the matching tasks and ask which one
- **Already done**: "**{title}** is already marked complete."
- **Invalid priority**: Default to `"medium"` and mention it

## Always

- Read the file fresh before every operation (never rely on in-memory state from a prior turn)
- Write back after every mutation
- Keep responses concise — one confirmation line is enough for add/update/complete
- When showing the list, always show the file path so the user knows where it lives
