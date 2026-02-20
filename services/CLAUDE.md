# Services Enforcement Rules

> Applies to: `services/**/*`, `apps/**/*`, `shared/**/*`
> These are enforcement rules. Architecture reasoning lives in the root `CLAUDE.md`.

---

## Architecture Boundaries

### Service Isolation

```typescript
// ❌ importing another service's src or DB entities
import { OrderEntity } from '../../order-service/src/entities/order.entity';

// ✅ shared types via shared/, cross-service reads via API client
import { OrderDto } from '@shared/domain/order.types';
const order = await orderServiceClient.getOrder(orderId);
```

**Communication rules (non-negotiable):**
- Same-domain reads → synchronous REST/gRPC only
- All state mutations → async event/queue
- Cross-domain → async event/queue
- Never → direct DB access or `services/x` imports from `services/y`

### Layer Dependency Direction

```
Controllers → Use Cases → Domain ← Infrastructure
```

Domain defines interfaces. Infrastructure implements them. Domain **never** imports infrastructure.

```typescript
// ❌ domain importing DB client
import { PrismaClient } from '@prisma/client'; // inside domain/

// ✅ domain defines contract; infra implements it
export interface OrderRepository { save(o: Order): Promise<void>; }
```

### API Contracts

- Routes: `GET|POST|PUT|PATCH|DELETE /v{n}/{plural-resource}`
- Response shape: always the standard envelope from root `CLAUDE.md`
- `POST` / `PUT`: require `Idempotency-Key` header — 400 if missing, 200 (original result) on duplicate
- Versioning: URL path (`/v1/`, `/v2/`). Old version supported ≥90 days after new version ships
- Breaking change → new path version. Non-breaking (add optional field) → no bump

### Event Contracts

- Topic format + envelope fields: defined in root `CLAUDE.md`
- Breaking schema change → new topic version (`.v2`). Both versions live ≥30 days
- All consumers **must** deduplicate on `eventId`

### Multi-Tenancy

```typescript
// ❌ missing tenant scope
db.order.findMany({ where: { userId } });

// ✅ always include tenantId — from ctx, never from global state
db.order.findMany({ where: { userId, tenantId: ctx.tenantId } });
```

---

## Security Guardrails

### Auth (every protected endpoint)

```typescript
router.use(authMiddleware); // validates JWT; injects ctx.userId, ctx.tenantId, ctx.roles

// After fetching a resource, always verify ownership explicitly
if (order.userId !== ctx.userId) throw new ForbiddenError('FORBIDDEN');
```

### Input Validation (every external boundary — no exceptions)

```typescript
const result = CreateOrderSchema.safeParse(req.body);
if (!result.success) return res.status(422).json({ error: { code: 'VALIDATION_ERROR', details: result.error } });
// use result.data — typed and safe
```

### Injection Prevention

```typescript
// ❌ string interpolation in SQL
db.query(`WHERE id = '${id}'`);
// ✅ parameterized only
db.query('WHERE id = $1', [id]);
```

### Secrets

```typescript
// ❌ hardcoded anywhere or committed in .env
// ✅ process.env.SECRET_KEY — injected by platform (Vault / Secrets Manager / K8s secret)
// .env must be in .gitignore
```

### Never Log (hard block)

Passwords · tokens · card numbers (PAN/CVV) · SSNs · bank accounts · private keys · `apiKey` / `secret` fields

```typescript
// ✅ log type and amount — never raw card data
logger.info('Payment processed', { orderId, paymentMethod: 'credit_card', amount });
```

### Headers & Rate Limiting

```typescript
app.use(helmet());
app.use(cors({ origin: process.env.ALLOWED_ORIGINS?.split(',') })); // never wildcard in prod
app.use('/api', rateLimit({ windowMs: 60_000, max: 100, standardHeaders: true }));
```

### Audit Log (auth events · data mutations · admin actions)

```typescript
auditLog({ action: 'order.created', actor: { userId: ctx.userId },
  resource: { type: 'order', id: order.id }, outcome: 'success',
  ip: req.ip, correlationId: ctx.correlationId, timestamp: new Date().toISOString() });
```

---

## Testing Standards

**Coverage gate: ≥80% unit per service. CI blocks merge below this.**

| Type        | Requirement |
|------------|-------------|
| Unit        | ≥80% line/branch per service |
| Integration | All new API endpoints (happy + error path) |
| E2E         | Critical user flows only |

### Structure

- Colocate test next to source: `order.service.ts` → `order.service.test.ts`
- No separate top-level `__tests__/` folder

### AAA Pattern (required)

