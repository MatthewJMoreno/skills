---
name: design-doc
description: >
  Generates a structured Markdown design document by reading the current conversation history.
  Intended to work in conjunction with the grill-me skill — after grill-me has teased out
  implementation details through Q&A, this skill synthesizes everything discussed into a
  formal design doc. Trigger this skill when the user says "write a design doc", "summarize
  this into a design doc", or when a grill-me session has just ended and the user wants to
  capture the output. Also trigger when the user says things like "turn this into a doc",
  "capture what we discussed", or "I'm ready to write this up".
---
 
# Design Doc Skill
 
## Purpose
 
Read the full conversation history and extract all relevant technical decisions, goals, and
details discussed — then render a clean Markdown design doc the user can copy, save, or
hand off.
 
This skill is designed to pair with **grill-me**: grill-me surfaces implementation details
through interrogation; design-doc captures them into a structured artifact.
 
---
 
## Default Tech Stack (do NOT list unless something new is being added)
 
The assumed baseline stack is:
- **Backend**: Java, Spring Boot
- **Frontend**: Angular, TypeScript
- **Database**: PostgreSQL
Only include a **Tech Stack** section in the output if the conversation introduces something
outside this baseline (e.g. Kafka, Redis, a new AWS service, a third-party API, etc.).
 
---
 
## Output Format
 
Render the design doc as a Markdown code block in chat. Do not create a file unless the
user asks.
 
---
 
## Document Structure
 
Produce only the sections for which there is actual content from the conversation.
Do not include empty or placeholder sections.
 
### 1. Goal / Problem Statement
What is being built and why. One short paragraph. Pull from the user's initial description
or framing in the conversation.
 
### 2. Design Details / Architecture
How the system is structured. Include:
- Key components and their responsibilities
- Data flow or sequence if discussed
- API contracts or data models if mentioned
- Any diagrams described verbally (render as a simple bullet structure or ASCII if useful)
### 3. Implementation Plan
High-level phases only. Format as an ordered list of phases with a one-line description each.
Example:
1. **Phase 1 – Data Layer**: Set up schema and repository layer
2. **Phase 2 – API**: Build REST endpoints
3. **Phase 3 – Frontend**: Wire up Angular components
### 4. Decisions Made
Capture explicit or implied decisions from the conversation, especially ones where
alternatives were considered. Format as a table:
 
| Decision | Rationale |
|----------|-----------|
| ...      | ...       |
 
### 5. Open Questions / Risks
Anything unresolved, flagged as uncertain, or identified as a risk during the conversation.
Bullet list.
 
### 6. Tech Stack (conditional)
Only include this section if the conversation introduces something outside the baseline stack.
List only the additions/changes with a brief note on why they're being used.
 
---
 
## Behavior Notes
 
- **Be extractive, not inventive.** Only include what was actually discussed. Do not infer
  or invent details not present in the conversation.
- **Prefer the user's own words** for goals and decisions where possible.
- **If a section has no content**, skip it entirely — don't include it with "TBD".
- **If the conversation is still mid-grill-me**, note at the top of the doc:
  > ⚠️ This doc was generated mid-session. Some sections may be incomplete.
- After rendering the doc, ask: *"Anything missing or that you'd like to adjust?"*
