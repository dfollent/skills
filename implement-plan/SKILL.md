---
name: implement-plan
description: Execute an implementation plan phase by phase, with verification between phases and pauses for manual testing. Consumes the plan from /create-plan.
attribution: Derived from HumanLayer's implement_plan (https://github.com/humanlayer/humanlayer/blob/main/.claude/commands/implement_plan.md) — Apache 2.0
triggers:
  - /implement-plan
model: opus
---

# Implement Plan

Triggered by: `/implement-plan [plan-file]`
Example: `/implement-plan 2026-04-14-SP-42-plan.md`

Execute an approved implementation plan phase by phase. The plan was produced by `/create-plan` and already incorporates decisions from the design discussion and structure outline. Your job is to follow the plan's intent, verify as you go, and pause for human checks between phases.

## Inputs

A plan file (output from `/create-plan`). Read it fully. Also read the design discussion referenced in the plan header - it contains the patterns to follow.

If no plan file is provided, ask for one.

## Process

### Step 1: Orient

- Read the plan fully
- Read the design discussion referenced in the plan (for patterns and decisions)
- Check for existing checkmarks (`- [x]`) - if resuming, pick up from the first unchecked item
- Read all files mentioned in the first phase before writing any code

### Step 2: Implement phase by phase

For each phase:

1. Read all files involved in the phase before making changes
2. Write failing tests first (TDD), then implement to make them pass
3. Follow the patterns from the design discussion - don't invent new patterns
4. Update checkboxes in the plan file as you complete items

### Step 3: Verify each phase

After completing a phase, run the automated verification listed in the plan (tests, types, lint). Fix issues before proceeding.

Then pause:

```
Phase [N] complete - ready for manual verification.

Automated checks passed:
- [list what passed]

Manual verification needed:
- [list manual checks from the plan]

Let me know when manual testing is done so I can proceed to Phase [N+1].
```

Do not proceed until the user confirms. Do not check off manual verification items yourself.

If instructed to run multiple phases consecutively, skip the pause until the last phase.

### Step 4: Handle mismatches

If the code doesn't match what the plan expects, stop:

```
Mismatch in Phase [N]:
Expected: [what the plan says]
Found: [actual situation]
Why this matters: [impact on the plan]

Options:
1. [Adapt the current phase to work with what exists]
2. [Flag that the upstream design/structure may need revisiting]
```

Do not silently work around mismatches. The plan was built on specific assumptions from the design discussion - if those assumptions are wrong, the human needs to know.

## Rules

- **TDD: write failing tests first.** Then implement to make them pass.
- **Follow the plan's intent, not just its letter.** If the codebase has shifted since the plan was written, adapt intelligently but communicate what changed.
- **No research or exploration.** That work is done. If you're missing information, the upstream artifacts are incomplete - flag it.
- **No new patterns.** Use the patterns from the design discussion. If they don't fit, that's a mismatch to surface, not a judgment call to make alone.
- **One phase at a time.** Complete and verify before moving on.
- **Stay under 40 instructions total.**
