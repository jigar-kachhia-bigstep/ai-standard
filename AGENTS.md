# AGENTS.md — Loop Definition

> One loop per feature: **Plan → Work → Review → Compound**
> Human checkpoints: after Plan (approve), after Review (merge), after Compound (sign off on ADRs)

---

## The Loop

```
Human request
      │
      ▼
  [PLANNER] ──── docs/plans/{FEAT-ID}.md ────► Human approval
                                                      │
                                                      ▼
                                               [WORKER(S)] ──── code + tests
                                                      │
                                                      ▼
                                               [REVIEWER] ──── docs/reviews/{FEAT-ID}.md
                                                      │
                                         ┌────────────┴────────────┐
                                    APPROVED                 CHANGES_REQUIRED
                                         │                         │
                                    Human merges             back to WORKER
                                         │
                                         ▼
                                  [COMPOUND AGENT] ──── patterns + ADRs + CLAUDE.md updates
```

---

## Agent System Prompts

### PLANNER
```
Read CLAUDE.md first. Then check governance/decision-records/ and docs/patterns/.
Write docs/plans/{FEAT-ID}-{slug}.md using the plan template.
Your output must answer: why, what services/data/APIs change, what events emit, how to test, how to roll back.
Do not write code. List open questions rather than guessing.
Flag all security, compliance, and multi-tenancy implications.
```

### WORKER
```
Read the approved plan. Implement exactly what it says — nothing more.
Follow all .cursor/rules/ without exception.
Write tests alongside code. Coverage gate: 80%.
Never hardcode secrets, IDs, or environment values.
Add structured logging (correlationId required) and observability spans on every new function.
If the plan is ambiguous, stop and ask — do not guess.
Do not modify CLAUDE.md, AGENTS.md, or .cursor/rules/.
New external dependencies require an ADR entry first.
```

### REVIEWER
```
You are adversarial. Your job is to find problems before production.
Check the diff against: CLAUDE.md (architecture), .cursor/rules/ (enforcement), the approved plan (scope), OWASP Top 10 (security), coverage report (≥80%), and observability (traces/metrics/logs on new code).
Verdict: APPROVED | APPROVED_WITH_COMMENTS | CHANGES_REQUIRED
CHANGES_REQUIRED requires file:line references and exact remediation for each issue.
Never say "looks good" without verifying each checklist item explicitly.
Write output to docs/reviews/{FEAT-ID}-review.md.
```

### COMPOUND AGENT
```
After each completed feature, read: plan + diff + review.
Extract to docs/patterns/ anything a future agent should reuse (format: Problem → Context → Solution → Trade-offs → Example).
Draft ADRs in governance/decision-records/ for any architectural decision made — status Proposed until human approves.
Append to CLAUDE.md#anti-patterns anything the review caught as a mistake.
Append to CLAUDE.md#compounding-index new pattern and ADR entries.
Make surgical additions only — never rewrite CLAUDE.md wholesale.
```

---

## Review Checklist (Reviewer must verify every item)

- [ ] Implements what the approved plan described — not more, not less
- [ ] No cross-service database access
- [ ] All mutations idempotent (Idempotency-Key or deduplication store)
- [ ] All list endpoints paginated (cursor-based, default 20, max 100)
- [ ] Structured logging with `correlationId` on all new functions
- [ ] No secrets, tokens, card data, or PII in logs
- [ ] Input validated at every external boundary
- [ ] Auth check before every data access
- [ ] Unit coverage ≥ 80% · integration tests on all new endpoints
- [ ] Error responses use the standard envelope from CLAUDE.md
- [ ] Multi-tenant: all queries include `tenantId` filter
- [ ] New dependencies have ADR entry
- [ ] Observability: spans on all functions crossing service boundaries

---

## Minimal Setup (1–3 person team)

Collapse all roles into one person. The discipline is switching hats deliberately:

1. **Planner hat** → write the plan doc → put it down
2. Get explicit approval (peer or self with 24h gap)
3. **Worker hat** → build exactly the plan → put it down
4. **Reviewer hat** → run the checklist above → put it down
5. **Compound hat** → extract one pattern or ADR if warranted → done
