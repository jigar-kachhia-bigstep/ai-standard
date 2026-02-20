# Domain Boundaries

> Defines which team owns which services and which data.
> Updated when ownership changes. Referenced by agents to know who to consult.

---

## Ownership Map

<!-- Update this for your project -->

| Domain         | Services / Modules                          | Owns Data              | Team        | Slack        |
|---------------|---------------------------------------------|------------------------|-------------|--------------|
| Commerce       | cart-service, order-service, catalog-service | orders, cart, products | Commerce    | #team-commerce |
| Financial      | payment-service                              | payments, refunds      | Financial   | #team-fintech |
| Notifications  | notification-service                         | notifications, templates | Platform  | #team-platform |
| Frontend       | web-storefront, admin-portal, seller-dashboard | —                   | Frontend    | #team-frontend |

---

## Boundary Rules

### Data Ownership

- A service owns its data exclusively
- No other service may read from or write to it directly
- To access data: call the owning service's API, or subscribe to its published events

### Cross-Domain Workflows

Define your cross-domain workflows here so agents understand the flow:

```
Checkout Flow:
  cart-service → order-service → payment-service → notification-service
  (via events, not synchronous chains)

Order Cancellation:
  order-service → payment-service (refund) → notification-service (email)
```

### Shared Data (Denormalized Read Models)

Sometimes services need read-only copies of another domain's data for performance:

| Read Model              | Source Domain | Consuming Service | Sync Method         |
|------------------------|--------------|------------------|---------------------|
| {product-summary}      | Commerce     | order-service    | Kafka event         |
| {user-display-name}    | Identity     | notification-svc | Kafka event         |

---

## Adding a New Service

When creating a new service, answer:

1. **What domain does it belong to?** (Add to ownership map above)
2. **What data does it own?** (Define in this document)
3. **What events does it publish?** (Add to CLAUDE.md event catalog)
4. **What does it consume?** (Document consumer dependencies)
5. **Does it need a new ADR?** (If it introduces a new pattern or technology)
