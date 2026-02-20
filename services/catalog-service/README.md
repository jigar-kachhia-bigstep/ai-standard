# catalog-service

**Domain**: Commerce
**Owner**: Commerce Team

## Purpose

Manages the product catalog: product listings, categories, pricing, and inventory levels.

## API

- `GET /v1/products` — search/list products
- `GET /v1/products/:id` — get product detail
- `POST /v1/products` — create product (seller/admin only)
- `PUT /v1/products/:id` — update product
- `GET /v1/categories` — list categories

## Data

- Product data: MongoDB (flexible schema for varied product types)
- Inventory counts: PostgreSQL (consistency required)
- Cache: Redis (5-minute TTL for product reads)

## Local Development

```bash
npm install && docker-compose up -d && npm run dev
```
