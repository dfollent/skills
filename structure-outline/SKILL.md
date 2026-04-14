---
name: structure-outline
description: Takes a design discussion and produces a high-level structure outline (~100-200 lines) showing phases, testing strategy, and order of changes. The human review point before detailed planning.
triggers:
  - /structure-outline
model: opus
---

# Structure Outline

Triggered by: `/structure-outline [design-discussion-file]`
Example: `/structure-outline 2026-04-14-SP-42-design-discussion.md`

Produce a high-level structure outline that shows how to get from current state to desired end state. This is the last human review point before detailed planning. If the structure is wrong, everything downstream is wasted work.

Analogy: if the plan is the C implementation, the structure outline is the header file - just signatures and types.

## Inputs

A design discussion document (output from `/design`). If not provided, ask the user for one.

Read the design discussion fully before proceeding.

## Process

### Step 1: Identify phases

Break the implementation into vertical slices. Each phase should be independently testable. Order phases so earlier ones don't need rework when later ones land.

### Step 2: Write the structure outline

Write to the working directory: `YYYY-MM-DD-TICKET-structure-outline.md`

```markdown
# Structure Outline: [Feature Name]

> Based on design discussion: [filename]

## Approach
[2-3 sentences on the overall strategy]

## Phases

### Phase 1: [Name]
**Goal**: [What this phase accomplishes - one sentence]
**Touches**: [List of files/components affected]
**Testable by**: [How to verify this phase works independently]

### Phase 2: [Name]
**Goal**: ...
**Touches**: ...
**Testable by**: ...
**Depends on**: Phase 1

...

## Testing Strategy
- [How tests are structured across phases]
- [What gets unit tested vs integration tested vs manually verified]

## Risks
- [Risk and mitigation, only if not already covered in design discussion]

## Open Questions
- [Only questions that emerged from structuring, not rehashed from design]
```

### Step 3: Present for review

Show the outline and ask the user to confirm:
1. Phase ordering makes sense
2. Nothing is missing
3. Dependencies between phases are correct
4. Testing strategy is adequate

Wait for the user's response. If they provide corrections or push back on any point:
1. Update the file to reflect their feedback
2. Re-present only the changed sections
3. Repeat until the user explicitly approves

Do NOT proceed to `/create-plan` until approved.

## Rules

- **Keep it under 200 lines.** This is a structure doc, not a plan. No code, no implementation details.
- **Vertical slices only.** Each phase delivers testable end-to-end value. Never "Phase 1: update all models, Phase 2: update all controllers."
- **No code snippets.** File paths yes, code no. That's the plan's job.
- **Don't rehash the design discussion.** Reference it, don't repeat it.
- **Stay under 30 instructions total.**
