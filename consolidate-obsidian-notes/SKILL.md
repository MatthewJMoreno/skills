---
name: obsidian-consolidator
description: >
  Consolidates, deduplicates, and reorganizes notes in an Obsidian vault directory.
  Use this skill whenever the user mentions Obsidian, a notes vault, messy notes,
  duplicate notes, or asks to "clean up", "consolidate", "organize", or "merge" 
  their markdown notes. Also trigger when the user points Claude at a directory of 
  .md files and wants help structuring or reducing them. The skill reads notes 
  non-destructively, analyzes intent and overlap, then generates new/merged files 
  without modifying the originals.
---

# Obsidian Vault Consolidator

Analyze a target directory of Obsidian `.md` notes, determine which notes overlap
or can be merged, identify a source of truth for each topic, and output new
consolidated files — never touching the originals.

---

## Step 1 — Scan the Target Directory

```bash
find <target_dir> -name "*.md" | sort
```

List all `.md` files. Note folder structure — it often signals intent (e.g., `daily/`, `projects/`, `references/`).

---

## Step 2 — Read and Parse Each Note

For each file, extract:

| Field | How to get it |
|---|---|
| **Frontmatter** | YAML between `---` fences at top of file |
| **Tags** | frontmatter `tags:` field + inline `#tag` usage |
| **Wikilinks** | `[[Note Name]]` or `[[Note Name\|alias]]` patterns |
| **Headings** | `#`, `##`, `###` — reveals note's internal structure |
| **Core intent** | 1-sentence summary: what is this note *for*? |

Keep originals untouched. Work from in-memory representations only.

---

## Step 3 — Cluster by Topic

Group notes that share:
- Overlapping headings or keywords
- Same or similar tags
- Wikilinks pointing at each other
- Nearly identical titles (e.g., `Auth.md`, `Authentication.md`, `auth-notes.md`)

Label each cluster with a topic (e.g., `"Authentication"`, `"Project X Planning"`).

---

## Step 4 — Identify Source of Truth per Cluster

For each cluster, pick the **source of truth** note using this priority:
1. Most complete (most headings, most content)
2. Most recently modified (check file metadata if available)
3. Most linked-to by other notes (highest incoming wikilink count)

The others are **satellites** — they may contain unique content to absorb, or they may simply be redundant.

---

## Step 5 — Produce a Consolidation Plan

Before writing any files, **present the plan** to the user in this format:

```
## Consolidation Plan

### Cluster: <Topic Name>
- Source of Truth: `<filename>.md`
- Satellites: `a.md`, `b.md`
- Action: Merge unique content from satellites into source of truth
- Unique content to absorb: <brief description>
- Satellites become: stubs with [[link]] to consolidated note

### Standalone Notes (no overlap found)
- `note1.md`, `note2.md` — no action needed
```

Ask: *"Does this plan look right? Want me to adjust anything before I generate the files?"*

Wait for confirmation before proceeding.

---

## Step 6 — Generate Consolidated Output Files

Output directory: `<target_dir>/consolidated/` (create if needed)

### For each Source of Truth note:
1. Copy its frontmatter verbatim
2. Merge in unique content from satellites (add under appropriate headings, or append a `## Additional Notes` section)
3. Preserve all `[[wikilinks]]`, `#tags`, and heading structure from all merged notes
4. Add a frontmatter field: `consolidated_from: [satellite1.md, satellite2.md]`
5. Write to `consolidated/<source-of-truth-filename>.md`

### For each Satellite note:
Generate a stub at `consolidated/<satellite-filename>.md`:
```md
---
<original frontmatter>
consolidated_into: "[[<source-of-truth-note>]]"
---

> This note has been consolidated into [[<source-of-truth-note>]].
```

### For standalone notes:
Copy as-is into `consolidated/` — no changes.

---

## Step 7 — Output a Summary Report

After writing all files, print:

```
## Consolidation Complete

Output: <target_dir>/consolidated/

| Action | Count |
|---|---|
| Merged clusters | N |
| Notes consolidated | N |
| Stubs created | N |
| Standalone (unchanged) | N |

### Files written:
- consolidated/AuthFlow.md  (merged from: auth.md, auth-notes.md)
- consolidated/auth.md  (stub → [[AuthFlow]])
...
```

---

## Obsidian-Specific Rules

- **Never break `[[wikilinks]]`** — if a satellite becomes a stub, the stub file must still exist so links don't go red in the graph view
- **Preserve frontmatter keys** — merge unique keys; on conflicts, prefer the source of truth's value
- **Tags** — union all tags from merged notes (dedup)
- **Aliases** — union all `aliases:` frontmatter values across merged notes
- **Do not add new wikilinks** that didn't exist in the originals unless explicitly asked

---

## Edge Cases

| Situation | Behavior |
|---|---|
| Two notes equally "complete" | Ask the user which to prefer as source of truth |
| Note has no headings or tags | Treat as atomic — include in standalone unless title overlap exists |
| Nested subdirectories | Recurse unless user said target a specific folder only |
| Note is a daily journal entry (`YYYY-MM-DD.md`) | Skip from clustering — treat always as standalone |
