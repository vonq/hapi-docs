---
title: Smartfill - Endpoint Reference
description: HTTP request/response details for the Smartfill posting requirements endpoints-create task and poll for results.
category: guides/posting-requirements
---

> For conceptual overview, see [Smartfill](./smartfill.md).

# Smartfill - Endpoint Reference

## POST /v3/smartfill/posting-requirements/

- **Auth**: token (+ `X-Customer-Id` if applicable)
- **Description**: Create a smartfill task for a specific product or contract. Returns a task ID for polling.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `contract_id` | string (UUID) | Conditional | The contract to fill posting requirements for. Mutually exclusive with `product_id`. |
| `product_id` | string | Conditional | The product to fill posting requirements for. Mutually exclusive with `contract_id`. |
| `context` | object | Yes | Vacancy context for AI processing (see [Context Object](./smartfill.md#context-object)) |

Provide exactly one of `contract_id` or `product_id`-sending both or neither returns `400`.

When using `product_id`, Smartfill uses the product's posting requirements. If that product has no marketplace or validation contract configured, Smartfill still runs in best-effort mode, but credential-dependent autocomplete facets may be skipped.

```http
POST https://marketplace.api.vonq.com/v3/smartfill/posting-requirements/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
Content-Type: application/json
```

```json
{
  "contract_id": "c4d5e6f7-a8b9-0123-def0-234567890123",
  "context": {
    "structured": {
      "title": "Senior Frontend Developer",
      "description": "We are looking for a Senior Frontend Developer with React experience...",
      "skills": ["React", "TypeScript", "CSS"],
      "location": "Amsterdam, Netherlands",
      "company": "ACME Corp",
      "employmentType": "permanent"
    },
    "unstructured": "Senior Frontend Developer position at ACME Corp in Amsterdam. 5+ years React/TypeScript experience required. Permanent, full-time role with hybrid work options."
  }
}
```

### Response (`201`)

```json
{
  "smartfill_task_id": "3f2b1a4c-8d7e-4f6a-9b0c-1d2e3f4a5b6c"
}
```

### Error Responses

| Status | Cause |
|--------|-------|
| `400` | Both `contract_id` and `product_id` provided, or neither; contract missing channel/facets; product not found or doesn't support smartfill; context validation failed |
| `403` | Smartfill not enabled for this account |

## GET /v3/smartfill/posting-requirements/{task_id}/

- **Auth**: token (+ `X-Customer-Id` if applicable)
- **Description**: Poll the status and results of a smartfill task.

```http
GET https://marketplace.api.vonq.com/v3/smartfill/posting-requirements/3f2b1a4c-8d7e-4f6a-9b0c-1d2e3f4a5b6c/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
```

### Response (`200`) - Task in Progress

```json
{
  "id": "3f2b1a4c-8d7e-4f6a-9b0c-1d2e3f4a5b6c",
  "contract_id": "c4d5e6f7-a8b9-0123-def0-234567890123",
  "product_id": null,
  "channel_id": "123",
  "status": "started",
  "created_at": "2025-07-15T10:30:00Z",
  "updated_at": "2025-07-15T10:30:05Z",
  "prefill_data": {
    "location": { "key": "amsterdam", "label": "Amsterdam" },
    "employment_type": "fulltime"
  }
}
```

### Response (`200`) - Task Completed

```json
{
  "id": "3f2b1a4c-8d7e-4f6a-9b0c-1d2e3f4a5b6c",
  "contract_id": "c4d5e6f7-a8b9-0123-def0-234567890123",
  "product_id": null,
  "channel_id": "123",
  "status": "completed",
  "created_at": "2025-07-15T10:30:00Z",
  "updated_at": "2025-07-15T10:30:12Z",
  "prefill_data": {
    "location": { "key": "amsterdam", "label": "Amsterdam" },
    "employment_type": "fulltime",
    "JobCategory": { "key": "software-development", "label": "Software Development" },
    "startDate": "2025-08-01",
    "SearchSummary": "Senior Frontend Developer with React and TypeScript expertise"
  }
}
```

### Task Statuses

| Status | Description |
|--------|-------------|
| `queued` | Task received, waiting to process |
| `started` | AI is processing the context |
| `completed` | All suggestions ready |
| `errored` | Processing failed-fall back to manual input |

### Error Responses

| Status | Cause |
|--------|-------|
| `404` | Task not found |
