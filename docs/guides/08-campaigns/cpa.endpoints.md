---
title: CPA+ in Campaigns - Endpoint Reference
description: JSON response examples for CPA+ fields in campaign responses-orderedCpa and cpaResults.
category: guides/campaigns
---

> For conceptual overview, see [CPA+ in Campaigns](./cpa.md).

# CPA+ in Campaigns - Endpoint Reference

CPA+ products are ordered via `POST /campaigns/order` and retrieved via `GET /campaigns/{campaignId}`. See [Campaign Ordering - Endpoint Reference](./ordering.endpoints.md) and [Status & Lifecycle - Endpoint Reference](./status.endpoints.md) for the base request/response formats.

## CPA+ Fields in Campaign Responses

### orderedCpa

In `orderedProductsSpecs[]`, the `orderedCpa` object records the hiring goal purchased:

```json
{
  "orderedProductsSpecs": [
    {
      "productId": "d5e6f7a8-b9c0-1234-ef01-345678901234",
      "status": "online",
      "orderedCpa": {
        "hiring_goal": 20,
        "hiring_goal_unit": "reviewed_applications"
      }
    }
  ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `orderedCpa` | object\|null | `null` for non-CPA+ products |
| `orderedCpa.hiring_goal` | integer | Target number of reviewed applications |
| `orderedCpa.hiring_goal_unit` | string | Always `"reviewed_applications"` |

### cpaResults

In `postings[]`, the `cpaResults` object tracks real-time progress:

```json
{
  "postings": [
    {
      "productId": "d5e6f7a8-b9c0-1234-ef01-345678901234",
      "name": "CPA+ Standard",
      "clicks": 120,
      "cpaResults": {
        "applications": 15,
        "reviewed_applications": 8
      }
    }
  ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `cpaResults` | object\|null | `null` for non-CPA+ products |
| `cpaResults.applications` | integer | Total finalized applications received |
| `cpaResults.reviewed_applications` | integer | Applications that passed AI screening |

Compare `reviewed_applications` against `hiring_goal` to show progress (e.g., "8 of 20 reviewed applications received").
