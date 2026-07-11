# Production Architecture

## 1. Monitoring strategy

Use two complementary paths:

1. **Event-driven path:** subscribe to supported Amazon Notifications API events and receive messages through SQS.
2. **Reconciliation path:** call the relevant SP-API endpoints on a controlled schedule and compare normalized snapshots.

The event path reduces alert latency. The reconciliation path catches unsupported fields, lost events, authorization interruptions and catalogue drift.

## 2. Core services

### Authorization service

- Implements Amazon Seller Central OAuth.
- Stores refresh tokens encrypted in a secret manager.
- Obtains short-lived LWA access tokens.
- Supports multiple sellers and marketplaces.

### SP-API client

- Applies AWS Signature Version 4 where required.
- Centralizes throttling, exponential backoff and retry handling.
- Records request IDs and error details for supportability.
- Normalizes marketplace-specific responses.

### Monitor worker

- Pulls monitored ASIN/SKU records in batches.
- Retrieves current listing attributes and offer state.
- Normalizes arrays, whitespace, prices and image ordering before comparison.
- Writes a snapshot only when a meaningful change occurs.

### Change detector

- Compares canonical JSON snapshots.
- Ignores configured fields and harmless formatting changes.
- Assigns severity according to rules.
- Creates an immutable change event with before/after evidence.

### Alert router

- Routes events by severity, marketplace, ASIN and field.
- Supports email, Telegram, Slack and generic webhooks.
- Retries failed deliveries and records delivery receipts.
- Supports quiet hours, digests and escalation.

## 3. Data model

Recommended primary tables:

- `seller_accounts`
- `marketplaces`
- `monitored_listings`
- `listing_snapshots`
- `change_events`
- `alert_rules`
- `alert_deliveries`
- `api_request_logs`

Every seller-owned row should include a tenant identifier. Change events should be append-only.

## 4. Change categories

- Title
- Bullet points
- Description
- Main and secondary images
- Price and currency
- Inventory quantity
- Listing/offer status
- Fulfilment channel
- Buy Box or featured-offer state, where available
- Listing issues and suppression state

## 5. Operational safeguards

- Idempotency keys for notifications
- Dead-letter queue for failed events
- Per-operation rate-limit budgets
- Circuit breaker during Amazon outages
- Clock-skew handling
- Health checks and structured logs
- Audit trail for user acknowledgements
- Encrypted backups and data-retention rules

## 6. MVP delivery sequence

1. Seller authorization and one marketplace
2. Secure Listings Items API connection
3. Baseline snapshots for a small set of ASINs
4. Scheduled comparison worker
5. Before/after change records
6. Email or Telegram alerts
7. Notifications API/SQS integration where supported
8. Multi-marketplace scaling and production hardening
