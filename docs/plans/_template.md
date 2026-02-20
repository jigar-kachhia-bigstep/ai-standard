# Plan: {FEAT-ID} — {Feature Name}

**Status**: Draft | Approved
**Team / Ticket**: {team} / {link}

> Get Approved status before writing any code.

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

## Testing Strategy

| Type        | What's covered                     |
|------------|-----------------------------------|
| Unit        | {use cases, domain logic}          |
| Integration | {endpoints, DB, event handlers}    |
| E2E         | {critical user flows if applicable} |

---

## Security & Compliance

- [ ] All new inputs validated
- [ ] Auth checks on new endpoints
- [ ] No PII in logs
- [ ] {compliance-specific check}

---

## Rollback Plan

- Feature flag: `FEATURE_{NAME}=false` → {behavior}
- DB rollback: `scripts/rollback/{FEAT-ID}.sql` (tested before deploy)
- Who can trigger rollback: {on-call / any engineer}
- SLO for rollback completion: {5 min / 30 min}

---

## Open Questions

| # | Question | Owner | Due |
|---|---------|-------|-----|
| 1 | {?}     | {name}| {date} |

---

## Definition of Done

- [ ] Tests passing, coverage ≥80%
- [ ] Reviewer verdict: APPROVED
- [ ] Staged and smoke-tested
- [ ] Runbook updated if new ops procedures needed
