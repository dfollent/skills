---
name: research-codebase
description: Research and document the codebase as-is. Takes research questions (not a ticket) and produces an objective, self-contained research document. Pure codebase archaeology - no opinions, no implementation suggestions.
attribution: Inspired by HumanLayer's research_codebase (https://github.com/humanlayer/humanlayer/blob/main/.claude/commands/research_codebase.md) — Apache 2.0
triggers:
  - /research-codebase
model: opus
---

# Research Codebase

Triggered by: `/research-codebase [questions-file-or-inline-questions]`
Example: `/research-codebase 2026-04-14-SP-42-research-questions.md`

Conduct objective codebase research driven by a set of questions. Produce a self-contained research document usable in a fresh context window.

**This skill must NOT see the ticket or task description.** It receives only research questions. This prevents implementation bias from coloring findings.

## Inputs

Research questions - either a file path (output from `/generate-questions`) or inline questions from the user. If no questions are provided, ask the user for them.

If the user tries to provide a ticket or task description instead of questions, redirect them to `/generate-questions` first.

## Process

### Step 1: Read questions and any referenced files

Read the questions file fully. If specific files are mentioned in the questions, read those files in the main context before spawning sub-agents.

### Step 2: Decompose into vertical research tasks

Break questions into parallel research tasks. Bias toward vertical slices - trace flows end-to-end rather than cataloging horizontal layers.

Spawn parallel sub-agents:
- **Explorer agents** to find where components live
- **Tracer agents** to follow a flow from entry point to data store
- **Pattern agents** to find how the codebase handles similar concerns elsewhere

Each sub-agent prompt must:
- Include only its specific questions
- State: "Document what exists. No opinions, no suggestions, no improvements."
- Request file:line references for all findings
- Request actual code snippets for patterns found

### Step 3: Synthesize findings

Wait for ALL sub-agents to complete. Then compile results into a self-contained research document.

### Step 4: Write output

Write to the working directory: `YYYY-MM-DD-TOPIC-research.md`

```markdown
# Research: [Topic Area]

> Self-contained research document. Usable without prior conversation context.

## Questions Investigated
[List of questions this research addressed]

## Summary
[3-5 bullet points answering the core questions]

## Detailed Findings

### [Finding Area 1]
[What exists, how it works, where it lives]
- `path/to/file.ext:L42` - [what this code does]

### [Finding Area 2]
...

## Patterns Found

Reusable patterns discovered in the codebase, with code snippets:

### [Pattern Name]
**Where**: `path/to/file.ext:L42-L60`
**Used by**: [list of consumers]
```[language]
// actual code snippet
```

### [Pattern Name]
...

## Data Flow
[How data moves through the relevant systems, traced end-to-end]

## Open Questions
[Things that could not be determined from code alone]
```

## Rules

- **Document what IS, not what SHOULD BE.** You are a cartographer, not an architect.
- **No implementation opinions.** If you catch yourself writing "should", "could be improved", or "consider", delete it.
- **Patterns must include real code.** Every pattern needs a snippet with file:line. Never invent examples.
- **Self-contained output.** The research doc must be usable in a fresh context window with zero prior conversation.
- **Vertical over horizontal.** Trace flows end-to-end. "How does a request become a response" beats "list all middleware."
- **Stay under 40 instructions total.**
