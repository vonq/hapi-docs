---
title: Campaign Ordering - Endpoint Reference
description: HTTP request/response details for the campaign ordering endpoint.
category: guides/campaigns
---

> For conceptual overview, see [Campaign Ordering](./ordering.md).

# Campaign Ordering - Endpoint Reference

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| POST | `/campaigns/order` | Create a campaign to post a vacancy on one or more job boards |

### POST /campaigns/order

- **Auth**: token + `X-Customer-Id` or JWT
- **Description**: Create a campaign to post a vacancy on one or more job boards.

**Query Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| `validateOnly` | boolean | When `true`, runs full validation without creating the campaign. See [Validation](./validation.md). |
| `loose` | boolean | When `true`, vacancy fields listed in ATS settings may be omitted from the payload. See [Loose Validation](./ordering.md#loose-validation). |

**Request Headers**

| Header | Required | Description |
|--------|----------|-------------|
| `X-Auth-Token` | Yes | Partner API token |
| `X-Customer-Id` | Yes | Customer identifier |
| `Content-Type` | Yes | `application/json` |
| `Accept-Language` | No | Locale for validation error messages. See [Localization](../../02-api-overview.md#localization). |

**Request Body**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `companyId` | string | Yes | Company identifier (same as `X-Customer-Id`) |
| `recruiterInfo` | object | Yes | Recruiter contact-see [Vacancy Fields](./vacancy-fields.md#recruiterinfo) |
| `postingDetails` | object | Yes | Job listing content-see [Vacancy Fields](./vacancy-fields.md#postingdetails) |
| `targetGroup` | object | Yes | Taxonomy classifications-see [Vacancy Fields](./vacancy-fields.md#targetgroup) |
| `orderedProducts` | array | Yes | Product IDs to order (flat array of strings) |
| `orderedProductsSpecs` | array | Conditional | Per-product configuration-required for JP products and for JM products with product-level posting requirements |
| `campaignName` | string | No | Human-readable campaign name |
| `currency` | string | No | Currency code (`EUR`, `USD`, `GBP`, `AUD`). Defaults to `EUR` |
| `orderReference` | string | No | External reference for your own tracking |
| `labels` | object | No | Key-value pairs for filtering and organization |
| `walletId` | string (UUID) | Conditional | Wallet to charge-required when `paymentMethod` is `wallet`, `direct_charge`, or `purchase_order` |
| `poNumber` | string | Conditional | Purchase order number-required when `paymentMethod` is `purchase_order` |
| `paymentMethod` | string | No | Payment method for the campaign. Allowed values: `wallet`, `purchase_order`, `ats_managed`, `direct_charge`. Defaults to `ats_managed`. For `wallet`, `direct_charge`, and `purchase_order`, the order must contain at least one product with a price greater than `0`. See [Payment Methods](./ordering.md#payment-methods). |
| `directApply` | object | No | Override Direct Apply webhook URL for this campaign. See [Direct Apply](../10-direct-apply/01-introduction.md). |

**orderedProductsSpecs Fields**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `productId` | string | Yes | Must match an ID in `orderedProducts` |
| `contractId` | string (UUID) | JP only | Contract to use for this product |
| `postingRequirements` | object | Conditional | Channel-specific field values as key-value pairs. Required when the selected JP contract or JM product has posting requirements. |
| `postingRequirementsLabels` | object | No | Human-readable labels for posting requirement values |
| `utm` | string | No | UTM tracking parameters |
| `postingDurationDays` | integer | No | Override posting duration (JP only) |

**Example-JM Campaign (marketplace products only, no product-level posting requirements)**

No `orderedProductsSpecs` is needed for JM products unless they have product-level posting requirements or you are adding UTM tracking:

```http
POST https://marketplace.api.vonq.com/campaigns/order HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
Content-Type: application/json
```

```json
{
  "companyId": "customer-123",
  "campaignName": "Senior Developer - Berlin",
  "currency": "EUR",
  "recruiterInfo": {
    "id": "recruiter-42",
    "name": "Jane Recruiter",
    "emailAddress": "jane@acme.example.com"
  },
  "postingDetails": {
    "title": "Senior Software Developer",
    "description": "<p>We are looking for a senior engineer to join our platform team...</p>",
    "organization": {
      "name": "Acme Corp",
      "companyLogo": "https://example.com/logo.png"
    },
    "workingLocation": {
      "addressLine1": "Friedrichstraße 123",
      "postcode": "10117",
      "city": "Berlin",
      "country": "DE"
    },
    "contactInfo": {
      "name": "HR Department",
      "emailAddress": "hr@acme.example.com"
    },
    "yearsOfExperience": 5,
    "employmentType": "permanent",
    "weeklyWorkingHours": { "from": 36, "to": 40 },
    "salaryIndication": {
      "period": "yearly",
      "range": { "from": 70000, "to": 90000, "currency": "EUR" }
    },
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
    "d1e2f3a4-b5c6-7890-abcd-ef1234567890"
  ]
}
```

<details>
<summary>Example-Mixed Campaign (JM + JP with posting requirements and UTM)</summary>

```http
POST https://marketplace.api.vonq.com/campaigns/order HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
Content-Type: application/json
```

```json
{
  "companyId": "customer-123",
  "campaignName": "Senior Developer - Berlin",
  "currency": "EUR",
  "recruiterInfo": {
    "id": "recruiter-42",
    "name": "Jane Recruiter",
    "emailAddress": "jane@acme.example.com"
  },
  "postingDetails": {
    "title": "Senior Software Developer",
    "description": "<p>We are looking for a senior engineer...</p>",
    "organization": { "name": "Acme Corp" },
    "workingLocation": {
      "addressLine1": "Friedrichstraße 123",
      "postcode": "10117",
      "city": "Berlin",
      "country": "DE"
    },
    "yearsOfExperience": 5,
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
      "utm": "utm_source=vonq&utm_medium=jobboard&utm_campaign=senior-dev"
    },
    {
      "productId": "d2e3f4a5-b6c7-8901-bcde-f12345678901",
      "contractId": "c4d5e6f7-a8b9-0123-def0-234567890123",
      "postingRequirements": {
        "location": "berlin-mitte",
        "JobCategory": "software-engineering",
        "startDate": "2025-08-10"
      },
      "postingRequirementsLabels": {
        "location": "Berlin Mitte",
        "JobCategory": "Software Engineering"
      },
      "utm": "utm_source=vonq&utm_medium=contract&utm_campaign=senior-dev",
      "postingDurationDays": 30
    }
  ],
  "labels": {
    "department": "engineering",
    "region": "EMEA"
  }
}
```

</details>

**Success Response (201)**

```json
{
  "campaignId": "e1f2a3b4-c5d6-7890-abcd-ef1234567890"
}
```

The response contains only the campaign ID. To get full campaign details (status, per-product statuses, delivery dates), call `GET /campaigns/{campaignId}`. See [Status & Lifecycle](./status.md).

**Error Response (422)**

Validation errors are returned as a nested object matching the request structure:

```json
{
  "postingDetails": {
    "title": ["This field is required."]
  },
  "orderedProductsSpecs": [
    {
      "credentials": {},
      "posting_requirements": {
        "location": "The field \"Location\" must have a value"
      }
    }
  ]
}
```

The `orderedProductsSpecs` error array is positional-the first object corresponds to the first product in your request. See [Validation](./validation.md) for error format details.

**Errors**

| Status | Cause |
|--------|-------|
| 201 | Campaign created successfully |
| 400 | Invalid request (e.g., loose validation is not enabled for the account) |
| 401 | Invalid or missing authentication |
| 403 | Contract customer group mismatch |
| 422 | Validation errors in vacancy fields or posting requirements |
