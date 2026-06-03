---

## description: Start the planning phase of a feature implementation
alwaysApply: false

# Planning Phase

You are starting the **planning** phase. A pre-planning brief should already exist. If the user hasn't provided one, ask for it before proceeding.

## Your task

Read the relevant code referenced in the brief, then produce a **structured implementation plan**.

## Plan format

Use this exact structure:

```markdown
## Implementation Plan: [Feature Name]

### Overview
[1-2 sentences summarizing the approach]

### Files to create/modify
- `path/to/file.ts` — [what changes and why]
- ...

### Steps
1. [Step description] — files: `path/to/file.ts`
2. [Step description] — files: `path/to/file.ts`
   - Depends on: step 1
...

### Risk areas
- [Potential issue and how to mitigate it]
```

## Rules

- **Read the code first.** Use `@` references from the brief to read the actual current state before planning. Do not guess at the code structure.
- **Structured, not free-form.** List files to change, step order, dependencies between steps, and risk areas.
- **Size-gate the plan.** If the plan exceeds ~8 steps, suggest splitting into independent sub-tasks. Each sub-task should be completable in one session, testable in isolation, and committable as a meaningful unit (one PR per sub-task).
- **Vertical slices, not horizontal layers.** Organize steps as end-to-end slices (type + logic + handler + test), not layer by layer (all types, then all logic, then all tests).
- **Do NOT write code yet** — that's the next phase.
- Wait for the user to review and approve the plan before moving on. Encourage them to challenge it: "What could go wrong with step N?" or "Is there a simpler approach?"

