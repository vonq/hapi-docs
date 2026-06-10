---
title: Direct Apply-Posting Requirements - Endpoint Reference
description: Full request/response details for the questionnaire validation endpoint and campaign order example.
category: guides/direct-apply
---

> For conceptual overview, see [Direct Apply-Posting Requirements](./posting-requirements.md).

## Endpoints

### POST /campaigns/validate-questionnaire/

- **Auth**: token (`X-Auth-Token` + `X-Customer-Id`)
- **Description**: Validate a questionnaire against a product's constraints before ordering.

This endpoint catches errors early-detailed, per-question validation messages rather than a generic "invalid questionnaire" error during campaign ordering.

**Body Parameters**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `product_id` | string | Yes | Product ID to validate against |
| `contract_id` | string | Conditional | Contract ID for My Contract/Job Post products. Omit for Job Marketing products. |
| `facet` | string | Yes | Questionnaire facet name from the posting requirements response (e.g., `questionnaire`, `customQuestions`) |
| `questionnaire` | array | Yes | Native JSON array of question objects (NOT stringified) |

<!-- theme: warning -->
> ### Native JSON Here
> Unlike campaign ordering, this endpoint accepts the questionnaire as a **native JSON array**, not a stringified string.

Validate a two-question questionnaire for a My Contract/Job Post product:

```http
POST https://marketplace.api.vonq.com/campaigns/validate-questionnaire/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
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
      "question": "Do you have a valid work permit?",
      "type": "choice",
      "answers": [
        { "id": "yes", "answer": "Yes" },
        { "id": "no", "answer": "No" }
      ]
    }
  ]
}
```

For Job Marketing products, use the product ID and omit `contract_id`.

**Success response (200):**

```json
{
  "questionnaire": {},
  "has_errors": false
}
```

**Validation error response (422):**

```json
{
  "questionnaire": {
    "0": ["Question too short (4 < 10)"],
    "1": ["Answer option 'A' too short (1 < 2)"]
  },
  "has_errors": true
}
```

**Response fields**

| Field | Type | Description |
|-------|------|-------------|
| `has_errors` | boolean | Whether validation errors were found |
| `questionnaire` | object | Error details. Keys are **zero-based string indices** of the question that failed (e.g., `"0"`, `"1"`), not question IDs. Values are arrays of error message strings. |

**Errors**

| Status | Cause |
|--------|-------|
| `400` | Invalid `product_id`, `contract_id`, or `facet` name |
| `422` | Questionnaire has validation errors (see `questionnaire` field for details) |
| `503` | Upstream validation service temporarily unavailable |

---

### Example Campaign Order with Direct Apply

Order a My Contract/Job Post campaign with Direct Apply enabled and a questionnaire:

```http
POST https://marketplace.api.vonq.com/campaigns/order HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
Content-Type: application/json
```

```json
{
  "postingDetails": {
    "title": "Senior Software Developer",
    "description": "<p>We are looking for a senior developer...</p>",
    "organization": {
      "name": "Acme Corp"
    }
  },
  "orderedProducts": ["d4e5f6a7-b8c9-0123-def0-234567890123"],
  "orderedProductsSpecs": [
    {
      "productId": "d4e5f6a7-b8c9-0123-def0-234567890123",
      "contractId": "c3d4e5f6-a7b8-9012-cdef-123456789012",
      "postingRequirements": {
        "applicationMethod": "directapply",
        "questionnaire": "[{\"id\":\"q1\",\"question\":\"Why are you interested in this role?\",\"type\":\"text\"},{\"id\":\"q2\",\"question\":\"Do you have a valid work permit?\",\"type\":\"choice\",\"answers\":[{\"id\":\"yes\",\"answer\":\"Yes\"},{\"id\":\"no\",\"answer\":\"No\"}]}]"
      }
    }
  ]
}
```

For Job Marketing products with Direct Apply posting requirements, include an `orderedProductsSpecs` entry with `productId` and `postingRequirements`, but omit `contractId`.
