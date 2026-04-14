---
name: project-onboard
description: "Orient to a project on the team's Confluence surface, or load context for a specific task. Use this skill whenever someone wants to: see what projects exist, understand the state of a project, find what needs attention, identify what to work on next, get a summary of open questions/blockers/assumptions, or load context for a specific task before starting work. Triggers include: 'what projects do we have', 'orient me to X', 'what's happening with project Y', 'what needs attention', 'I have 2 hours, what should I pick up', 'show me the project tree', 'what's blocked', 'load context for workstream 2', 'I want to work on the API task', 'pick up the auth refresh task'. Use this even when the user just names a project or task and seems to want context — they're orienting."
---

# Project Orient

Orient a user (or an AI session) to a project on the team's shared Confluence surface, or load focused context for a specific task. The goal: anyone can self-serve into a project without a meeting, a handover, or asking the author of a document to explain it.

## What this skill does

Three modes, escalating in depth:

### Team-level: "What projects do we have?"

1. Use the Confluence MCP to list pages in the relevant team space
2. Identify top-level pages that represent projects (project = top-level parent page in a team space)
3. Present a concise list: project name, brief description (from the page's opening paragraph), and child page count

### Project-level: "Orient me to project X"

1. Read the project's root page to understand the project brief / goal
2. Fetch the page tree (all child pages and their children) — titles and hierarchy, not full content yet
3. Scan for inline tags using CQL search — do NOT read each page individually. Run one search per tag type against the project root as ancestor:
   ```
   text ~ "[OPEN QUESTION]" AND ancestor = <root-page-id>
   text ~ "[ASSUMPTION]" AND ancestor = <root-page-id>
   text ~ "[BLOCKER]" AND ancestor = <root-page-id>
   text ~ "[DECISION]" AND ancestor = <root-page-id>
   text ~ "[ACTION]" AND ancestor = <root-page-id>
   ```
   This replaces N page reads with a handful of targeted queries.
4. For resolved tags, search for the arrow form: `text ~ "[ASSUMPTION →"`, etc. Only fetch the full page content of a result if the excerpt doesn't include enough context.
5. Present the orientation as:

```
## Project: [Name]
**Goal:** [One-line from the root page]

### Document Tree
- [Root page title]
  - [Child page title] (Research:, Plan:, etc.)
    - [Grandchild page title]
  ...

### Outstanding Items
**Open Questions ([count]):**
- [question text] — in [page title]
- ...

**Blockers ([count]):**
- [blocker text] — in [page title]
- ...

**Unvalidated Assumptions ([count]):**
- [assumption text] — in [page title]
- ...

**Actions ([count]):**
- [action text] — in [page title]
- ...

### Recent Decisions
- [decision text] — in [page title]
```

### Task-level: "I want to work on workstream 2" / "Load context for the API task"

This mode loads focused context for a specific task so the user can start working with their own tools and skills (TDD, plan-and-implement, review, etc.). The orient skill sets the stage, then gets out of the way.

1. **Find the target.** Use the project tree or search to locate the specific doc/task the user is referring to.
2. **Read the target doc** in full — this is the primary context.
3. **Load a fixed, minimal set — do not walk the tree speculatively:**
   - **Always read:** The project root page (overall direction and goals)
   - **Always read:** The direct parent of the target (scope and lineage)
   - **Always read:** The target page itself
   - **Skip by default:** Referenced docs, siblings, other workstreams. Only load these if the user explicitly asks or the target doc makes it clear work cannot proceed without them.
4. **Surface relevant tags** from the loaded docs — especially unresolved `[OPEN QUESTION]`, `[ASSUMPTION]`, and `[BLOCKER]` items that might affect the work.
5. **Present a brief summary** of what you loaded and what the user should be aware of before starting work:

```
## Context for: [Task name]
**From:** [Page title] in [Project name]

**Goal:** [What this task is about, from the doc]

**Context loaded:**
- [Project root page] — overall direction
- [Parent page] — parent workstream scope
- [Target page] — the task itself
- [Referenced page] — dependency mentioned in target

**Watch out for:**
- [ASSUMPTION] X hasn't been validated yet
- [OPEN QUESTION] Y is still unresolved and may affect this work
- [BLOCKER] Z is flagged but may be cleared — check before proceeding

**You're ready to work.** Use your preferred approach — the project-update skill is available when you want to write findings back.
```

After presenting the context summary, the skill is done. The user proceeds with their own workflow (coding, research, review, etc.) and uses `project-update` when they're ready to write back.

## How to use the Confluence MCP

You have access to the Atlassian Confluence MCP. Use its tools to:

- **List pages in a space** to find projects (team-level orient)
- **Read page content** to extract the project brief and scan for inline tags
- **Navigate page hierarchy** (parent/child relationships) to build the tree
- **Search within a space** if the user asks about a specific topic

Read pages progressively. Use `getConfluencePageDescendants` to get the tree structure (titles and IDs only) before deciding what to read in full. Use CQL search for tag scanning — never read every page to find tags. Only fetch full page content when you need the actual prose (root page brief, target task doc).

## Handling legacy docs

Many existing Confluence pages won't have inline tags. That's fine. When orienting:

- Report what you find. If a doc has no tags, just show it in the tree without tag counts.
- Do NOT proactively add tags to existing docs during orientation. That's the contribute skill's job.
- If you notice implicit assumptions, questions, or blockers while reading (even without tags), you can mention them in your orientation summary with a note like "*(implicit — not tagged in doc)*". This helps the user see what's there without modifying the content.
