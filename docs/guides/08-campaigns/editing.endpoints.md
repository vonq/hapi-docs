---
title: Campaign Editing - Endpoint Reference
description: HTTP request/response details for the campaign editing endpoint.
category: guides/campaigns
---

> For conceptual overview, see [Campaign Editing](./editing.md).

# Campaign Editing - Endpoint Reference

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| PUT | `/campaigns/{campaignId}/edit` | Update a live campaign's vacancy fields and posting requirements |

### PUT /campaigns/{campaignId}/edit

- **Auth**: token + `X-Customer-Id` or JWT
- **Description**: Update a live campaign's vacancy fields and posting requirements.

**Query Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| `loose` | boolean | When `true`, vacancy fields listed in `settings.campaigns.loose_validation` may be omitted from the edit payload-mirrors the ordering endpoint. Requires the account to be enabled for loose validation, otherwise returns `400`. See [Loose Validation](./editing.md#loose-validation). |

<!-- theme: danger -->
> ### Full Payload Required (PUT, Not PATCH)
> This endpoint requires the **complete campaign payload**-not just the fields you want to change. Send all required fields, with non-editable fields matching their original values. Build the edit payload by fetching the current campaign with `GET /campaigns/{campaignId}`, modifying the editable fields, and stripping read-only fields before submitting.

**Building the Edit Payload**

1. Fetch the current campaign: `GET /campaigns/{campaignId}`
2. Remove read-only fields from the response:
   - Campaign level: `campaignId`, `status`, `createdOn`, `modifiedOn`, `totalPrice`, `postings`, `isEditable`
   - Per product: `status`, `statusDescription`, `deliveredOn`, `duration`, `durationPeriod`, `jobBoardLink`
3. Modify the editable fields you want to change
4. Keep all non-editable fields at their original values
5. Submit the payload

**Request Example**

```http
PUT https://marketplace.api.vonq.com/campaigns/e1f2a3b4-c5d6-7890-abcd-ef1234567890/edit HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
Content-Type: application/json
```

```json
{
  "companyId": "customer-123",
  "campaignName": "Senior Data Engineer - Berlin",
  "recruiterInfo": {
    "id": "recruiter-42",
    "name": "Jane Recruiter",
    "emailAddress": "jane@acme.example.com"
  },
  "postingDetails": {
    "title": "Senior Data Engineer",
    "description": "<p>Updated: We are looking for a senior data engineer...</p>",
    "organization": {
      "name": "Acme Corp",
      "companyLogo": "https://example.com/logo.png"
    },
    "workingLocation": {
      "addressLine1": "Friedrichstraße 123",
      "postcode": "10117",
      "city": "Berlin",
      "country": "DE",
      "allowsRemoteWork": true
    },
    "contactInfo": {
      "name": "HR Department",
      "emailAddress": "hr@acme.example.com"
    },
    "yearsOfExperience": 3,
    "employmentType": "permanent",
    "weeklyWorkingHours": { "from": 36, "to": 40 },
    "salaryIndication": {
      "period": "yearly",
      "range": { "from": 80000, "to": 100000, "currency": "EUR" }
    },
    "jobPageUrl": "https://acme.example.com/careers/data-engineer",
    "applicationUrl": "https://acme.example.com/apply/data-engineer"
  },
  "targetGroup": {
    "educationLevel": [{ "vonqId": "2", "description": "Bachelor / Graduate" }],
    "seniority": [{ "vonqId": "5", "description": "Senior level" }],
    "industry": [{ "vonqId": "15", "description": "Technology" }],
    "jobCategory": [{ "vonqId": "46", "description": "Data Engineering" }]
  },
  "orderedProducts": [
    "d2e3f4a5-b6c7-8901-bcde-f12345678901"
  ],
  "orderedProductsSpecs": [
    {
      "productId": "d2e3f4a5-b6c7-8901-bcde-f12345678901",
      "contractId": "c4d5e6f7-a8b9-0123-def0-234567890123",
      "utm": "utm_campaign=acme_q1",
      "postingRequirements": {
        "location": "Berlin Mitte",
        "category": "IT / Data Engineering"
      }
    }
  ]
}
```

**Success Response (202)**

No response body. The `202 Accepted` status confirms the edit has been accepted for processing and will be pushed to the job boards.

**Error Responses**

Non-editable field changed:
```json
{
  "companyId": ["companyId cannot be updated"]
}
```

Campaign not editable:
```json
{
  "non_field_errors": ["The specified campaign is not editable."]
}
```

Product list mismatch:
```json
{
  "orderedProducts": [
    "Product IDs passed in request do not match with the product IDs in the campaign"
  ]
}
```

Loose validation requested but not enabled (`?loose=true`):
```json
{
  "detail": "Loose validation is not enabled for this partner"
}
```

**Errors**

| Status | Cause |
|--------|-------|
| 202 | Edit accepted for processing |
| 400 | Validation failed-non-editable field changed, missing required fields, invalid values, or `?loose=true` used when the account is not enabled for loose validation |
| 404 | Campaign not found or `X-Customer-Id` does not match |
