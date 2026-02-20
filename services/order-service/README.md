# order-service

**Domain**: Commerce
**Owner**: Commerce Team
**On-Call**: See `governance/domain-boundaries.md`

---

## Purpose

Manages the lifecycle of customer orders: creation, payment confirmation, fulfilment, and cancellation.

---

## Responsibilities

- Create and persist orders
- Manage order state machine (PENDING → CONFIRMED → FULFILLING → COMPLETE | CANCELLED)
- Coordinate the checkout saga (via events)
- Expose order read APIs for storefront and admin portal

## NOT Responsible For

- Payment processing → `payment-service`
- Inventory management → `catalog-service`
- Sending notifications → `notification-service`

---

## API

See `src/api/openapi.yaml` for full spec.

**Key endpoints**:
- `POST /v1/orders` — create order
- `GET /v1/orders/:id` — get order
- `GET /v1/orders` — list orders (paginated, cursor-based)
- `POST /v1/orders/:id/cancel` — cancel order

---

## Events

**Publishes**:
- `commerce.order.created.v1`
- `commerce.order.confirmed.v1`
- `commerce.order.cancelled.v1`

**Consumes**:
- `financial.payment.processed.v1` → confirm order
- `financial.payment.failed.v1` → cancel order

---

## Local Development

```bash
# Install dependencies
npm install

# Start with local DB and message bus
docker-compose up -d

# Run migrations
npm run db:migrate

# Start in dev mode
npm run dev
```

---

## Runbook

`docs/runbooks/order-service.md`
