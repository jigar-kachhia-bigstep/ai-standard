# notification-service

**Domain**: Platform
**Owner**: Platform Team

## Purpose

Sends all outbound communications: email, SMS, push notifications. Deduplicates to prevent users from receiving the same notification twice.

## Responsibilities

- Route notifications to correct channel (email, SMS, push)
- Render notification templates
- Deduplicate: same notification to same user within 1-hour window is suppressed
- Track delivery status

## Events Consumed

- `commerce.order.confirmed.v1` → send order confirmation email
- `commerce.order.cancelled.v1` → send cancellation email
- `financial.payment.failed.v1` → send payment failure notification

## Local Development

```bash
npm install && docker-compose up -d && npm run dev
```

**Note**: In local dev, emails are captured by Mailpit at `http://localhost:8025`
