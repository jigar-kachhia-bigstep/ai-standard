# CLAUDE.md — Architecture Memory

> **Reasoning brain.** `.cursor/rules/` enforces. This file explains *why* and *what*.
> Read this before every architectural decision. Never put enforcement rules here.

---

## System Identity

| Property          | Value                                              |
|------------------|----------------------------------------------------|
| **Project**       | {YOUR_PROJECT_NAME}                               |
| **Type**          | {B2C / SaaS / Internal / Marketplace}             |
| **Architecture**  | {Microservices / Modular Monolith / Monolith}     |
| **Frontend**      | {React / Next.js / Vue / Angular}                 |
| **Backend**       | {Node.js / Go / .NET / Python}                    |
| **Database**      | {Postgres / MongoDB / MySQL}                      |
| **Infra**         | {AWS / GCP / Azure / Kubernetes}                  |
| **Compliance**    | {PCI / GDPR / HIPAA / None}                       |
| **Scale**         | {X users/day, Y RPS}                              |
| **Multi-tenant**  | {Yes / No}                                        |
| **AI**            | {Yes / No}                                        |

---

## Principles (priority order — higher wins on conflict)

1. **Correctness** — Wrong and fast is worse than right and slow.
2. **Explicit** — Side effects, contracts, and dependencies must be declared.
3. **Secure** — Auth, validation, and audit are never optional.
4. **Observable** — If you can't measure it, you can't operate it.
5. **Simple** — The minimum complexity that solves the real problem.

**Conflict rule**: When two principles clash, the higher-numbered one yields.
Example: #5 Simple may yield to #3 Secure — never add auth complexity for its own sake, but never skip auth for simplicity.

---

## Domain Map

```
┌──────────────────────────────────────────────┐
│                API / BFF Layer               │
└────────────────┬────────────────┬────────────┘
                 │                │
     ┌───────────▼────┐  ┌────────▼───────┐
     │   Domain A     │  │   Domain B     │
     │ - service-1    │  │ - service-3    │
     │ - service-2    │  │ - service-4    │
     └────────────────┘  └────────────────┘
```

**Communication rules (canonical — referenced from cursor rules, not repeated):**
- **Synchronous**: reads only, latency-sensitive, same-domain
- **Async (events/queues)**: all state mutations, cross-domain workflows
- **Never**: direct DB access across service boundaries

---

## System Contracts

### Error Response (all services must use this shape)

```typescript
// Success
{ data: T, meta?: { requestId: string, pagination?: { nextCursor: string, hasMore: boolean } } }

// Error
{ error: { code: string, message: string, requestId: string, retryable: boolean, details?: unknown } }
```

**Standard error codes**: `VALIDATION_ERROR` · `UNAUTHORIZED` · `FORBIDDEN` · `NOT_FOUND` · `CONFLICT` · `RATE_LIMITED` · `INTERNAL_ERROR`

### Pagination (all list endpoints)

- Strategy: cursor-based (`?cursor=&limit=`)
- Default limit: 20. Max limit: 100. Never unbounded.
- Empty result: `{ data: [], meta: { hasMore: false } }` — never 404

### Event Naming & Versioning

```
{domain}.{aggregate}.{event}.v{n}   →   commerce.order.created.v1
```

- Breaking schema change = new topic version (`.v2`). Both versions run ≥30 days.
- All events carry: `eventId` · `tenantId` · `correlationId` · `causationId` · `timestamp` · `version`
- Consumers deduplicate on `eventId`. See `docs/patterns/` for implementation.

### Tenant Context (multi-tenant projects)

- Source of truth: JWT claim `tenantId` — injected by API gateway as `x-tenant-id` header
- Must be passed through every function as `ctx.tenantId` — never read from global state
- Every DB query must filter by `tenantId`. RLS is defense-in-depth, not the primary control.

### Logging Levels

| Level  | When to use                                             |
|--------|---------------------------------------------------------|
| ERROR  | Unhandled exception, data loss risk, service degraded   |
| WARN   | Expected failure (validation, 404), perf degradation    |
| INFO   | Business event (order created, payment processed)       |
| DEBUG  | Dev/staging only — stripped in production               |

Required fields on every log: `correlationId`, `severity`, `service`. Add `tenantId`, `userId` when available.
Never log: passwords, tokens, card numbers, SSNs, secrets (see `.cursor/rules/03`).

### Cache Invalidation

- On mutation: `del(key)` after DB write in same operation. If del fails: log WARN + let TTL expire (accept brief staleness).
- TTL defaults: user-facing reads 60s · catalog 300s · config 3600s · session = explicit TTL
- Multi-tenant: all keys prefixed `t:{tenantId}:{resource}:{id}`

### Deployment

```
branch → CI (lint + test + security) → staging (auto) → manual approval → canary 5% → 25% → 100%
```

Auto-rollback triggers: error rate >1% for 5 min · p99 latency >2× baseline for 5 min

---

## Service → Data Store Map

| Service      | DB           | Cache  | Reason         |
|-------------|-------------|--------|----------------|
| {service-1} | {db-type}   | Redis  | {reason}       |
| {service-2} | {db-type}   | —      | {reason}       |

---

## Team Ownership

| Team     | Services              | On-Call  |
|---------|-----------------------|----------|
| {team}  | {service-1, svc-2}   | {Yes/No} |

---

## Compounding Index

### Patterns (`docs/patterns/`)
| ID      | Name               | Status |
|---------|--------------------|--------|
| PAT-001 | {pattern name}     | Draft  |

### Decisions (`governance/decision-records/`)
| ID      | Title              | Status   |
|---------|--------------------|----------|
| ADR-001 | {decision title}   | Proposed |

---

## Anti-Patterns (append after each review cycle)

1. **Shared DB across services** — creates a distributed monolith.
2. **God events** — entire entity state in one event instead of meaningful deltas.
3. **Business logic in gateway** — gateway routes, it does not decide.
4. **Global tenant state** — tenant context must flow through the call stack, never globals.
5. **Synchronous saga chains** — A→B→C→D sync chains multiply latency and failures.
