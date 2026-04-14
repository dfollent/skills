# Claude Code Skills

A collection of [Claude Code skills](https://docs.anthropic.com/en/docs/claude-code/skills) for structured software delivery.

## Quick Reference

| Skill | Description | Install |
|-------|-------------|---------|
| `/generate-questions` | Ticket to intent-neutral research questions | `npx skills@latest add dfollent/skills/generate-questions` |
| `/research-codebase` | Objective codebase research from questions | `npx skills@latest add dfollent/skills/research-codebase` |
| `/design` | ~200-line design discussion for human review | `npx skills@latest add dfollent/skills/design` |
| `/structure-outline` | High-level phase outline for human review | `npx skills@latest add dfollent/skills/structure-outline` |
| `/create-plan` | Detailed tactical plan for agent execution | `npx skills@latest add dfollent/skills/create-plan` |
| `/implement-plan` | Phase-by-phase plan execution with verification | `npx skills@latest add dfollent/skills/implement-plan` |
| `/grill-me` | Stress-test a plan or design | `npx skills@latest add dfollent/skills/grill-me` |
| `/project-orient` | Load project context from Confluence | `npx skills@latest add dfollent/skills/project-orient` |
| `/project-update` | Write findings back to Confluence | `npx skills@latest add dfollent/skills/project-update` |
| `/project-lint` | Audit Confluence doc tree for quality | `npx skills@latest add dfollent/skills/project-lint` |

Install all skills at once:

```bash
npx skills@latest add dfollent/skills
```

## Coding Workflow

Skills that break delivery into focused phases. Each phase runs in its own context window, produces a static markdown artefact, and feeds into the next. Human review concentrates where it has the most leverage.

### The flow

**1. `/generate-questions`** - Takes a ticket, produces intent-neutral research questions. You review and refine the questions before research starts.

**2. `/research-codebase`** - Takes only the questions (never sees the ticket). Sends sub-agents through vertical slices of the codebase. Outputs an objective research doc with code snippets, patterns found, and data flows. No opinions.

**3. `/design`** - Takes research + ticket. Produces a ~200-line design discussion: current state, desired end state, patterns to follow (real code snippets), design decisions, and open questions for you to answer. This is your first review point (and a good place to use `/grill-me`).

**4. `/structure-outline`** - Takes the design discussion. Produces a ~100-200 line phase outline: what order to build things, what each phase delivers, how to test each phase independently. Second review point (also good for `/grill-me`).

**5. `/create-plan`** - Takes structure + design. Produces a detailed tactical plan the agent executes. You skim this, no need for full review. The real decisions were made at steps 3-4.

**6. `/implement-plan`** - Execute the plan, phase by phase, with verification between phases.

### Why it works

- **You review 200 lines of design, not 1000 lines of plan or code.** That's where your leverage is highest, and it's more sustainable.
- **Each skill stays under 40 instructions.** Smaller prompts = fewer skipped steps.
- **Research is unbiased.** The questions phase sees what you're building, the research phase doesn't. Prevents the model from cherry-picking facts that support what it already wants to build.
- **Every phase produces a standalone markdown doc.** You can resume from any point in a fresh session.

### Skipping steps

Not every task needs all six steps. Use your judgement:

- **Small bug fix?** Skip straight to implementation.
- **Well-understood change?** Start at `/design` or `/create-plan`.
- **Complex feature in unfamiliar code?** Run the full flow.

## Project Skills (Confluence)

Two skills for project work in Confluence. One reads from the project surface, the other writes back to it.

### `/project-orient`

Load context from Confluence without a meeting or handover.

- "What projects do we have?" - lists all projects in a space
- "Orient me to project X" - project tree, open questions, blockers, unvalidated assumptions, recent decisions
- "I want to work on the API task" - loads the target doc + parent + project root, surfaces anything that might affect the work

### `/project-update`

Write findings back so the project surface stays current.

- Appends to existing docs or creates child pages, with provenance (who wrote it, when, which tool)
- Inline tags for open questions, assumptions, blockers, decisions, actions
- Tags get resolved in-place: `[ASSUMPTION -> CONFIRMED]`, `[BLOCKER -> CLEARED]`
- Captures implementation write-backs: PR links, branch names, learnings, rejected assumptions

### Why this matters

- No one needs a handover meeting to pick up work. `/project-orient` self-serves into any project.
- No one writes status reports. `/project-update` captures state as a side effect of doing the work.
- AI sessions can scan the tags programmatically. The project surface is machine-readable and human-readable.
- The next person (or AI session) picking up the project gets full context without asking anyone.

### `/project-lint`

Audits a project's Confluence document tree for quality, consistency, and structure. Finds stale items, contradictions, orphan pages, and missing tags.

## Utility Skills

### `/grill-me`

Stress-test a plan or design. Claude interviews you relentlessly about every aspect, walking down each branch of the decision tree and resolving dependencies one by one. Best used after `/design` or `/structure-outline`.

Attribution: [Matt Pocock](https://github.com/mattpocock/skills/blob/main/grill-me/SKILL.md) (MIT License).

## Acknowledgements

The coding workflow is informed by [Dex Horthy's](https://github.com/humanlayer) CRISPY methodology and the evolution from Research-Plan-Implement (RPI). Several skills are derived from or inspired by [HumanLayer's](https://github.com/humanlayer/humanlayer) Claude Code commands (Apache 2.0). See [NOTICE](NOTICE) for full attribution.
