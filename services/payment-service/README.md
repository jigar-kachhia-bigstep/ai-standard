# payment-service

**Domain**: Financial
**Owner**: Financial Team
**On-Call**: 24/7 — See `governance/domain-boundaries.md`

---

## Purpose

Processes all payment transactions. The only service that handles payment credentials and communicates with payment providers (Stripe, etc.).

---

## Responsibilities

- Process payment charges and refunds
- Manage payment methods (tokenized — no raw card data stored)
- Maintain an immutable payment ledger
- Emit payment events for downstream services

## NOT Responsible For

- Order management → `order-service`
- Fraud scoring → `ai-service`

---

## ⚠️ PCI-DSS Notice

This service is in scope for PCI-DSS. All changes require:
- Security Agent review
- Financial team lead approval
- Changes to payment data handling require security team sign-off

See `governance/compliance/pci-controls.md`

---

## API

Internal only — not exposed through the public API gateway.
Called via `order-service` event → `financial.payment.process.v1`

---

## Events

**Publishes**:
- `financial.payment.processed.v1`
- `financial.payment.failed.v1`
- `financial.payment.refunded.v1`

**Consumes**:
- `commerce.order.created.v1` → process payment

---

## Local Development

```bash
npm install
docker-compose up -d
npm run db:migrate
npm run dev
```

**Note**: Use Stripe test mode keys from `.env.example`. Never use live keys locally.
