---
name: design
description: Design Discussion. Takes research + ticket, produces a ~200-line design doc with patterns, decisions, and open questions for human review. The critical alignment step before planning.
triggers:
  - /design
---

# Design Discussion

Triggered by: `/design [ticket-or-description]`
Example: `/design SP-42` or `/design Add rate limiting to public API`

Produce a concise design discussion document (~200 lines) that lets the human align the agent before any planning or code happens.

## Inputs

This skill needs two things:

1. **Research document** - output from a prior `/research_codebase` session
2. **Ticket/task description** - what we're building

If a research doc path is provided, read it. If not, ask the user for one or offer to run `/research_codebase` first.

If a Jira ticket ID is provided (e.g. SP-42), fetch it with `mcp__claude_ai_Atlassian__getJiraIssue`.

## Process

### Step 1: Gather inputs

1. Read the research document fully (no limit/offset).
2. Read the ticket or task description.
3. If either is missing, ask the user before proceeding.

### Step 2: Identify patterns to follow

Using the research document's file references, read the specific code sections that represent patterns relevant to this task. Find:

- How similar features are implemented in this codebase
- Naming conventions, file structure, test patterns
- Integration points the new work must connect to

Extract 3-5 concrete code snippets (with file:line references) that the implementation should follow.

### Step 3: Produce the design discussion

Write a markdown file to the working directory.
Filename: `YYYY-MM-DD-TICKET-design-discussion.md` (or `YYYY-MM-DD-description-design-discussion.md` if no ticket).

Use this structure:

```markdown
# Design Discussion: [Feature Name]

## Current State
[What exists today. 3-5 bullet points with file:line references.]

## Desired End State
[What should exist after implementation. Be specific about observable behavior.]

## Patterns to Follow

These code snippets from the codebase should guide implementation:

### Pattern 1: [Name]
**File**: `path/to/file.ext:L42-L60`
**Why this pattern**: [Brief reason]
```[language]
// relevant code snippet
```

### Pattern 2: [Name]
...

## Design Decisions

| Decision | Choice | Reasoning |
|---|---|---|
| [Decision 1] | [Choice made] | [Why] |
| [Decision 2] | [Choice made] | [Why] |

## Open Questions

Questions I need your input on before proceeding:

1. [Specific question about approach or business logic]
2. [Trade-off that needs human judgment]
3. [Ambiguity in requirements]

## Out of Scope

- [Thing we are explicitly not doing]
- [Adjacent work to defer]

## Risks

- [Risk 1 and mitigation]
- [Risk 2 and mitigation]
```

### Step 4: Present for review

After writing the file, present a summary and explicitly ask the user to:

1. Confirm or correct the patterns to follow
2. Answer the open questions
3. Challenge any design decisions
4. Flag missing risks or scope items

Do NOT proceed to planning until the user has reviewed and approved.

### Step 5: Iterate

If the user provides corrections or answers:

1. Update the design discussion file
2. If any correction invalidates a pattern, research the replacement pattern
3. Re-present the changed sections
4. Repeat until the user approves

## Rules

- **Keep it under 200 lines.** This is a review artifact, not a spec. Brevity is the point.
- **No implementation details.** No code to write, no file changes to make. That's the plan's job.
- **Patterns must be real code.** Every snippet must come from the actual codebase with a file:line reference. Never invent example code.
- **Open questions must be genuine.** Only ask things you cannot determine from the research or code. Never pad with obvious questions.
- **No opinions in disguise.** If you have a recommendation, state it as a design decision with reasoning. Don't hide it as a question.
- **Stay under 40 instructions total in your system behavior.** Stay within the instruction budget.
