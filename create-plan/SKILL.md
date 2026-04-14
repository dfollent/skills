---
name: create-plan
description: Takes a structure outline and design discussion, produces a detailed tactical implementation plan. This is a doc for the agent to execute, not a doc for humans to review at length.
attribution: Inspired by HumanLayer's create_plan (https://github.com/humanlayer/humanlayer/blob/main/.claude/commands/create_plan.md) — Apache 2.0
triggers:
  - /create-plan
model: opus
---

# Create Plan

Triggered by: `/create-plan [structure-outline-file]`
Example: `/create-plan 2026-04-14-SP-42-structure-outline.md`

Produce a detailed, actionable implementation plan that an agent can execute phase by phase. The human already reviewed the design discussion and structure outline. This plan translates those into tactical instructions.

## Inputs

1. **Structure outline** (output from `/structure-outline`) - required
2. **Design discussion** (output from `/design`) - referenced in the structure outline

Read both fully. If either is missing, ask the user.

## Process

### Step 1: Gather implementation details

For each phase in the structure outline, spawn parallel sub-agents to find:
- Exact files that need changes (with line numbers)
- Code patterns to follow (from the design discussion's "Patterns to Follow" section)
- Existing test patterns in the relevant areas
- Import conventions, naming conventions, config patterns

### Step 2: Write the plan

Write to the working directory: `YYYY-MM-DD-TICKET-plan.md`

```markdown
# Implementation Plan: [Feature Name]

> Structure outline: [filename]
> Design discussion: [filename]

## Phase 1: [Name from structure outline]

### 1.1 [Change group]
**File**: `path/to/file.ext`
**Action**: [Create | Modify | Delete]
**Details**:
- [Specific change with enough detail to execute]
- [Follow pattern from `path/to/example.ext:L42-L60`]

### 1.2 [Change group]
...

### Phase 1 Verification
#### Automated:
- [ ] `[test command]` passes
- [ ] `[typecheck command]` passes

#### Manual:
- [ ] [Specific thing to verify]

**Pause for manual verification before proceeding.**

---

## Phase 2: [Name]
...
```

### Step 3: Present the plan

Tell the user the plan file location. Suggest they skim it but remind them the real review happened at design + structure level.

## Rules

- **This plan is for agent execution.** Write it as instructions, not as a proposal.
- **No open questions.** Every decision was made in the design discussion. If you find an unresolved question, stop and ask before continuing.
- **Reference patterns, don't reinvent.** Point to the design discussion's code snippets. Don't discover new patterns here.
- **Pause between phases.** Every phase ends with verification and an explicit pause for manual checks.
- **No research.** The research is done. If you're missing information, the upstream artifacts are incomplete - flag it rather than spawning explorers.
- **Stay under 40 instructions total.**
