---

## description: Start the pre-planning phase of a feature implementation
alwaysApply: false

# Pre-Planning Phase

You are starting the **pre-planning** phase for a new feature. Your goal is to help the user build a concise implementation brief before any code is written or planned.

## Your task

Guide the user through defining the following elements. Ask questions to fill gaps — don't assume anything that isn't stated:

1. **Goal** — What is the feature or change? One sentence.
2. **Why** — What problem does this solve? What motivated this work?
3. **Constraints** — Technical boundaries: existing dependencies to use, performance requirements, backward compatibility needs, infrastructure limits.
4. **Don't touch** — Files, modules, or APIs that must NOT be changed.
5. **Relevant code** — Ask the user to point you to the key files/directories using `@` references so you can read the current state.

## Rules

- Do NOT write code or propose an implementation yet.
- Do NOT produce a step-by-step plan yet — that's the next phase.
- Keep the output brief: 10-20 lines max.
- State constraints explicitly, not just goals. "Add caching" is a goal. "Add caching using the existing Redis instance, no new dependencies, TTL configurable per-route" is a constraint set.
- Ask for the "why" if the user didn't provide it — it drives better trade-off decisions later.

## Output format

Once you've gathered enough information, produce a brief in this format:

```
Goal: [one sentence]
Why: [motivation / problem]
Constraints:
- [constraint 1]
- [constraint 2]
- ...
Don't touch: [files/modules/contracts to preserve]
Relevant code: [@ references]
```

Ask the user to confirm or adjust the brief before moving on.
