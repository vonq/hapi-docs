---
title: Campaign Cancellation - Endpoint Reference
description: HTTP request/response details for campaign and product cancellation endpoints.
category: guides/campaigns
---

> For conceptual overview, see [Campaign Cancellation](./cancellation.md).

# Campaign Cancellation - Endpoint Reference

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| PUT | `/campaigns/{campaignId}/cancellation` | Cancel an entire campaign |
| PUT | `/campaigns/{campaignId}/product_cancellation` | Cancel a single product within a campaign |

### PUT /campaigns/{campaignId}/cancellation

- **Auth**: token + `X-Customer-Id` or JWT
- **Description**: Cancel an entire campaign-all ordered products are taken offline.

**Request Body**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `campaignId` | string | Yes | Must match the path parameter |
| `status` | string | Yes | Must be `"offline"` |

```http
PUT https://marketplace.api.vonq.com/campaigns/e1f2a3b4-c5d6-7890-abcd-ef1234567890/cancellation HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
Content-Type: application/json
```

```json
{
  "campaignId": "e1f2a3b4-c5d6-7890-abcd-ef1234567890",
  "status": "offline"
}
```

**Success Response (202)**

```json
{
  "campaignId": "e1f2a3b4-c5d6-7890-abcd-ef1234567890"
}
```

The response includes a `Location` header pointing to `/campaigns/{campaignId}/status` for polling.

**Errors**

| Status | Cause |
|--------|-------|
| 400 | `campaignId` in body does not match URL, `status` is not `"offline"`, or campaign is not online (see [Cancelling a Non-Online Campaign](./cancellation.md#cancelling-a-non-online-campaign)) |
| 403 | JWT present but `X-Customer-Id` does not match campaign's customer |
| 404 | Campaign not found or does not belong to authenticated partner |

### PUT /campaigns/{campaignId}/product_cancellation

- **Auth**: token + `X-Customer-Id` or JWT
- **Description**: Cancel a single product within a campaign. The campaign remains active with its other products.

**Request Body**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `campaignId` | string | Yes | Must match the path parameter |
| `productId` | string | Yes | The product to cancel-must be in `online` status |
| `status` | string | Yes | Must be `"offline"` |

```http
PUT https://marketplace.api.vonq.com/campaigns/e1f2a3b4-c5d6-7890-abcd-ef1234567890/product_cancellation HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
Content-Type: application/json
```

```json
{
  "campaignId": "e1f2a3b4-c5d6-7890-abcd-ef1234567890",
  "productId": "d6e7f8a9-b0c1-2345-f012-456789012345",
  "status": "offline"
}
```

**Success Response (202)**

```json
{
  "campaignId": "e1f2a3b4-c5d6-7890-abcd-ef1234567890"
}
```

**Errors**

| Status | Cause |
|--------|-------|
| 400 | Product is not in `online` status, or campaign is already `offline` |
| 403 | JWT present but `X-Customer-Id` does not match campaign's customer |
| 404 | Campaign or product not found |
