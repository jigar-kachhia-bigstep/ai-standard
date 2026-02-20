# Plan: {FEAT-ID} — {Feature Name}

**Status**: Draft | Approved  ← Human sets this manually after reviewing the plan
**Complexity**: S | M | L | XL — {one sentence justification}
**Team / Ticket**: {team} / {link}

> Get Approved status before writing any code.
> Approved status does not trigger any automation — it is a signal to the team that this plan is safe to implement.

---

## Why & What

{Problem being solved. What success looks like in one measurable sentence.}

**In scope**: {list}
**Out of scope**: {list}

---

## Services / Modules Affected

| Service | Change | Owner |
|---------|--------|-------|
| {name}  | {new endpoint / DB change / event} | {team} |

---

## Data Model Changes

```sql
-- migration (must use CONCURRENTLY for indexes)

-- rollback:
```

---

## API Changes

```
POST /v{n}/{resource}
Headers: Idempotency-Key (required)
Request:  { ... }
Response: { data: { id, status }, meta: { requestId } }
Errors:   {CODE: description}
```

---

## Events

| Topic | Direction | Consumers / Producers |
|-------|-----------|----------------------|
| `domain.agg.event.v1` | emits / consumes | {service} |

---

## Task Decomposition

Break into atomic WORKER tasks — each task = one PR, one reviewable unit.

| # | Task | Service | Depends on |
|---|------|---------|-----------|
| 1 | {create DB migration + rollback script} | {service} | — |
| 2 | {add domain entity + unit tests} | {service} | 1 |
| 3 | {add use case + unit tests} | {service} | 2 |
| 4 | {add HTTP handler + integration tests} | {service} | 3 |
| 5 | {emit / consume event + idempotency} | {service} | 3 |

---

## Testing Strategy

| Type        | What's covered                      |
|------------|-------------------------------------|
| Unit        | {use cases, domain logic}           |
| Integration | {endpoints, DB, event handlers}     |
| E2E         | {critical user flows if applicable} |

---

## Security & Compliance

- [ ] All new inputs validated at external boundary
- [ ] Auth checks on all new endpoints
- [ ] Ownership check after every resource fetch
- [ ] No PII in logs
- [ ] Multi-tenant: tenantId on all new DB queries
- [ ] {compliance-specific check}

---

## Rollback Plan

- Feature flag: `FEATURE_{NAME}=false` → {behavior}
- DB rollback: `scripts/rollback/{FEAT-ID}.sql` (tested before deploy)
- Who can trigger rollback: {on-call / any engineer}
- SLO for rollback completion: {5 min / 30 min}

---

## Open Questions

| # | Question | Owner | Due | Answer |
|---|---------|-------|-----|--------|
| 1 | {?}     | {name}| {date} | — |

*Unresolved questions with no answer by approval date block approval.*

---

## Definition of Done

- [ ] All task decomposition PRs merged
- [ ] Tests passing, coverage ≥80%
- [ ] Reviewer verdict: APPROVED
- [ ] Staged and smoke-tested
- [ ] Runbook updated if new ops procedures needed
- [ ] Compound Agent has run (patterns + ADRs extracted)
