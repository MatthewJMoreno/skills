---
name: ubiquitous-language
description: >
  Documents the ubiquitous language (domain glossary) of a codebase into a structured
  UBIQUITOUS_LANGUAGE.md file that both humans and LLMs can use as shared context.
  Trigger this skill whenever the user wants to document domain terms, create a project
  glossary, establish shared vocabulary, improve LLM code session quality with domain
  context, or says things like "document our domain terms", "create a glossary",
  "ubiquitous language", "domain vocabulary", or "help the LLM understand our codebase".
  Also trigger proactively at the end of any conversation where significant domain-specific
  terms have emerged and no UBIQUITOUS_LANGUAGE.md exists yet.
---
 
# Ubiquitous Language Documenter
 
Helps teams document the shared vocabulary of their codebase into a single
`UBIQUITOUS_LANGUAGE.md` file. The file serves dual purpose: a reference for
humans, and high-signal context for LLMs in coding sessions.
 
---
 
## Entry Format
 
Every term uses this exact structure:
 
```markdown
## TermName
 
**Definition:** What this term means in this domain. Be precise. Avoid circular definitions.
 
**Aliases:** Other names this thing goes by (e.g. `picture`, `photo`, `apod`). Write "None" if none.
 
**Related Terms:** Comma-separated links to other entries (e.g. `MediaType`, `ApodDate`). Write "None" if none.
 
**Notes:** Edge cases, gotchas, API specifics, or anything that would trip up a new developer
or an LLM. Write "None" if none.
```
 
Rules:
- `##` heading is the canonical term name — use the exact casing used in code
- All four fields must be present, even if the value is "None"
- Keep definitions to 2–3 sentences max
- Notes is where nuance lives — don't cram it into Definition
---
 
## Workflow
 
### Step 0 — Check for existing file
 
Look for `UBIQUITOUS_LANGUAGE.md` in the repo root (or current working directory).
 
- **File exists** → skip to [Append Mode](#append-mode)
- **No file** → proceed with [Discovery Pipeline](#discovery-pipeline)
---
 
### Discovery Pipeline (fresh file)
 
Run all three discovery sources before showing the user anything. The goal is
to arrive at a single unified candidate list.
 
#### Source 1 — Conversation History
 
Search past conversations for domain terms. Scope to the current project if
one exists; fall back to all conversations if not. Tell the user which scope
you're using.
 
Use `conversation_search` with queries like:
- The project name (e.g. `nasa apod`, `apod app`)
- Technical terms that appeared in this conversation
- API or model names mentioned
Look for: nouns used with domain specificity, terms defined mid-conversation,
things the user corrected or clarified, API field names, entity names.
 
Collect candidates with any context clues about their meaning.
 
#### Source 2 — Codebase Scan
 
If a codebase is accessible, scan for candidate terms:
 
```bash
# Find class/interface/type/enum names
grep -rE "^(class|interface|type|enum)\s+\w+" --include="*.ts" --include="*.tsx" \
  --include="*.js" --include="*.py" --include="*.kt" --include="*.swift" \
  -h . | grep -v node_modules | grep -v ".git" | sort -u
 
# Find exported types/models
grep -rE "export (type|interface|class|enum)\s+\w+" --include="*.ts" \
  -h . | grep -v node_modules | sort -u
```
 
Adapt globs to the project's language. Also scan:
- API response type definitions
- Route/endpoint names
- Database model/schema names
- Config keys with domain meaning
Ignore: generic names (`Utils`, `Helper`, `Manager`, `Config`, `Base`), framework
boilerplate, and anything that's clearly infrastructure rather than domain.
 
#### Source 3 — Speculative Interview
 
After sources 1 and 2, probe the user for terms that don't appear in code yet
but should. Use high-temperature creative probing — make speculative suggestions
even at low confidence. Some will miss; the ones that land surface the most
valuable entries.
 
Good probing questions (ask a few, not all):
- "Are there any terms that mean one thing in the UI and something different in the API response?"
- "Is there anything your team calls X but the third-party API calls Y?"
- "Are there any states or statuses a `[core entity]` can be in?"
- "Do any terms have different meanings depending on context?"
- "Is there anything a new developer always gets confused about?"
- "Are there domain concepts you talk about in planning that don't appear in the code yet?"
- "Does `[term from codebase]` have any edge cases or gotchas worth capturing?"
Speculate freely: *"I noticed `mediaType` in your code — does it behave differently
for video vs image responses from NASA? That might be worth a Note."*
 
#### Merge and Deduplicate
 
Combine all candidates from sources 1–3:
- Merge duplicates (same term, different sources) into one entry
- Use conversation context to pre-fill fields where possible
- Mark fields as `[needs clarification]` where meaning is uncertain
- Sort alphabetically
---
 
### Confirmation Pass
 
Present the full candidate list to the user **before writing anything**. Format:
 
```
I found N candidate terms across your conversation history, codebase, and our discussion.
Here's what I'm proposing for UBIQUITOUS_LANGUAGE.md — review each entry and let me know
what to change, remove, or add before I write the file.
 
---
 
## Apod
 
**Definition:** The Astronomy Picture of the Day for a given date, including its title,
media, and explanation as returned by the NASA APOD API.
 
**Aliases:** `picture`, `astronomy-picture`
 
**Related Terms:** `MediaType`, `ApodDate`
 
**Notes:** [needs clarification] — does this term refer to the raw API response,
the normalized model, or both?
 
---
[... remaining entries ...]
```
 
Wait for the user to confirm, correct, or add entries. Iterate in conversation
until they say they're happy. Then write the file.
 
---
 
### Write the File
 
```markdown
# Ubiquitous Language
 
> Shared vocabulary for this codebase. Include this file as context in LLM coding
> sessions to improve reasoning about domain concepts.
>
> Last updated: YYYY-MM-DD
 
---
 
[entries in alphabetical order]
```
 
Write to `UBIQUITOUS_LANGUAGE.md` in the repo root (or working directory).
Confirm the path with the user if ambiguous.
 
---
 
## Append Mode
 
When `UBIQUITOUS_LANGUAGE.md` already exists:
 
1. Read the file to understand existing entries
2. Ask the user: *"Which term(s) do you want to add or update?"*
3. If they're not sure, run a lightweight version of the discovery pipeline:
   - Scan recent conversations for terms not yet in the file
   - Scan codebase for new class/type names not yet documented
   - Ask 2–3 speculative interview questions
4. Preview the new/updated entries
5. Append to the file (or update in-place for edits)
6. Update the `Last updated` date in the header
---
 
## Quality Checks
 
Before finalizing any entry, verify:
- [ ] Definition doesn't use the term to define itself
- [ ] Aliases includes any name this thing goes by in third-party APIs
- [ ] Related Terms links are accurate (the linked terms actually exist or will exist in the file)
- [ ] Notes captures anything that would trip up an LLM (e.g. overloaded terms, API quirks, enum values)
---
 
## Tips for Good Entries
 
**Definitions** should answer: *"If I handed this object to a new developer with no context, what would they need to know?"*
 
**Notes** is where the real value lives for LLMs. Be specific:
- ✅ "MediaType is either `'image'` or `'video'` — NASA occasionally returns YouTube URLs for video entries, not direct media files"
- ❌ "Can be different types"
**Aliases** should include third-party names, old names, and colloquial names the team uses in conversation even if not in code.
 
