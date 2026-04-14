---
name: generate-questions
description: Takes a ticket or task description and generates focused research questions about the codebase. First half of the untainted research pattern - this skill sees the intent, the research skill does not.
triggers:
  - /generate-questions
---

# Generate Research Questions

Triggered by: `/generate-questions [ticket-or-description]`
Example: `/generate-questions SP-42` or `/generate-questions Add rate limiting to public API`

Generate research questions that will drive objective codebase research. This skill sees the ticket. The subsequent `/research-codebase` skill will only see these questions - never the ticket itself. This separation prevents implementation opinions from tainting research.

## Inputs

A ticket ID or task description. If a Jira ticket ID is provided (e.g. SP-42), fetch it with `mcp__claude_ai_Atlassian__getJiraIssue`.

## Process

### Step 1: Understand the task

Read the ticket/description. Identify:
- What needs to change
- Which systems are likely involved
- What you don't know about the codebase yet

### Step 2: Generate questions

Produce 5-15 research questions organized by theme. Each question should be:
- **Answerable by reading code** - not product/business questions
- **Intent-neutral** - someone reading the questions alone should not be able to infer what feature is being built
- **Specific** - "How does the auth middleware chain work?" not "How does auth work?"

Categories to consider:
- Existing patterns and conventions for similar features
- Data models and schemas involved
- Integration points and interfaces
- Test patterns used in related areas
- Error handling and edge case conventions
- Configuration and environment concerns

### Step 3: Output

Write a markdown file: `YYYY-MM-DD-TICKET-research-questions.md`

```markdown
# Research Questions: [Topic Area]

## [Theme 1]
1. [Question]
2. [Question]

## [Theme 2]
3. [Question]
4. [Question]

...
```

Present the questions to the user for review. They may add, remove, or refine questions before running `/research-codebase`.

## Rules

- **No implementation opinions.** Questions only, no suggestions about how to build anything.
- **Intent-neutral framing.** Strip the "why" from questions. Ask "how does X work?" not "how should we modify X to do Y?"
- **Bias toward vertical slices.** Ask questions that trace end-to-end flows, not horizontal layers.
- **Stay under 30 instructions total.**
