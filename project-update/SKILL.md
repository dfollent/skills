---
name: project-update
description: "Write back findings, updates, observations, or new content to a project's Confluence surface. Use this skill whenever someone wants to: update a project doc, write back findings from a session, add questions or blockers to a project, record a decision, capture meeting notes, append research, create a new doc in the project tree, create a top-level project document (brief, design, architecture), or mark an assumption as confirmed/rejected. Triggers include: 'update the project', 'write back my findings', 'add this to the research doc', 'record this decision', 'capture these meeting notes', 'mark this as resolved', 'create a new doc under workstream 2', 'create a solution architecture doc', 'add a product brief to the project', 'save my findings to the project'. Use this skill any time content is being written to the project Confluence surface — even if the user doesn't explicitly say 'update'."
---

# Project Update

Write back to the team's shared Confluence project surface with consistent conventions for provenance, inline tags, and tree structure. Every write should make the project more self-service — the next person or AI session picking up this project should be able to understand what happened without asking anyone.

## Core principle

Every contribution is attributed, tagged, and placed correctly in the tree. The conventions exist so that the project surface is machine-parseable (AI sessions can scan for tags and provenance) and human-readable (team members see a clear, structured page).

## Provenance format

Every contribution — whether appended to an existing doc or created as a new child page — includes provenance. Use this format:

```
---

**[TAG]** — Author Name, DD Mon YYYY

Content of the contribution here. Can be multiple paragraphs.

[OPEN QUESTION] Any embedded questions tagged inline.
```

