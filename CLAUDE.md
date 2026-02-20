# CLAUDE.md — Architecture Memory

> **Reasoning brain.** `.cursor/rules/` enforces. This file explains *why* and *what*.
> Read this before every architectural decision. Never put enforcement rules here.
> Keep this file ≤150 lines. Move verbose specs to `docs/` and link.

---

## System Identity

<!-- Fill every field before your first AI-assisted task. Unfilled fields = bad context. -->

| Property         | Value                                              |
|------------------|----------------------------------------------------|
| **Project**      | {YOUR_PROJECT_NAME}                                |
| **Type**         | {B2C / SaaS / Internal / Marketplace}              |
| **Architecture** | {Microservices / Modular Monolith / Monolith}      |
| **Frontend**     | {React / Next.js / Vue / Angular} + version        |
| **Backend**      | {Node.js / Go / .NET / Python} + version           |
| **Package mgr**  | {npm / pnpm / bun / yarn} — used for all services  |
| **ORM / DB**     | {Prisma / TypeORM / Drizzle} + {Postgres / MongoDB}|
| **Test runner**  | {Jest / Vitest / Pytest}                           |
| **Infra**        | {AWS / GCP / Azure / Kubernetes}                   |
| **Compliance**   | {PCI / GDPR / HIPAA / None}                        |
| **Multi-tenant** | {Yes / No}                                         |
| **AI**           | {Yes / No}                                         |

---

## Principles (priority order — higher wins on conflict)

1. **Correctness** — Wrong and fast is worse than right and slow.
2. **Explicit** — Side effects, contracts, and dependencies must be declared.
3. **Secure** — Auth, validation, and audit are never optional.
4. **Observable** — If you can't measure it, you can't operate it.
5. **Simple** — The minimum complexity that solves the real problem.

**Conflict rule**: Higher-numbered principle yields. #5 Simple may yield to #3 Secure — never skip auth for simplicity.
Full principle definitions: [`governance/architecture-principles.md`](governance/architecture-principles.md)

---

## Domain Map

See full ownership + cross-domain workflows: [`governance/domain-boundaries.md`](governance/domain-boundaries.md)

```
┌──────────────────────────────────────────────┐
│               API / BFF Layer                │
└────────────────┬─────────────────────────────┘
                 │
     ┌───────────▼────────────────────────────┐
     │  Commerce: cart · order · catalog      │
     ├────────────────────────────────────────┤
     │  Financial: payment                    │
     ├────────────────────────────────────────┤
     │  Platform: notification                │
     └────────────────────────────────────────┘
```

**Communication rules:**
- **Synchronous**: reads only, latency-sensitive, same-domain
- **Async (events/queues)**: all state mutations, cross-domain workflows
- **Never**: direct DB access across service boundaries

---

## System Contracts

### Error Response

```typescript
// Success
{ data: T, meta?: { requestId: string, pagination?: { nextCursor: string, hasMore: boolean } } }
// Error
{ error: { code: string, message: string, requestId: string, retryable: boolean, details?: unknown } }
```

**Error codes**: `VALIDATION_ERROR` · `UNAUTHORIZED` · `FORBIDDEN` · `NOT_FOUND` · `CONFLICT` · `RATE_LIMITED` · `INTERNAL_ERROR`

### Pagination
- Cursor-based: `?cursor=&limit=`. Default 20, max 100. Never unbounded.
- Empty: `{ data: [], meta: { hasMore: false } }` — never 404.

### Events
```
{domain}.{aggregate}.{event}.v{n}  →  commerce.order.created.v1
```
- Breaking change = new topic version (`.v2`). Both live ≥30 days.
- Required fields: `eventId` · `tenantId` · `correlationId` · `causationId` · `timestamp` · `version`
- Consumers deduplicate on `eventId`.

### Tenant Context
- Source of truth: JWT `tenantId` → injected as `x-tenant-id` header by gateway.
- Pass as `ctx.tenantId` through every function. Never read from global state.
- Every DB query filters by `tenantId`. RLS is defense-in-depth, not primary control.

### Logging Levels

| Level | When                                                    |
|-------|---------------------------------------------------------|
| ERROR | Unhandled exception, data loss risk, service degraded   |
| WARN  | Expected failure (validation, 404), perf degradation    |
| INFO  | Business event (order created, payment processed)       |
| DEBUG | Dev/staging only — stripped in production               |

Required fields: `correlationId`, `severity`, `service`. Add `tenantId`, `userId` when available.
**Never log**: passwords · tokens · card numbers · SSNs · secrets.

### Cache TTL Defaults
`user-facing reads: 60s` · `catalog: 300s` · `config: 3600s` · `session: explicit TTL`
Multi-tenant keys: `t:{tenantId}:{resource}:{id}`

### Deployment
```
branch → CI (lint + test + security) → staging (auto) → manual approval → canary 5% → 25% → 100%
```
Auto-rollback: error rate >1% for 5 min · p99 latency >2× baseline for 5 min.

---

## Service → Data Store Map

<!-- Add a row per service as you build. -->

| Service              | DB       | Cache | Reason         |
|---------------------|----------|-------|----------------|
| order-service        | Postgres | Redis | transactional  |
| catalog-service      | Postgres | Redis | high read rate |
| payment-service      | Postgres | —     | consistency    |
| cart-service         | Redis    | —     | ephemeral      |
| notification-service | Postgres | —     | audit trail    |

---

## Compounding Index

### Patterns (`docs/patterns/`)
| ID      | Name | Status |
|---------|------|--------|
| *(none yet — added by Compound Agent after each feature)* | | |

### Decisions (`governance/decision-records/`)
| ID      | Title | Status |
|---------|-------|--------|
| *(none yet — added by Compound Agent after each feature)* | | |

---

## Anti-Patterns

1. **Shared DB across services** — creates a distributed monolith.
2. **God events** — entire entity state in one event instead of meaningful deltas.
3. **Business logic in gateway** — gateway routes, it does not decide.
4. **Global tenant state** — tenant context must flow through the call stack, never globals.
5. **Synchronous saga chains** — A→B→C→D sync chains multiply latency and failures.
