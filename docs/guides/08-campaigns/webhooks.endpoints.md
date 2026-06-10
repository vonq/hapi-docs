---
title: Campaign Webhooks - Endpoint Reference
description: Webhook notification payload structure and endpoint requirements for campaign status webhooks.
category: guides/campaigns
---

> For conceptual overview, see [Campaign Webhooks](./webhooks.md).

# Campaign Webhooks - Endpoint Reference

## Webhook Notification

HAPI sends HTTP POST requests to your configured callback URL when a campaign is updated.

### Notification Payload

```json
{
  "request_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "campaign_id": "e2f3a4b5-c6d7-8901-bcde-f12345678901",
  "modified_on": "2025-03-15T10:30:00+00:00"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `request_id` | string (UUID) | Unique identifier for this notification-use for deduplication |
| `campaign_id` | string (UUID) | The campaign that was modified |
| `modified_on` | string (ISO 8601) | When the campaign was last modified |

The payload is intentionally lightweight-it tells you *which* campaign changed and *when*, but not *what* changed. Fetch the full details with `GET /campaigns/{campaignId}`.

### Your Callback Endpoint Requirements

| Requirement | Detail |
|-------------|--------|
| Method | Must accept HTTP `POST` requests |
| Content type | Must accept `Content-Type: application/json` |
| Response | Must return a `2xx` status code to acknowledge receipt |
| Timeout | Must respond within 30 seconds |
| Idempotency | Must handle duplicate deliveries-use `request_id` to detect duplicates |

### Fetching Campaign Details After Notification

After receiving a notification, fetch the full campaign state:

```http
GET https://marketplace.api.vonq.com/campaigns/{campaignId} HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
```

See [Status & Lifecycle - Endpoint Reference](./status.endpoints.md) for the full response format of `GET /campaigns/{campaignId}`.
