---

## description: Start the build phase of a feature implementation
alwaysApply: false

# Build Phase

You are starting the **build** (implementation) phase. An approved plan should already exist. If the user hasn't provided one, ask for it before proceeding.

## Your task

Implement the plan step by step, working in **vertical slices**. Each slice should produce a testable, committable increment.

## Rules

### Execution

- Follow the approved plan. If you see a better approach mid-implementation, flag it to the user before deviating.
- Work one slice at a time. After completing each slice, stop and let the user verify/test before moving on.
- Suggest a commit after each meaningful slice is verified. Commits are save points — if things go wrong, the user can revert to the last commit.

### Context management

- If the plan has sub-tasks meant for separate sessions, implement only the current sub-task. Don't try to do everything in one pass.
- Reference the plan to track progress. Mark completed steps and call out the current step.

### Feedback loop

- After each slice, ask the user to run tests and share results.
- If the user reports an error, ask for the exact error output — don't guess at the cause.
- Use test failures as the primary feedback signal: they contain the exact error, expected vs actual, and the stack trace.

### Quality

- Don't add code the plan didn't call for. No gold-plating.
- Keep changes minimal and focused on the current step.
- If a step reveals that the plan needs adjustment, pause and discuss before continuing.

## Progress tracking

After each completed step, update progress in this format:

```markdown
- [x] Step 1: [description] — DONE
- [x] Step 2: [description] — DONE
- [ ] Step 3: [description] ← CURRENT
- [ ] Step 4: [description]
```
