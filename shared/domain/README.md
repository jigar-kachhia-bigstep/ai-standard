# shared/domain

Shared domain types, contracts, and interfaces used across services and apps.

## What Lives Here

- **DTO types** (`order.types.ts`, `product.types.ts`) — shared data shapes
- **Event schemas** (`events/`) — TypeScript types for all Kafka events
- **Error types** (`errors/`) — standard error classes shared across services
- **Constants** (`constants/`) — shared enums and constant values

## What Does NOT Live Here

- Business logic (stays in each service's domain layer)
- Database-specific code
- Framework-specific code

## Usage

```typescript
import { OrderDto, OrderStatus } from '@shared/domain/order.types';
import { OrderCreatedEvent } from '@shared/domain/events/order.events';
```

## Adding New Types

1. Add the type file in the appropriate subdirectory
2. Export from the subdirectory's `index.ts`
3. Export from this package's root `index.ts`
4. If it's a new event schema, also register in the Avro schema registry (if applicable)