```typescript
describe('CreateOrderUseCase / when item is out of stock', () => {
  it('should throw InsufficientStockError and not save the order', async () => {
    // Arrange
    const deps = buildDeps({ inventory: { inStock: false } });
    // Act & Assert
    await expect(new CreateOrderUseCase(deps).execute(buildCmd()))
      .rejects.toThrow(InsufficientStockError);
    expect(deps.orderRepo.save).not.toHaveBeenCalled();
  });
});
```

### Hard Rules

```typescript
// ❌ conditional logic in assertions → ✅ deterministic expect() calls
// ❌ hardcoded UUIDs → ✅ faker.string.uuid() in factory
// ❌ production data → ✅ factories only
// ❌ shared mutable DB state between tests → ✅ TRUNCATE in afterEach
// ❌ staging/prod DB in tests → ✅ testcontainers or local test DB
```

### Factory Pattern (required for all domain objects)

```typescript
export function buildOrder(overrides: Partial<Order> = {}): Order {
  return { id: faker.string.uuid(), status: 'PENDING', ...overrides };
}
```

### Integration Tests (real HTTP stack, both paths per endpoint)

```typescript
it('POST /v1/orders — 201 on valid input', async () => {
  const res = await request(app).post('/v1/orders')
    .set('Authorization', `Bearer ${token}`).send(buildOrderPayload());
  expect(res.status).toBe(201);
  expect(res.body.data.id).toBeDefined();
});

it('POST /v1/orders — 422 when items is empty', async () => {
  const res = await request(app).post('/v1/orders')
    .set('Authorization', `Bearer ${token}`).send({ items: [] });
  expect(res.status).toBe(422);
  expect(res.body.error.code).toBe('VALIDATION_ERROR');
});
```

### What to Test

| Layer           | Type        | Focus |
|----------------|-------------|-------|
| Domain entities | Unit        | Business rules, invariants |
| Use cases       | Unit        | Orchestration, all error paths |
| HTTP handlers   | Integration | Auth, request parsing, response shape |
| Repositories    | Integration | Queries, transactions, tenant scoping |
| Event consumers | Unit        | Idempotency, error paths, compensation |

---

## Performance Rules

### Queries

```typescript
// ❌ N+1
for (const order of orders) { await db.orderItem.findMany({ where: { orderId: order.id } }); }
// ✅ eager load
const orders = await db.order.findMany({ where: { userId }, include: { items: true } });
```

```typescript
// ❌ unbounded list
// ✅ always paginate — default 20, max 100, cursor-based
const rows = await db.order.findMany({
  where: { userId }, take: limit,
  cursor: cursor ? { id: cursor } : undefined,
  orderBy: { createdAt: 'desc' }
});
```

```sql
-- ❌ CREATE INDEX ON orders(user_id);         ← locks table
-- ✅ CREATE INDEX CONCURRENTLY ON orders(user_id);
```

### Caching (TTL defaults from root CLAUDE.md)

```typescript
async function getProduct(id: string, ctx: Ctx): Promise<Product> {
  const key = `t:${ctx.tenantId}:product:${id}`; // always tenant-scoped
  const hit = await redis.get(key);
  if (hit) return JSON.parse(hit);
  const product = await db.product.findUnique({ where: { id } });
  await redis.setex(key, 300, JSON.stringify(product));
  return product;
}

// Invalidate on write — if del() fails, log WARN and let TTL expire
await db.product.update({ where: { id }, data });
await redis.del(`t:${ctx.tenantId}:product:${id}`).catch(e => logger.warn('Cache del failed', e));
```

### Async

```typescript
// ❌ sequential awaits for independent work
const user = await getUser(id);
const prefs = await getPrefs(id);
// ✅
const [user, prefs] = await Promise.all([getUser(id), getPrefs(id)]);

// ❌ CPU-heavy sync work on event loop → ✅ offload to worker thread or job queue
```

### Connections & Outbound Calls

```typescript
// Explicit pool size — never rely on defaults
const pool = new Pool({ max: 10, idleTimeoutMillis: 30_000, connectionTimeoutMillis: 2_000 });

// Timeout on every outbound HTTP call
await fetch(url, { signal: AbortSignal.timeout(5_000) });

// Circuit breaker on every downstream service dependency
const result = await circuitBreaker.fire(downstreamCall);
```

### Frontend

```tsx
// ❌ spinner for content areas → ✅ skeleton screens
// ❌ sort/filter inside render → ✅ useMemo
// ❌ render all N items → ✅ virtualize lists >50 items
// ❌ eager-load all components → ✅ lazy() + Suspense for below-fold
```
