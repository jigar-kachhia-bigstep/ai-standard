# Review Protocol Enforcement

> Applies to: `docs/reviews/**/*`
> These are enforcement rules. Architecture reasoning lives in the root `CLAUDE.md`.

---

## Verdicts

| Verdict | Meaning |
|---------|---------|
| `APPROVED` | All checklist items pass. Ready to merge. |
| `APPROVED_WITH_COMMENTS` | Minor issues only. Merge after acknowledging. |
| `CHANGES_REQUIRED` | Blocking issue(s). Fix before merge. File:line required. |

---

## Before Running the Checklist

1. Trace the happy path end-to-end. Where can it fail silently?
2. Trace the top 3 error paths. Are they handled and tested?
3. Think adversarially: OWASP Top 10, tenant isolation, input validation, auth bypass.
4. Only then run the checklist below.

---

## Checklist (verify every item — do not assume)

**Correctness**
- [ ] Implements what the approved plan described — not more, not less
- [ ] Edge cases covered (empty, null, zero, max)

**Architecture** (rules in `services/CLAUDE.md` § Architecture Boundaries)
- [ ] No cross-service DB access · shared code in `shared/` only
- [ ] API responses use the standard envelope (see root `CLAUDE.md`)
- [ ] New dependencies have ADR entry

**Security** (rules in `services/CLAUDE.md` § Security Guardrails)
- [ ] All external input validated · auth check before data access
- [ ] No PII / secrets / tokens in logs · SQL parameterized

**Testing** (rules in `services/CLAUDE.md` § Testing Standards)
- [ ] Unit coverage ≥80% · integration tests on all new endpoints
- [ ] Error paths tested · factories used (no hardcoded IDs)

**Observability**
- [ ] Structured logs with `correlationId` on all new functions
- [ ] Spans on all functions crossing service boundaries

**Performance** (rules in `services/CLAUDE.md` § Performance Rules)
- [ ] No N+1 · all lists paginated · cache invalidated on mutation

**Operations**
- [ ] Config from env vars · health check still passes
- [ ] DB migration: `CONCURRENTLY` index · rollback script present

---

## Blocking Issue Format

```
#### BLOCK-{N} — {title}
**Severity**: CRITICAL | HIGH | MEDIUM
**File**: path/to/file.ts:{line}
**Found**: {exact problematic code}
**Required**: {correct pattern}
**Rationale**: {rule reference + why it matters}
```

---

## Review Output Format

Save to `docs/reviews/{FEAT-ID}-review.md`:

```markdown
# Review: {FEAT-ID} — {Name}
**Verdict**: APPROVED | APPROVED_WITH_COMMENTS | CHANGES_REQUIRED
**Date / PR**: {date} / #{pr}

## Summary
{2-3 sentences}

## Blocking Issues
{BLOCK-N entries or "None"}

## Checklist
| Area          | Result | Notes |
|--------------|--------|-------|
| Correctness  | ✅/❌  |       |
| Architecture | ✅/❌  |       |
| Security     | ✅/❌  |       |
| Testing      | ✅/❌  |       |
| Observability| ✅/❌  |       |
| Performance  | ✅/❌  |       |
| Operations   | ✅/❌  |       |
```
