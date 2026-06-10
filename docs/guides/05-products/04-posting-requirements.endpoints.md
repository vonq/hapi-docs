---
title: Product Posting Requirements - Endpoint Reference
description: Full request/response details for product specs and facet autocomplete endpoints.
category: guides/products
---

> For conceptual overview, see [Product Posting Requirements](./04-posting-requirements.md).

# Product Posting Requirements - Endpoint Reference

## GET /products/{product_id}/specs/

- **Auth**: token + `X-Customer-Id`, or JWT
- **Description**: Retrieve the posting requirements for a product. Returns channel information and a list of facets-the fields you need to render and collect values for.

**Path Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `product_id` | string (UUID) | The product identifier |

```http
GET https://marketplace.api.vonq.com/products/a3e2f5d1-7c4b-4e8a-9f2d-1b3c5e7f9a0b/specs/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
```

```json
{
  "id": "100",
  "name": "Indeed",
  "logo_url": "https://example.com/channels/100/rect_indeed.png",
  "logo_square_url": "https://example.com/channels/100/square.png",
  "type": "job board",
  "url": "https://www.indeed.com",
  "posting_requirements": [
    {
      "name": "country_id",
      "label": "Country",
      "sort": "1",
      "required": true,
      "type": "SELECT",
      "options": [
        { "key": "NL", "label": "Netherlands" },
        { "key": "DE", "label": "Germany" },
        { "key": "US", "label": "United States" }
      ],
      "rules": [],
      "message": null,
      "primary_taxonomy": null,
      "autocomplete": null,
      "display_rules": null
    },
    {
      "name": "city",
      "label": "City",
      "sort": "2",
      "required": true,
      "type": "AUTOCOMPLETE",
      "options": [],
      "rules": [{ "rule": "maxlength", "data": "100" }],
      "message": "Start typing to search for a city",
      "primary_taxonomy": "job_work_location_city",
      "autocomplete": {
        "required_parameters": ["term"]
      },
      "display_rules": null
    },
    {
      "name": "job_description",
      "label": "Job Description",
      "sort": "3",
      "required": true,
      "type": "TEXTAREA",
      "options": [],
      "rules": [{ "rule": "maxlength", "data": "5000" }],
      "message": null,
      "primary_taxonomy": "job_description",
      "autocomplete": null,
      "display_rules": null
    }
  ]
}
```

**Response fields**:

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Channel ID |
| `name` | string | Channel name |
| `logo_url` | string | Rectangular channel logo URL |
| `logo_square_url` | string | Square channel logo URL |
| `type` | string | Channel type (e.g., `"job board"`, `"social media"`) |
| `url` | string | Channel website URL |
| `posting_requirements` | array | List of facets (see [Facet Structure](./04-posting-requirements.md#facet-structure)) |

---

## GET/POST /products/{product_id}/specs/facets/{facet_name}/options/

- **Auth**: token + `X-Customer-Id`, or JWT
- **Description**: Fetch autocomplete options for a facet. Use this when the facet has a non-null `autocomplete` object. Call as the user types to provide search-as-you-type suggestions.

**Path Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `product_id` | string (UUID) | Product ID |
| `facet_name` | string | The facet's `name` value |

**Query/Body Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `term` / `search` | string | Search text for autocomplete |
| _(dependent fields)_ | varies | Other facet values that may be needed as context |

Search for city options:

```http
POST https://marketplace.api.vonq.com/products/a3e2f5d1-7c4b-4e8a-9f2d-1b3c5e7f9a0b/specs/facets/city/options/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
Content-Type: application/json

{
  "term": "Amster"
}
```

```json
[
  { "key": "1234", "label": "Amsterdam" },
  { "key": "1235", "label": "Amstetten" }
]
```

**Response fields**:

| Field | Type | Description |
|-------|------|-------------|
| `key` | string | The value to submit for this option |
| `label` | string | Display label |
| `sort` | string | Sort order |
| `parent` | string \| null | Parent option key (for hierarchical facets) |

---

## Example: Submitting Posting Requirements in a Campaign Order

```json
{
  "orderedProducts": ["a3e2f5d1-7c4b-4e8a-9f2d-1b3c5e7f9a0b"],
  "orderedProductsSpecs": [
    {
      "productId": "a3e2f5d1-7c4b-4e8a-9f2d-1b3c5e7f9a0b",
      "postingRequirements": {
        "country_id": "NL",
        "city": "1234",
        "job_description": "We are looking for an experienced Software Engineer..."
      }
    }
  ]
}
```

The keys in `postingRequirements` match the `name` field of each facet. The values depend on the facet type-free text for `TEXT`/`TEXTAREA`, a `key` value for `SELECT`/`AUTOCOMPLETE`, etc.
