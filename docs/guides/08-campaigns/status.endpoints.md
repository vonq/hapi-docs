---
title: Status & Lifecycle - Endpoint Reference
description: HTTP request/response details for campaign status and listing endpoints.
category: guides/campaigns
---

> For conceptual overview, see [Status & Lifecycle](./status.md).

# Status & Lifecycle - Endpoint Reference

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/campaigns/{campaignId}/status` | Lightweight status check for a single campaign |
| GET | `/campaigns/{campaignId}` | Full campaign details including status, delivery dates, job board links, and metrics |
| GET | `/campaigns` | List campaigns with filtering and pagination |

### GET /campaigns/{campaignId}/status

- **Auth**: token + `X-Customer-Id` or JWT
- **Description**: Lightweight status check-returns campaign and per-product statuses without the full campaign payload.

Use this for polling when you only need to know whether statuses have changed.

```http
GET https://marketplace.api.vonq.com/campaigns/e1f2a3b4-c5d6-7890-abcd-ef1234567890/status HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
```

```json
{
  "data": {
    "campaignId": "e1f2a3b4-c5d6-7890-abcd-ef1234567890",
    "status": "online",
    "orderedProductsStatuses": [
      {
        "productId": "d1e2f3a4-b5c6-7890-abcd-ef1234567890",
        "status": "online",
        "statusDescription": null,
        "statusSolution": null,
        "statusRawError": null
      },
      {
        "productId": "d2e3f4a5-b6c7-8901-bcde-f12345678901",
        "status": "in progress",
        "statusDescription": null,
        "statusSolution": null,
        "statusRawError": null
      }
    ]
  }
}
```

**Response Fields**

| Field | Type | Description |
|-------|------|-------------|
| `data.campaignId` | string | Campaign UUID |
| `data.status` | string | Campaign-level status: `in progress`, `online`, or `offline` |
| `data.orderedProductsStatuses` | array | Per-product status array |
| `data.orderedProductsStatuses[].productId` | string | Product ID |
| `data.orderedProductsStatuses[].status` | string | `in progress`, `online`, `offline`, or `not processed` |
| `data.orderedProductsStatuses[].statusDescription` | string\|null | Human-readable status info |
| `data.orderedProductsStatuses[].statusSolution` | string\|null | Suggested action for the user-may contain HTML, display as raw string (do not render) |
| `data.orderedProductsStatuses[].statusRawError` | string\|null | Raw error from job board-display as raw string (for debugging) |

**Errors**

| Status | Cause |
|--------|-------|
| 404 | Campaign does not exist or does not belong to authenticated partner |

### GET /campaigns/{campaignId}

- **Auth**: token + `X-Customer-Id` or JWT
- **Description**: Full campaign details-vacancy fields, per-product statuses, delivery dates, job board links, click metrics, and more.

```http
GET https://marketplace.api.vonq.com/campaigns/e1f2a3b4-c5d6-7890-abcd-ef1234567890 HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
```

<details>
<summary>Full response example</summary>

```json
{
  "data": {
    "campaignId": "e1f2a3b4-c5d6-7890-abcd-ef1234567890",
    "draftCampaignId": null,
    "campaignName": "Senior Software Developer - Berlin",
    "companyId": "customer-123",
    "status": "online",
    "isEditable": true,
    "createdOn": "2025-01-15T10:00:00+00:00",
    "modifiedOn": "2025-01-15T10:35:00+00:00",
    "totalPrice": { "amount": 49900, "currency": "EUR" },
    "recruiterInfo": {
      "id": "recruiter-42",
      "name": "Jane Recruiter",
      "emailAddress": "jane@acme.example.com"
    },
    "postingDetails": {
      "title": "Senior Software Developer",
      "description": "<p>We are looking for a senior engineer...</p>",
      "organization": { "name": "Acme Corp", "companyLogo": "https://example.com/logo.png" },
      "workingLocation": {
        "addressLine1": "Friedrichstraße 123",
        "postcode": "10117",
        "postCode": "10117",
        "city": "Berlin",
        "country": "DE"
      },
      "employmentType": "permanent",
      "weeklyWorkingHours": { "from": 36, "to": 40 },
      "jobPageUrl": "https://acme.example.com/careers/senior-dev",
      "applicationUrl": "https://acme.example.com/apply/senior-dev"
    },
    "targetGroup": {
      "educationLevel": [{ "vonqId": "2", "description": "Bachelor / Graduate" }],
      "seniority": [{ "vonqId": "3", "description": "Mid-Senior level" }],
      "industry": [{ "vonqId": "48", "description": "Academic" }],
      "jobCategory": [{ "vonqId": "11", "description": "Customer Service" }]
    },
    "orderedProducts": [
      "d1e2f3a4-b5c6-7890-abcd-ef1234567890",
      "d2e3f4a5-b6c7-8901-bcde-f12345678901"
    ],
    "orderedProductsSpecs": [
      {
        "productId": "d1e2f3a4-b5c6-7890-abcd-ef1234567890",
        "status": "online",
        "statusDescription": null,
        "deliveredOn": "2025-01-15T10:30:00+00:00",
        "duration": "30 days",
        "durationPeriod": { "range": "days", "period": 30 },
        "utm": "utm_source=vonq&utm_medium=jobboard&utm_campaign=senior-dev",
        "jobBoardLink": "https://www.linkedin.com/jobs/view/123456",
        "contractId": null,
        "postingRequirements": null,
        "metadata": {}
      },
      {
        "productId": "d2e3f4a5-b6c7-8901-bcde-f12345678901",
        "status": "in progress",
        "statusDescription": null,
        "deliveredOn": null,
        "duration": "10 days",
        "durationPeriod": { "range": "days", "period": 10 },
        "utm": null,
        "jobBoardLink": null,
        "contractId": "c4d5e6f7-a8b9-0123-def0-234567890123",
        "postingRequirements": { "location": "berlin-mitte" },
        "metadata": {}
      }
    ],
    "postings": [
      {
        "name": "LinkedIn Job Posting",
        "clicks": 42,
        "productId": "d1e2f3a4-b5c6-7890-abcd-ef1234567890"
      },
      {
        "name": "My Contract: Adzuna",
        "clicks": 0,
        "productId": "d2e3f4a5-b6c7-8901-bcde-f12345678901"
      }
    ]
  }
}
```

</details>

**Key Status-Related Fields**

| Field | Type | Description |
|-------|------|-------------|
| `data.status` | string | Campaign-level status |
| `data.draftCampaignId` | string\|null | Draft campaign ID if this campaign was created from a draft, otherwise `null` |
| `data.isEditable` | boolean | Whether campaign can be edited-see [Editing](./editing.md) |
| `data.createdOn` | string | Campaign creation timestamp (ISO 8601) |
| `data.modifiedOn` | string | Last modification timestamp-updated on any product status change |
| `data.totalPrice` | object | Total campaign cost: `amount` (integer, in cents) and `currency` |

**Per-Product Fields** (in `orderedProductsSpecs[]`)

| Field | Type | Description |
|-------|------|-------------|
| `status` | string | Product status: `in progress`, `online`, `offline`, `not processed` |
| `statusDescription` | string\|null | Human-readable status info |
| `deliveredOn` | string\|null | ISO 8601 timestamp when posting went live-`null` until `online` |
| `duration` | string | Posting duration as text (e.g., `"30 days"`) |
| `durationPeriod` | object | Structured duration: `range` (`"days"`) and `period` (integer) |
| `jobBoardLink` | string\|null | URL to the live posting-`null` until `online` |
| `utm` | string\|null | UTM tracking parameters if provided during ordering |
| `contractId` | string\|null | Contract ID for JP products, `null` for JM |
| `metadata` | object | Custom metadata for this product spec entry (empty object `{}` when not set) |

**Click Metrics** (in `postings[]`)

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Product display name |
| `clicks` | integer | Number of clicks on the posting |
| `productId` | string | Product ID |

**Errors**

| Status | Cause |
|--------|-------|
| 403 | JWT present but `X-Customer-Id` does not match campaign's customer |
| 404 | Campaign does not exist or does not belong to authenticated partner |

### GET /campaigns

- **Auth**: token + `X-Customer-Id` or JWT
- **Description**: List campaigns with filtering and pagination.

```http
GET https://marketplace.api.vonq.com/campaigns?limit=50&offset=0&sortBy=modifiedOn.desc HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
```

**Query Parameters**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | integer | 50 | Page size |
| `offset` | integer | 0 | Pagination offset |
| `sortBy` | string | `createdOn.desc` | Sort: `modifiedOn.asc`, `modifiedOn.desc`, `createdOn.asc`, `createdOn.desc` |
| `modifiedOn_gte` | datetime | - | Campaigns modified on or after this date-use for incremental sync |
| `modifiedOn_lte` | datetime | - | Campaigns modified on or before this date |
| `createdOn_gte` | datetime | - | Created on or after this date |
| `createdOn_lte` | datetime | - | Created on or before this date |
| `companyId` | string | - | Filter by company ID |
| `label[]` | string[] | - | Filter by labels |

The response includes `total`, `limit`, `offset`, `data` (array of campaign objects), and `meta` with pagination links (`first`, `last`, `next`, `previous`).

Each campaign in the list includes full `orderedProductsSpecs` and `postings` arrays, so you can read per-product statuses without additional API calls.

**Incremental Sync Example**

Use the `modifiedOn_gte` filter to fetch only campaigns that changed since your last sync:

```
GET /campaigns?modifiedOn_gte=2025-01-15T09:00:00Z&sortBy=modifiedOn.asc&limit=50
```
