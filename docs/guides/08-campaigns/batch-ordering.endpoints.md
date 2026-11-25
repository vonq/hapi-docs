---
id: campaign-batch-ordering-endpoints
title: Batch Ordering - Endpoint Reference
description: HTTP request/response details for creating, reading and cancelling a batch of campaigns.
category: guides/campaigns
related:
- campaign-batch-ordering
endpoints:
- POST /v3/campaigns/batch/
- GET /v3/campaigns/batch/{batch_id}/
- PUT /v3/campaigns/batch/{batch_id}/cancellation/
---

> For conceptual overview, see [Batch Ordering](./batch-ordering.md).

# Batch Ordering - Endpoint Reference

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| POST | `/v3/campaigns/batch/` | Create up to 20 campaigns together |
| GET | `/v3/campaigns/batch/{batch_id}/` | Read a batch's campaigns and status |
| PUT | `/v3/campaigns/batch/{batch_id}/cancellation/` | Take a batch's campaigns offline |

### POST /v3/campaigns/batch/

- **Auth**: token + `X-Customer-Id` or JWT
- **Description**: Create up to 20 campaigns in one transaction: all of them, or none.

**Query Parameters**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `loose` | boolean | No | `true` applies [loose validation](./validation.md#loose-validation) to every campaign |

**Request Body**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `campaigns` | array | Yes | 1 to 20 campaigns, each the body of a single [`POST /campaigns/order`](./ordering.md) |

Two campaigns ordered together:

```http
POST https://marketplace.api.vonq.com/v3/campaigns/batch/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
Content-Type: application/json
```

```json
{
  "campaigns": [
    {
      "campaignName": "Software Developer - Amsterdam",
      "companyId": "a9b8c7d6-e5f4-3210-ba98-765432109876",
      "orderReference": "PO_567_2019",
      "currency": "EUR",
      "orderedProducts": ["c4d5e6f7-a8b9-0123-def0-234567890123"],
      "orderedProductsSpecs": [],
      "recruiterInfo": {"name": "Jane Doe", "emailAddress": "jane.doe@example.com"},
      "postingDetails": {"title": "Software Developer", "...": "..."},
      "targetGroup": {"...": "..."}
    },
    {
      "campaignName": "Software Developer - Rotterdam",
      "...": "..."
    }
  ]
}
```

**Success Response (201)**

The batch id and the created campaigns' ids, in the order sent:

```json
{
  "batchId": "5f0c2a7e-3b1d-4e8a-9c6f-2d7b8e1a4c3f",
  "campaigns": [
    {"campaignId": "e1f2a3b4-c5d6-7890-abcd-ef1234567890"},
    {"campaignId": "f2a3b4c5-d6e7-8901-bcde-f12345678901"}
  ]
}
```

**Refused Campaigns**

When any campaign is refused, no campaign is created. `campaigns` holds one entry per campaign in the order sent: `{}` for a campaign that passed, and the error body of a single order for a refused one. The status is the highest among the refused campaigns.

```json
{
  "campaigns": [
    {},
    {"companyId": ["This value should not be blank."], "orderedProductsSpecs": [{}]}
  ]
}
```

Error responses may include a structured `errorDetails` array, with every campaign's errors in one list and each `path` prefixed with the campaign's position:

```json
{
  "campaigns": [
    {},
    {"companyId": ["This value should not be blank."], "orderedProductsSpecs": [{}]}
  ],
  "errorDetails": [
    {
      "path": "campaigns.1.companyId",
      "code": "not_blank",
      "message": "This value should not be blank.",
      "params": {}
    }
  ]
}
```

**Errors**

| Status | Cause |
|--------|-------|
| 400 | `campaigns` is missing, empty, longer than 20 or not a list of objects, or a campaign was refused (see [Edge Cases & Gotchas](./batch-ordering.md#edge-cases--gotchas)) |
| 401 | Authentication is missing or invalid |
| 403 | `X-Customer-Id` is required for your account and missing, or a campaign was refused with `403` as its single order would be |

### GET /v3/campaigns/batch/{batch_id}/

- **Auth**: token or JWT
- **Description**: Return a batch's campaigns in the order they were sent, with each campaign's status and the batch's status.

**Path Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| `batch_id` | string | The `batchId` returned when the batch was created |

```http
GET https://marketplace.api.vonq.com/v3/campaigns/batch/5f0c2a7e-3b1d-4e8a-9c6f-2d7b8e1a4c3f/ HTTP/1.1
X-Auth-Token: <your Partner token here>
```

**Success Response (200)**

```json
{
  "batchId": "5f0c2a7e-3b1d-4e8a-9c6f-2d7b8e1a4c3f",
  "status": "online",
  "campaigns": [
    {"campaignId": "e1f2a3b4-c5d6-7890-abcd-ef1234567890", "status": "online"},
    {"campaignId": "f2a3b4c5-d6e7-8901-bcde-f12345678901", "status": "in progress"}
  ]
}
```

**Response Fields**

| Field | Description |
|-------|-------------|
| `status` | The batch's status, derived from its campaigns' (see [Batch Status](./batch-ordering.md#batch-status)) |
| `campaigns[].status` | Each campaign's status, as `GET /campaigns/{campaignId}/status` reports it |

**Notes**

- With the partner token, any of your batches is returned. With a customer JWT, only that customer's batches.

**Errors**

| Status | Cause |
|--------|-------|
| 401 | Authentication is missing or invalid |
| 404 | No batch with this id for your account, or for the customer of the JWT |

### PUT /v3/campaigns/batch/{batch_id}/cancellation/

- **Auth**: token or JWT
- **Description**: Take a batch's campaigns offline, each as a single cancellation would.

**Path Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| `batch_id` | string | The `batchId` returned when the batch was created |

The request has no body:

```http
PUT https://marketplace.api.vonq.com/v3/campaigns/batch/5f0c2a7e-3b1d-4e8a-9c6f-2d7b8e1a4c3f/cancellation/ HTTP/1.1
X-Auth-Token: <your Partner token here>
```

**Success Response (202)**

Each campaign's answer, in the order the campaigns were sent. Here the second campaign was already offline, on an account that asks for an error then:

```json
{
  "batchId": "5f0c2a7e-3b1d-4e8a-9c6f-2d7b8e1a4c3f",
  "campaigns": [
    {"campaignId": "e1f2a3b4-c5d6-7890-abcd-ef1234567890", "result": "accepted"},
    {
      "campaignId": "f2a3b4c5-d6e7-8901-bcde-f12345678901",
      "result": "refused",
      "error": {"campaignId": ["The campaign is not online."]}
    }
  ]
}
```

**Response Fields**

| Field | Description |
|-------|-------------|
| `campaigns[].result` | `accepted` or `refused` |
| `campaigns[].error` | For a refused campaign, the error body `PUT /campaigns/{campaignId}/cancellation` would return |

**Notes**

- `202` means the cancellation was accepted, not that the postings are already removed. Follow the batch's `status` until it is `offline`.
- With the partner token, any of your batches is cancelled. With a customer JWT, only that customer's batches.

**Errors**

| Status | Cause |
|--------|-------|
| 401 | Authentication is missing or invalid |
| 404 | No batch with this id for your account, or for the customer of the JWT |
