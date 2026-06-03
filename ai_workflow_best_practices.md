# AI-Assisted Development: Strategies and Best Practices

## Current Workflow: What's Strong

The **Pre-planning → Planning → Implementation** pipeline is solid because it addresses the two biggest failure modes of AI-assisted development:

1. **Ambiguity kills quality.** Vague prompts produce vague code. The pre-planning step directly mitigates this.
2. **Large changes without a plan drift.** Context windows are finite. A reviewed plan acts as an anchor.

But there are refinements that can make each step significantly more effective.

---

## Refined Strategy by Phase

### Phase 1: Pre-Planning (Context Loading)

**What you're doing right:** Giving context upfront, removing ambiguity.

**What to optimize:**

- **Reference files explicitly with `@`.** Don't describe code structure — point me to it. `@src/modules/auth/` is worth 100 words of explanation. I can read code faster than I can parse your description of it.
- **State constraints, not just goals.** "Add caching" is a goal. "Add caching, must work with our existing Redis instance, no new dependencies, TTL configurable per-route" is a constraint set. Constraints reduce my search space dramatically.
- **Tell me what NOT to change.** "Don't touch the existing API contract" or "keep backward compatibility with v2 clients" prevents me from generating technically correct but practically unusable solutions.
- **Share the "why".** If I know *why* you need a change, I make better trade-off decisions autonomously instead of stopping to ask.

**Output of this phase:** A concise brief — not a spec. Think 10-20 lines max. Something like:

```
Goal: Add rate limiting to the public API
Why: We're seeing abuse on /api/search, 50k req/min from single IPs
Constraints:
- Use existing Redis (already in docker-compose)
- Per-route configurable limits
- Must not break existing auth middleware chain
- Return 429 with Retry-After header
Don't touch: Internal API routes (/api/internal/*)
Relevant code: @src/middleware/ @src/routes/api/
```

---

### Phase 2: Planning (This is Where Most People Under-Invest)

**What you're doing right:** Letting me read code before planning, reviewing the plan.

**What to optimize:**

- **Ask for the plan in a structured format.** Free-form plans are hard to review. Ask for something like: files to create/modify, order of changes, dependencies between steps, and risk areas. This makes review faster and gives you checkpoints.
- **Challenge the plan.** Don't just approve it. Ask "what could go wrong with step 3?" or "is there a simpler approach to X?". I'll often reveal a better path when pushed.
- **Size-gate the plan.** If a plan has more than ~8-10 steps, it's too big for a single session. Split it. This is the single most impactful thing you can do for large implementations.

**For small changes (< 5 files):** You can often collapse Phase 1 and 2 into one message. Give me the brief + ask for implementation directly. Over-planning small changes wastes time.

**For large changes (> 10 files):** Split the plan into independent sub-tasks. Each sub-task should be:
- Completable in one session
- Testable in isolation
- Committable as a meaningful unit

This maps naturally to **one PR per sub-task**, which is also better for code review.

---

### Phase 3: Implementation (Where Context Window Management Matters)

**What you're doing right:** Iterating with testing and feedback.

**What to optimize:**

- **Work in vertical slices, not horizontal layers.** Don't ask me to "create all the types first, then all the handlers, then all the tests." Instead: "implement the rate limiter middleware end-to-end, with tests." Vertical slices keep the context coherent and produce testable increments.
  - *Horizontal* = build all types → all queries → all handlers → all tests. By the last layer, early decisions are out of context and bugs are expensive to fix.
  - *Vertical* = build one feature path top-to-bottom (type + query + handler + test), then the next. Each slice is testable, committable, and keeps full context in scope.
  - Example: building a notification system → Slice 1: email notifications end-to-end, Slice 2: in-app notifications, Slice 3: preferences, Slice 4: read status. Each delivers working functionality.
- **Commit frequently and tell me.** After each meaningful chunk, commit. If the session gets long, I can lose track of what was already done vs. what's pending. A commit is a save point. If things go wrong, you can say "revert to last commit and try again."
- **Give precise feedback.** "This doesn't work" forces me to guess. "The middleware runs but `req.rateLimit` is undefined in the handler — I checked and the middleware is registered in the right order" gives me a surgical fix path.
- **Use test failures as feedback.** Run tests and paste failures. Test output is the most information-dense feedback you can give me. It has the exact error, the expected vs actual, and the stack trace.

