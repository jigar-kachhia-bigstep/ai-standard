# How to Prompt Agents

> Use these recipes to get focused, accurate output from Claude and Cursor.
> Bad input = bad output. A vague prompt forces the agent to guess — and it will guess wrong.

---

## The Golden Rule

**State the task + the constraint + the context. Never just the task.**

```
❌ "Add payment endpoint"
✅ "Add POST /v1/payments to payment-service per FEAT-007 plan.
    Constraint: must use Idempotency-Key header (see CLAUDE.md contracts).
    Context: this is the first write endpoint in this service."
```

---

## Prompting a PLANNER

```
Create plan for FEAT-{ID}: {feature name in 5 words}.

Problem: {2-3 sentence description of what's broken or missing and why it matters}.
Success looks like: {one measurable outcome — e.g. "checkout completes in <500ms p99"}.

Constraints:
- {compliance requirement, e.g. "GDPR: no PII in event payload"}
- {performance SLA if known}
- {deadline if hard}

Out of scope:
- {explicit exclusions — prevents scope creep}

Open questions I already know about:
- {?}
```

---

## Prompting a WORKER

```
Implement task #{N} from FEAT-{ID}.

Plan: docs/plans/FEAT-{ID}-{slug}.md (already approved)
Your task row: "{copy the exact row from the decomposition table}"

Before writing code:
- Restate this task in your own words.
- List which files you will create or modify.
- Flag any ambiguity as UNCLEAR before proceeding.

Reminder: implement exactly what the plan says. Nothing more.
```

**For Cursor inline prompts (Cmd+K):**
```
{what to do} following CLAUDE.md contracts and .cursor/rules/.
{specific constraint, e.g. "must include tenantId filter" or "must paginate"}
```

---

## Prompting a REVIEWER

```
Review FEAT-{ID}, task #{N}.

Plan: docs/plans/FEAT-{ID}-{slug}.md
Diff: {PR link or paste diff}

Run the full checklist from .cursor/rules/06-review-protocol.mdc.
Be adversarial.
- Trace the happy path: where can this fail silently?
- Trace the top 3 error paths: are they handled and tested?
- Think like an attacker: where is the exploit surface?

Output: docs/reviews/FEAT-{ID}-review.md
Verdict must be: APPROVED | APPROVED_WITH_COMMENTS | CHANGES_REQUIRED
CHANGES_REQUIRED: file:line reference + exact fix required for every block.
```

---

## Prompting a COMPOUND AGENT

```
Run compound extraction for FEAT-{ID}.

Inputs:
- Plan: docs/plans/FEAT-{ID}-{slug}.md
- Review: docs/reviews/FEAT-{ID}-review.md
- Diff: {PR link or paste diff}

Extract:
1. Any reusable pattern (Problem → Context → Solution → Trade-offs → Example)
   → docs/patterns/PAT-{NNN}-{slug}.md
2. Any architectural decision made → governance/decision-records/ADR-{NNN}-{slug}.md
3. Any anti-pattern the review caught → append to CLAUDE.md#anti-patterns
4. Update CLAUDE.md#compounding-index with new entries.

Make surgical additions only. Do not rewrite existing content.
```

---

## Anti-Patterns in Prompting

| ❌ Bad | ✅ Good |
|--------|--------|
| "Fix the bug" | "Fix the null pointer in `order.service.ts:142` when `userId` is missing" |
| "Add tests" | "Add unit tests for `CreateOrderUseCase` covering: out-of-stock, invalid userId, success path" |
| "Improve performance" | "Eliminate the N+1 query in `order.repository.ts:getOrdersWithItems()` using eager load" |
| "Make it secure" | "Add Zod validation for the request body in `POST /v1/orders` handler" |
| "Refactor this" | (don't ask for refactors unless in scope — agents will over-engineer) |

---

## Cursor-Specific Tips

- **`@file`** — reference a specific file for context: `@order.service.ts add correlationId logging`
- **`@docs`** — reference docs folder: `@CLAUDE.md what's the error envelope shape?`
- **Cmd+K in editor** — keep prompts to one constraint at a time; Cursor works best with focused, single-responsibility edits
- **Chat vs inline** — use Chat for multi-file tasks; use inline (Cmd+K) for single-function changes
