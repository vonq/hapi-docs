---
title: Campaign Smartfill - Endpoint Reference
description: HTTP request/response details for the Smartfill vacancy fields and product search filter endpoints.
category: guides/campaigns
---

> For conceptual overview, see [Campaign Smartfill](./smartfill.md).

# Campaign Smartfill - Endpoint Reference

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| POST | `/v3/smartfill/vacancy-fields/` | Create a Smartfill task for vacancy fields |
| GET | `/v3/smartfill/vacancy-fields/{task_id}/` | Retrieve status and results of a vacancy fields Smartfill task |
| POST | `/v3/smartfill/product-search-filters/` | Create a Smartfill task for product search filters |
| GET | `/v3/smartfill/product-search-filters/{task_id}/` | Retrieve status and results of a product search filters Smartfill task |

### POST /v3/smartfill/vacancy-fields/

- **Auth**: token + `X-Customer-Id`
- **Description**: Create a Smartfill task for vacancy fields.

**Request Body**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `context` | object | Yes | Job context-provide `structured`, `unstructured`, or both |
| `context.structured` | object | No | Key-value pairs (e.g., `jobTitle`, `taxonomies`). Max 20,000 characters total. |
| `context.unstructured` | string | No | Free text (e.g., full job description). Min 32 chars, max 20,000 chars. |

At least one of `structured` or `unstructured` must be provided. Both are recommended for best results.

```http
POST https://marketplace.api.vonq.com/v3/smartfill/vacancy-fields/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
Content-Type: application/json
```

```json
{
  "context": {
    "structured": {
      "jobTitle": "Sales Engineer",
      "taxonomies": {
        "jobFunctions": ["Sales", "Engineering"],
        "jobType": "permanent"
      }
    },
    "unstructured": "We are looking for a Sales Engineer to join our Rotterdam office. The role involves technical presales, customer demos, and working with the engineering team. Requirements: 3+ years experience, strong communication skills. We offer competitive salary, 32-40 hours/week."
  }
}
```

**Response (201)**

```json
{
  "smartfill_task_id": "b1c2d3e4-f5a6-7890-abcd-ef1234567890"
}
```

Use this `smartfill_task_id` value as the `{task_id}` path parameter when polling the GET endpoint.

### GET /v3/smartfill/vacancy-fields/{task_id}/

- **Auth**: token + `X-Customer-Id`
- **Description**: Retrieve the status and results of a vacancy fields Smartfill task.

```http
GET https://marketplace.api.vonq.com/v3/smartfill/vacancy-fields/b1c2d3e4-f5a6-7890-abcd-ef1234567890 HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
```

**Response (200)**

```json
{
  "id": "b1c2d3e4-f5a6-7890-abcd-ef1234567890",
  "created_at": "2025-08-10T14:15:22Z",
  "updated_at": "2025-08-10T14:15:28Z",
  "status": "completed",
  "prefill_data": {
    "postingDetails": {
      "contactInfo": {
        "name": "VONQ Recruitment Team"
      },
      "workingLocation": {
        "addressLine1": "Rotterdam, NL"
      }
    },
    "recruiterInfo": {
      "name": "VONQ Recruitment Team"
    },
    "taxonomy": {
      "industry": {
        "id": 48,
        "name": "Aviation / Space travel"
      },
      "seniority": {
        "id": 1,
        "name": [
          { "languageCode": "en_GB", "value": "Executive/Director" },
          { "languageCode": "de_DE", "value": "Geschäftsführer / C-Level" },
          { "languageCode": "nl_NL", "value": "Executive/Director" }
        ]
      },
      "jobFunction": {
        "id": 37,
        "name": "IT - Software Development"
      },
      "educationLevel": {
        "id": 1,
        "name": [
          { "languageCode": "en_GB", "value": "Master / Post-Graduate / PhD" },
          { "languageCode": "de_DE", "value": "Master / PhD / Doktor" },
          { "languageCode": "nl_NL", "value": "Master / Postdoctoraal" }
        ]
      }
    }
  }
}
```

**Response Fields**

| Field | Type | Description |
|-------|------|-------------|
| `id` | string (UUID) | Task identifier |
| `created_at` | string (ISO 8601) | When the task was created |
| `updated_at` | string (ISO 8601) | When results were last updated-check for changes between polls |
| `status` | string | `queued`, `started`, `completed`, or `errored` |
| `prefill_data` | object | Suggested values-grows incrementally until `completed` |

**Prefill Data Structure**

| Section | Fields | Maps To |
|---------|--------|---------|
| `prefill_data.postingDetails` | `contactInfo`, `workingLocation`, and other posting detail fields | `postingDetails` in campaign order |
| `prefill_data.recruiterInfo` | `name`, `emailAddress` | `recruiterInfo` in campaign order |
| `prefill_data.taxonomy` | `industry`, `seniority`, `jobFunction`, `educationLevel` | `targetGroup` in campaign order |

Taxonomy suggestions include `id` and `name`. Some taxonomy values return multilingual names (array of `{ languageCode, value }` objects)-select the translation matching your user's locale.

**Task Statuses**

| Status | Meaning |
|--------|---------|
| `queued` | Task is waiting to be processed |
| `started` | AI is generating suggestions-partial results may be available in `prefill_data` |
| `completed` | All suggestions generated |
| `errored` | Task failed-`prefill_data` may contain partial results |

### POST /v3/smartfill/product-search-filters/

Create a Smartfill task for product search filters:

```http
POST https://marketplace.api.vonq.com/v3/smartfill/product-search-filters/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
Content-Type: application/json
```

```json
{
  "context": {
    "structured": {
      "jobTitle": "Software Engineer",
      "taxonomies": {
        "jobFunctions": ["Engineering"],
        "jobType": "permanent"
      }
    },
    "unstructured": "Looking for software engineers with experience in JavaScript, located in Paris, France, for the automotive industry."
  }
}
```

**Response (201)**

```json
{
  "smartfill_task_id": "b1c2d3e4-f5a6-7890-abcd-ef1234567890"
}
```

Use this `smartfill_task_id` value as the `{task_id}` path parameter when polling the GET endpoint.

### GET /v3/smartfill/product-search-filters/{task_id}/

Poll for product search filter results:

```json
{
  "id": "b1c2d3e4-f5a6-7890-abcd-ef1234567890",
  "status": "completed",
  "prefill_data": {
    "job_title": {
      "id": 115,
      "name": "JavaScript Developer",
      "job_function": { "id": 6, "name": "Software Development" }
    },
    "job_function": {
      "id": 6,
      "name": "Software Development"
    },
    "location": {
      "id": 18163,
      "fully_qualified_place_name": "Paris, France",
      "canonical_name": "Paris"
    },
    "industry": {
      "id": 22,
      "name": "Automotive"
    }
  }
}
```

Use these suggestions to pre-populate the product search form. The `id` values can be passed directly to the product search endpoint filters.
