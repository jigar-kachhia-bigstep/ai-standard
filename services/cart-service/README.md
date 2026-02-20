# cart-service

**Domain**: Commerce
**Owner**: Commerce Team

## Purpose

Manages the shopping cart lifecycle: add/remove items, apply coupons, calculate totals.

## API

- `GET /v1/cart` — get current cart
- `POST /v1/cart/items` — add item
- `DELETE /v1/cart/items/:itemId` — remove item
- `POST /v1/cart/checkout` — initiate checkout (hands off to order-service)

## Data

- Cart state stored in Redis (TTL: 30 days)
- Persisted to DB on checkout initiation for audit trail

## Local Development

```bash
npm install && docker-compose up -d && npm run dev
```
