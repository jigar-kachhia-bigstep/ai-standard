# Architecture Principles

> Expands on the 5 principles in CLAUDE.md. Read CLAUDE.md first for priority order and conflict rule.
> Reference principle IDs in ADRs and code reviews. ("This violates P3.")

---

## P1 — Correctness

**Statement**: Wrong and fast is worse than right and slow.

**Means**: Validate inputs even at latency cost · use pessimistic locking when consistency matters · never cache data that's dangerous stale (payment status, auth state)

**Violated when**: Skipping validation "for performance" · removing retries because they're slow

---

## P2 — Explicit

**Statement**: Side effects, contracts, and dependencies must be declared — not inferred.

**Means**: Functions declare what they need (no hidden globals) · events have published schemas · service contracts are versioned and discoverable (OpenAPI / schema registry)

**Violated when**: Functions reading from global/ambient state · events with undocumented fields · services that break when a dependency changes something "unrelated"

---

## P3 — Secure

**Statement**: Auth, validation, and audit are load-bearing — not afterthoughts.

**Means**: Every endpoint validates input before business logic · every protected resource checks ownership · sensitive operations emit audit entries

**Violated when**: "It's an internal service, skip auth" · trusting client-provided IDs without verification · missing audit trail on financial or PII mutations

---

## P4 — Observable

**Statement**: If you can't measure it, you can't operate it.

**Means**: Every new cross-boundary function gets a trace span · every new endpoint emits rate/error/latency metrics · every error is logged with enough context to reproduce it

**Violated when**: "It failed but we don't know why" · debugging requires SSH into production · alerts fire with no actionable context

---

## P5 — Simple

**Statement**: The minimum complexity that correctly solves the real problem.

**Means**: No abstractions for a single call site · no configurability for non-existent use cases · no backward-compat shims for code you own entirely

**Violated when**: A helper used once · a config flag with one ever-used value · an interface with one implementation that needs no test double

---

## Conflict Resolution

| Conflict                    | Resolution                                                            |
|----------------------------|-----------------------------------------------------------------------|
| P1 vs P5                   | Correctness wins — never skip validation for simplicity               |
| P3 vs P5                   | Secure wins — never skip auth for simplicity                          |
| P4 vs P5                   | Add instrumentation at service boundaries; internals can be pragmatic |
| P2 vs P5                   | Explicit at public contracts; pragmatic inside private modules        |
