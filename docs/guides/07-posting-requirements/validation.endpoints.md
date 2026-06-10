---
title: Validation - Endpoint Reference
description: HTTP request/response details for posting requirement validation endpoints.
category: guides/posting-requirements
---

> For conceptual overview, see [Validation](./validation.md).

# Validation - Endpoint Reference

## POST /campaigns/validate-channel-posting/

- **Auth**: token + `X-Customer-Id`, or JWT
- **Description**: Validate posting requirements for a specific product + contract combination. Use this to validate one channel's fields at a time.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `product_id` | string | Yes | The product ID |
| `contract_id` | string (UUID) | Yes | The contract ID |
| `posting_requirements` | array | Yes | List of `{name, value}` objects for each posting requirement |
| `vacancy` | array | No | List of `{name, value}` objects for vacancy fields that certain facets depend on |

<!-- theme: info -->
> **Format note**: Posting requirements here use `{name, value}` objects (array format), which differs from the `postingRequirements` object (key-value format) used in `orderedProductsSpecs` when ordering.

The optional `vacancy` array is needed when facets conditionally require vacancy fields. For example, if a channel's application method is set to "email", then `contactInfo.emailAddress` becomes required. Possible vacancy field names: `applicationUrl`, `contactInfo.emailAddress`, `contactInfo.phoneNumber`, `workingLocation.addressLine1`, `workingLocation.city`, `workingLocation.zipCode`, `title`, `jobPageUrl`.

```http
POST https://marketplace.api.vonq.com/campaigns/validate-channel-posting/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
Content-Type: application/json
```

```json
{
  "product_id": "d4e5f6a7-b8c9-0123-def0-234567890123",
  "contract_id": "c4d5e6f7-a8b9-0123-def0-234567890123",
  "vacancy": [
    { "name": "contactInfo.emailAddress", "value": "hr@acme.com" }
  ],
  "posting_requirements": [
    { "name": "location", "value": "berlin-mitte" },
    { "name": "JobCategory", "value": "software-engineering" },
    { "name": "startDate", "value": "2025-08-10" }
  ]
}
```

### Success Response (`200`)

```json
{
  "errors": {
    "credentials": {},
    "posting_requirements": {}
  },
  "has_errors": false
}
```

### Error Response (`422`)

Errors are split into `credentials` and `posting_requirements`:

```json
{
  "errors": {
    "credentials": {},
    "posting_requirements": {
      "JobCategory": "The field \"Job Category\" must have a value",
      "startDate": "The field \"Start Date\" must have a value"
    }
  },
  "has_errors": true
}
```

**Notes**:
- Credential errors indicate issues with the contract's stored credentials, not the posting requirements themselves. If you see these, the contract may need updating. See [Contracts-Notes](../06-contracts/notes.md).
- Posting requirement error keys match the facet `name`.

### Errors

| Status | Cause |
|--------|-------|
| 422 | Posting requirements or credentials are invalid |
| 401 | Invalid or missing authentication |

## POST /campaigns/validate-questionnaire/

- **Auth**: token + `X-Customer-Id`, or JWT
- **Description**: Validate a questionnaire configuration for Direct Apply channels before including it in a campaign order.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `product_id` | string | Yes | The product ID |
| `contract_id` | string (UUID) | Conditional | Required for products using a contract |
| `facet` | string | Yes | The facet name (typically `"questionnaire"`) |
| `questionnaire` | array | Yes | List of question objects (native JSON array, NOT stringified) |

```http
POST https://marketplace.api.vonq.com/campaigns/validate-questionnaire/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
Content-Type: application/json
```

```json
{
  "product_id": "d4e5f6a7-b8c9-0123-def0-234567890123",
  "contract_id": "c3d4e5f6-a7b8-9012-cdef-123456789012",
  "facet": "questionnaire",
  "questionnaire": [
    {
      "id": "q1",
      "question": "Why are you interested in this role?",
      "type": "text"
    },
    {
      "id": "q2",
      "question": "Preferred start date?",
      "type": "choice",
      "answers": [
        { "id": "a1", "answer": "Immediately" },
        { "id": "a2", "answer": "Within 1 month" }
      ]
    }
  ]
}
```

### Success Response (`200`)

```json
{
  "questionnaire": {},
  "has_errors": false
}
```

### Error Response (`422`)

Error keys are stringified question indices (0-based):

```json
{
  "questionnaire": {
    "1": ["Question too short (5 < 10)"]
  },
  "has_errors": true
}
```

**Notes**:
- Question IDs must be unique within the questionnaire.
- Validation is performed remotely by the channel's validation service-a `503` response means the service is temporarily unavailable. Implement retry with backoff.
- Not all products support questionnaires. Check the product's specs for a facet with `type: "QUESTIONNAIRE"`. See [Direct Apply-Posting Requirements](../10-direct-apply/posting-requirements.md).