---

## Size-Based Playbook

### Small (1-3 files, bug fix, small feature)

```
One message: context + what to do + relevant files
Skip formal planning
Implement → test → done
```

This should be a single turn or two. Don't over-process it.

### Medium (4-10 files, new feature, refactor)

```
Message 1: Brief with constraints + ask for plan
Message 2: Review plan, request adjustments
Message 3+: Implement in 2-3 vertical slices, test each
```

### Large (10+ files, new system, migration)

```
Session 0: Pre-planning brief, ask for decomposition into sub-tasks
Session 1-N: One sub-task per session, each following the medium pattern
Use git branches per sub-task
```

**Critical insight for large work:** Start a new chat for each sub-task. Long conversations degrade quality because the context window fills up. A fresh chat with a focused brief ("implement sub-task 3 from this plan: [paste plan]") outperforms a 50-message thread every time.

---

## Starting a New Session (Continuity Between Chats)

The AI has zero memory between sessions. Treat each new chat like handing off to a different developer. A strong session-opener includes:

1. **The plan** (or the relevant sub-task) — the roadmap
2. **What's already done** — so work isn't repeated ("steps 1-2 are done and committed")
3. **File references with `@`** — so the AI reads current state instead of guessing
4. **A clear starting point** — "start with step 3"

```
I'm implementing [feature X]. Here's the plan:
[paste plan with completed items marked]

Steps 1-2 are done. Start with step 3: [description].
Relevant code: @src/services/rate-limiter.ts @src/middleware/
```

**Don't:** say "continue from last chat", link previous conversations, or paste entire conversation histories.

---

## Tracking Progress Across Sessions

Use the **plan document as the single source of truth**. Update it as you go — mark completions, annotate decisions, add commit hashes. When starting a new session, paste the relevant section.

```markdown
## Implementation Plan: Rate Limiting

- [x] Step 1: Create rate limiter middleware — DONE (commit abc123)
- [x] Step 2: Add Redis adapter — DONE (commit def456)
  - Note: used ioredis instead of redis, better TS support
- [ ] Step 3: Per-route configuration  ← CURRENT
- [ ] Step 4: 429 response formatting
- [ ] Step 5: Tests
```

- **Git history** is a good complement (rollback points, diffs to paste) but poor as a primary tracker — it shows *what changed*, not *why* or *what's next*.
- **A separate progress file** only makes sense for multi-week efforts with 20+ sub-tasks. For most work, the annotated plan is simpler and avoids keeping two documents in sync.

---

## Anti-Patterns to Avoid

| Anti-pattern | Why it hurts | Instead |
|---|---|---|
| "Make it production-ready" | Too vague, I'll add things you don't need | Specify exactly what "ready" means |
| Pasting entire files as context | Wastes context window | Use `@file` references |
| Accepting the first plan without review | Plans often have a simpler alternative | Push back at least once |
| One massive implementation message | I lose coherence after ~500 lines of changes | Break into vertical slices |
| Not running tests between changes | Errors compound silently | Test after each slice |
| "Fix it" after a failure | I need the error message | Paste the exact error/output |
| Staying in one long chat forever | Context quality degrades | New chat per sub-task |

---

## Meta-Strategies

1. **Treat me as a senior dev who just joined the team.** I'm technically capable but don't know your codebase, your team's conventions, or your deployment constraints. The more of that you front-load, the better I perform.

2. **Use Cursor Rules for recurring context.** If you always want me to follow certain patterns (testing style, import conventions, error handling approach), put them in `.cursor/rules/`. That way you don't re-explain them every session.

3. **Verify, don't trust.** I'm good at generating plausible code. Plausible ≠ correct. Always run it. My confidence in my output has zero correlation with its correctness.

4. **Use me for the boring parts.** Tests, boilerplate, migrations, type definitions, refactoring — I'm fast and consistent at these. Use your own time for architecture decisions, code review, and the parts that require deep domain knowledge.
