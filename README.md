# SentinelSP — Amazon Listing Change Monitor Demo

[**Open the live demo**](https://adamhumen123-blip.github.io/amazon-sp-api-listing-monitor-demo/)

A polished, interactive proof-of-concept for monitoring Amazon listings through the Selling Partner API (SP-API), comparing current data with stored baselines, and alerting operators when material changes are detected.

![SentinelSP dashboard](assets/dashboard.png)

## Live demo features

- Catalogue dashboard with listing health, price, inventory and status
- One-click **Simulate change** workflow
- Before/after evidence for title, price, bullet, image and inventory changes
- Search, filters, catalogue management and CSV export
- Alert routing UI for email, Telegram and Slack
- Live event stream and notification centre
- Responsive interface suitable for GitHub Pages
- Demo/live-mode separation so credentials are never placed in frontend code

## Run locally

No build step is required.

```bash
python -m http.server 8080
```

Open `http://localhost:8080`.

You can also open `index.html` directly in a modern browser, although a local server is recommended.

## GitHub Pages deployment

This repository includes a GitHub Actions workflow that publishes the static application whenever `main` is updated. If Pages has not yet been activated for the repository, open **Settings → Pages** and select **GitHub Actions** as the source.

## Production architecture

The public demo deliberately uses realistic mock data. A production build adds a secure backend:

```text
Amazon Seller Central OAuth
          │
          ▼
Secure API service ── LWA token refresh + AWS request signing
          │
          ├── Notifications API / SQS event ingestion
          ├── Listings Items API reconciliation
          ├── Product Pricing and inventory enrichment
          └── Rate-limit aware retry queue
          │
          ▼
Snapshot store → Change detection → Alert router → Dashboard
```

See [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) for the implementation plan.

## Important SP-API note

“Real-time” should be implemented as event-driven monitoring where an Amazon notification type is available, combined with scheduled reconciliation. This prevents changes from being missed and avoids promising instantaneous events for fields that Amazon does not publish as notifications.

## Suggested production stack

- **Backend:** Node.js/TypeScript, NestJS or Fastify
- **Database:** PostgreSQL
- **Queue/events:** Amazon SQS
- **Scheduling:** EventBridge or a worker queue
- **Secrets:** AWS Secrets Manager
- **Frontend:** React/Next.js
- **Alerts:** Amazon SES, Telegram Bot API, Slack webhooks
- **Hosting:** AWS ECS/Fargate, Lambda, or another managed container platform

## Security

Do not commit LWA client secrets, refresh tokens, AWS keys or seller credentials. Production credentials must be encrypted at rest and accessed only by the backend.