- The horizontal rule (`---`) separates this contribution from previous content
- The tag, author, and date are on the first line in bold
- Content follows as plain text
- For AI-generated contributions, the author format is: **[Harness] [Model] (via [User's Name])** — e.g. `Claude Code Opus 4.6 (via Dan Follent)`, `Copilot GPT-5.4 (via Dan Follent)`. This captures which tool and model produced the content, which matters as different harnesses and models have meaningfully different quality profiles.
- For human contributions made through Claude Code, the author is the human's name

### Inline tags

Tags are inline, in square brackets, uppercase. The vocabulary is open — use whatever tag best describes the nature of the item. Common examples:

| Tag | When to use | Example |
|-----|-------------|---------|
| `[OPEN QUESTION]` | Something that needs answering before work can proceed | "Can the website team support this in the current sprint?" |
| `[ASSUMPTION]` | A belief that hasn't been validated | "The existing auth gateway supports token refresh" |
| `[BLOCKER]` | Something that stops progress | "Cannot proceed until API rate limits are confirmed with vendor" |
| `[DECISION]` | An outcome from a discussion or review | "Team agreed to use approach B for payment integration" |
| `[RISK]` | Something that could go wrong | "Vendor API may change before launch" |
| `[DEPENDENCY]` | Something this work depends on | "Requires payments team to ship their API first" |
| `[ACTION]` | A lightweight follow-up assigned to someone | "Dan to confirm website team capacity by Thursday 10 Apr" |
| `[REVIEW FINDING]` | Something discovered during review, refinement, or reading | "The auth token approach in Workstream 2 conflicts with the gateway config" |

These are examples, not a closed set. If `[TRADE-OFF]`, `[SPIKE NEEDED]`, or `[CONSTRAINT]` better describes what you're capturing, use that. The convention is the format (inline, square brackets, uppercase), not the vocabulary.

### Resolved tags

When an item is resolved, update the tag inline by appending an arrow and outcome rather than deleting it:

- `[ASSUMPTION → CONFIRMED]`, `[ASSUMPTION → REJECTED]`
- `[OPEN QUESTION → RESOLVED]`
- `[BLOCKER → CLEARED]`
- `[RISK → MITIGATED]`, `[RISK → ACCEPTED]`
- `[ACTION → DONE]`

Always include a note on who resolved it and when. The resolution format is also open — use whatever outcome word fits.

Example:
```
[ASSUMPTION → CONFIRMED] The existing auth gateway supports token refresh — confirmed by Sarah, 8 Apr 2026
```

## Two modes of writing

### 1. Append to an existing doc

Use when: adding observations, questions, status updates, meeting notes, review findings to an existing page.

The contribution goes at the bottom of the doc in the provenance format above. The original content is preserved. You can also make inline edits to existing content (see below).

### 2. Create a child page

Use when: the content could be useful standalone to a future session. This is the key heuristic — if someone might want to load this content independently (without the parent doc), it should be a child page.

This applies at every level of the tree — including directly under the project root. A solution architecture doc, a product brief, and a workstream plan are all child pages of their parent, using the same mechanism as a research doc nested three levels deep. The difference is placement (which parent), not the operation.

#### When to split vs. keep inline

Think from the perspective of a future AI session's context window — tokens are a zero-sum resource:

- **The content serves a different audience or task than the parent.** A PM reading the architecture shouldn't have to load the implementation plan. A developer loading implementation context shouldn't have to load architecture they don't need.
- **The content could be the target of an orient call on its own.** "Orient me to workstream 2" should load workstream 2, not a parent doc containing workstreams 1-4.
- **The parent document would exceed ~3-4K tokens (~2000-3000 words) with this content added.** At that size, a session loading this doc plus its parent and a sibling is already at 10K+ tokens before doing any work.
- **The content will accumulate independently.** If different people or sessions will contribute to this section over time, it should be its own node — otherwise concurrent edits create pressure and provenance gets muddled.

When in doubt, split. Smaller, focused documents are cheaper to load, easier to orient into, and simpler to maintain.

#### Common child page types

**Directly under the project root:**
- Product brief or requirements document
- Solution architecture or design canvas
- High-level workstream definitions

**Under a workstream or other parent:**
- Research findings that are substantial (more than a few paragraphs)
- Meeting notes from a specific session
- Implementation plans
- Spike or POC results
- Design documents for a specific component

**Naming convention for child pages:** Use a type prefix:
- `Brief: [topic]`
- `Design: [topic]`
- `Research: [topic]`
- `Meeting Notes: [date or topic]`
- `Spike: [topic]`
- `Plan: [topic]`
- `Implementation: [topic]`
- `Review: [topic]`

Create the child page under the most relevant parent in the tree. Top-level project documents go directly under the project root. Research about a specific workstream goes under that workstream's page.

## Inline edits with provenance

You can edit existing content in a page — not just append. When doing so, add a provenance note at the point of change so the edit is traceable:

```
**[EDIT]** — Dan Follent, 6 Apr 2026: Narrowed Workstream 2 scope to exclude payment integration per team decision
```

For tag state changes (assumption confirmed, question resolved), update the tag inline as shown in the "Resolved tags" section above.

## Retrofitting legacy content

When you're writing to a page that has existing content without inline tags:

- Look at the sections you're touching. If you see implicit assumptions, questions, or blockers in the existing text, tag them as part of your write.
- Use provenance on the retrofit: `**[RETROFIT]** — Claude (via Dan Follent), 6 Apr 2026: Tagged existing assumptions in this section`
- Only retrofit sections you're already editing or directly adjacent to your contribution. Don't retrofit the entire doc — that's a separate explicit request.

## What to capture in write-back from code work

When writing back after working in a codebase, capture what the next person or AI session would need:

- **Status**: What was done and what's the current state
- **Summary**: Brief description of the approach taken and why
- **Links**: PR links, branch names, relevant file paths — concrete references back to the code
- **Learnings**: Anything discovered during implementation that the project should know — constraints, surprises, changed assumptions, things that were harder or easier than expected
- **Open items**: Any remaining work, questions that emerged, or follow-ups needed

Example write-back after implementation work:
```
---

**[REVIEW FINDING]** — Claude (via Dan Follent), 6 Apr 2026

Implemented the auth token refresh endpoint in `api-gateway` repo.

**Status:** PR open, tests passing, pending review
**Branch:** `feature/token-refresh` | **PR:** [#142](link)
**Approach:** Used the existing middleware chain rather than a new endpoint. The gateway's rate limiter needed a bypass rule for refresh calls.

**Learnings:**
- The rate limiter config lives in `infra-config` repo, not in the gateway repo. Future auth changes need to touch both.
- [ASSUMPTION → REJECTED] Token refresh was assumed to be stateless — it actually requires a Redis lookup for session validation.

[OPEN QUESTION] Should we add monitoring for refresh token failure rates? Currently no alerting on this path.
```
