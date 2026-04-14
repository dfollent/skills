---
name: project-lint
description: "Audit a project's document tree for quality, consistency, and structure. Use this skill when someone wants to: health-check a project, find stale or unresolved items, identify contradictions between documents, retrofit tagging conventions onto legacy docs, assess whether documents should be split, or review the overall state of a project's documentation. Triggers include: 'lint the project', 'health-check this project', 'audit the docs', 'what's stale', 'find contradictions', 'retrofit tags', 'is this doc too big', 'review the project structure'. Use this even when the user says something like 'clean up the project docs' or 'what's wrong with our docs'."
---

# Project Lint

Audit a project's document tree on the team's shared surface (currently Confluence). The goal: surface inconsistencies, staleness, implicit items, structural issues, and convention drift — so that the project wiki stays healthy and useful over time.

Lint is read-only. It reports findings. It does not modify content. The user decides what to act on, and uses `project-update` to make changes.

## What lint checks

### 1. Implicit items in prose

Scan document content for items that would benefit from inline tags but aren't tagged. These are the highest-value findings — they surface hidden project state.

Look for:
- **Assumptions stated as facts** — "The gateway supports token refresh" with no evidence or validation
- **Questions buried in paragraphs** — "We'll need to figure out whether the rate limits apply here"
- **Risks mentioned in passing** — "This could be a problem if the vendor changes their API"
- **Decisions implied but not recorded** — "We went with approach B" with no tag or provenance
- **Blockers mentioned casually** — "We can't proceed until the infra team confirms"
- **Dependencies not tracked** — "This depends on workstream 3 finishing first"
- **Actions implied but not assigned** — "Someone should check with the vendor"

Report each with its location and a suggested tag. Use whatever tag name best describes the nature of the item — the tag vocabulary is open, not prescribed.

Example output:
```
**Implicit items found:**
- "The gateway supports token refresh" (Architecture doc, section 2)
  → suggest: [ASSUMPTION] — no evidence or validation cited
- "We'll need to figure out whether rate limits apply" (Design canvas, para 4)
  → suggest: [OPEN QUESTION]
- "This depends on the payments team shipping their API first" (Workstream 2, para 1)
  → suggest: [DEPENDENCY]
```

### 2. Stale tagged items

Find tagged items that appear unresolved and may be stale:
- `[OPEN QUESTION]` with no corresponding `[OPEN QUESTION → RESOLVED]` — especially if the document hasn't been updated recently
- `[ASSUMPTION]` not yet confirmed or rejected
- `[ACTION]` not yet marked done
- Any tagged item where the surrounding content suggests the situation has changed but the tag wasn't updated

Report with age where possible (based on provenance dates or page modification timestamps).

### 3. Contradictions between documents

Compare claims, decisions, and assumptions across the document tree:
- Two documents stating different approaches to the same problem
- A decision in one doc that contradicts an assumption in another
- A resolved question whose answer conflicts with content elsewhere in the tree

This is the most valuable and hardest check. Be specific about what contradicts what and where.

### 4. Existing structured sections

Some documents already have their own organisational structures — RAID logs, decision registers, assumption tables, checklists. Lint should:
- **Recognise these** — don't suggest re-tagging items that are already tracked in a structured section
- **Check them for staleness** — a RAID log with risks last updated 3 weeks ago deserves a flag
- **Note coverage gaps** — a RAID log that tracks risks and issues but not assumptions, while assumptions appear untracked in prose elsewhere

### 5. Structural suggestions (document splitting)

Flag documents that may benefit from splitting into child nodes. The heuristics, viewed from the perspective of AI session token efficiency:

- **Document serves multiple audiences or tasks.** A doc with a solution architecture section and an implementation plan section means a PM reading architecture loads the implementation plan too, and a developer loading implementation context gets architecture they may not need.
- **A section could be the target of an orient call on its own.** "Orient me to workstream 2" should load workstream 2, not a parent doc containing workstreams 1-4.
- **Document exceeds ~3-4K tokens (~2000-3000 words) and contains multiple distinct concerns.** At that size, a session loading this doc plus its parent and a sibling is already at 10K+ tokens before doing any work.
- **Sections accumulate independently.** Different people or sessions contribute to different sections — creates concurrency pressure and makes provenance harder to track.

Report with reasoning:
```
**Structural suggestion:**
This doc is ~4500 tokens with 3 distinct sections (architecture, API design, deployment).
A session working on API design would load ~3000 tokens of unrelated content.
Consider splitting into child docs: `Design: API`, `Design: Deployment`
```

### 6. Cross-reference gaps

- Documents discussing the same topic without referencing each other
- Parent documents that don't reference all their children
- Orphan documents with no inbound references from the rest of the tree

## How to run lint

### Project-level lint

1. Use the Confluence MCP to fetch the project's page tree (titles, hierarchy)
2. Read each page's content
3. Run all checks above across the full tree
4. Present findings grouped by category, with the most actionable items first

### Document-level lint

If the user asks to lint a specific document:
1. Read the target document
2. Read its parent and immediate siblings for contradiction checking
3. Run checks 1-5 on the target, check 3 against parent/siblings
4. Present findings for that document only

## Output format

```
## Project Lint: [Project Name]
**Scanned:** [N] documents, [date]

### Implicit Items ([count])
- [item] — in [page title], [section]
  → suggest: [TAG]
...

### Stale Items ([count])
- [TAG] "[item text]" — in [page title], last updated [date/age]
...

### Contradictions ([count])
- [Doc A] says [X] but [Doc B] says [Y]
  Location: [specific sections]
...

### Structural Suggestions ([count])
- [Page title]: [reason to split], suggest [child doc names]
...

### Cross-Reference Gaps ([count])
- [Page A] and [Page B] both discuss [topic] but don't reference each other
...

### Existing Structures Noted
- [Page title] has a [RAID log / decision register / etc.] — [status: current / stale / gaps noted]
...
```

## What lint does NOT do

- **Does not modify any content.** Lint is read-only. Use `project-update` to act on findings.
- **Does not prescribe tag vocabulary.** Tags are open — whatever best describes the nature of the item. Lint suggests tags, it doesn't enforce a fixed set.
- **Does not re-tag structured sections.** If a RAID log already tracks risks, lint reports on its health but doesn't suggest inline tags on those items.
- **Does not mandate splits.** Structural suggestions include reasoning. The user decides.
