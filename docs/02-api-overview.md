---
title: API Overview
description: Base URLs, environments, HTTP conventions, pagination, rate limits, error handling, localization, CORS, and caching.
category: getting-started
endpoints: []
prerequisites: [introduction]
concepts: [pagination, rate_limits, error_handling, localization, cors]
related: [introduction, authentication]
audience: [developer]
difficulty: beginner
---

# API Overview

> Base URLs, environments, HTTP conventions, pagination, rate limits, error handling, and cross-cutting concerns for the VONQ Hiring API (HAPI).

## Base URL & Environments

HAPI provides two environments. Each requires its own API token-credentials from one environment do not work in the other.

| Environment | Base URL | Purpose |
|-------------|----------|---------|
| **Production** | `https://marketplace.api.vonq.com` | Live environment. Orders are processed and charged. |
| **Sandbox** | `https://marketplace-sandbox.api.vonq.com` | Testing environment. Orders are accepted but never executed or charged. |

Use Sandbox for development and integration testing. Contact your VONQ account manager to receive Sandbox credentials.

<!-- theme: info -->
> Some endpoints use a `/v3/` path prefix (e.g., `/v3/smartfill/`, `/v3/screening/`, `/v3/ats/`). This is a routing prefix, not a separate API version-all endpoints are part of the same HAPI v2 API.

## HTTP Methods

| Method | Usage |
|--------|-------|
| `GET` | Retrieve one or more resources |
| `POST` | Create a resource or trigger an action |
| `PUT` | Replace or update a resource |
| `PATCH` | Partially update a resource |
| `DELETE` | Remove a resource |

## Content Type

All request and response bodies use `application/json`. Set the `Content-Type` header on every request that includes a body:

```
Content-Type: application/json
```

## Common Headers

| Header | Required | Description |
|--------|----------|-------------|
| `X-Auth-Token` | Yes (server-to-server) | API token for authentication. See [Authentication](./guides/03-authentication-and-users/authentication.md). |
| `Content-Type` | Yes (on POST/PUT/PATCH) | Must be `application/json` |
| `X-Customer-Id` | Conditional | Identifies the end-customer. Required by contracts, wallets, and some campaign endpoints. See [Entities](./guides/03-authentication-and-users/entities.md). |
| `Accept-Language` | No | Locale for translated responses. See [Localization](#localization). |
| `User-Agent` | Recommended | Identifies your integration (e.g., `YourATS/2.1.0`). Helps VONQ provide more effective support. |

## Pagination

List endpoints paginate automatically using `limit` and `offset` query parameters. The response envelope varies by endpoint - each endpoint's documentation shows its exact pagination format and defaults.

| Parameter | Type | Description |
|-----------|------|-------------|
| `limit` | integer | Number of items per page (default and max vary by endpoint) |
| `offset` | integer | Number of items to skip (default: 0) |

When the response includes `next`/`previous` links, use those to traverse pages rather than calculating offsets manually.

## Sorting & Filtering

Some list endpoints support sorting and filtering via query parameters. The available parameters vary by endpoint-check the relevant documentation page for specifics.

Example-sorting campaigns by creation date:

```
GET /campaigns?sortBy=createdOn.desc
```

## Rate Limits

All endpoints are rate limited to **60 requests per 60 seconds** per partner (per API token). The limit resets on a rolling window.

When you exceed the rate limit, the API returns `429 Too Many Requests`. Back off and retry after the window resets.

## Error Handling

The API uses standard HTTP status codes. Responses in the `2xx` range indicate success, `4xx` indicates a client error, and `5xx` indicates a server-side issue.

| Status Code | Meaning |
|-------------|---------|
| `200 OK` | Request succeeded |
| `201 Created` | Resource created successfully |
| `400 Bad Request` | Invalid JSON or malformed request |
| `401 Unauthorized` | Missing or invalid authentication |
| `403 Forbidden` | Valid credentials but insufficient permissions |
| `404 Not Found` | Resource does not exist |
| `422 Unprocessable Entity` | Validation failed-the request body contains invalid field values |
| `429 Too Many Requests` | Rate limit exceeded |
| `500 Internal Server Error` | Unexpected server error |

Error response bodies vary by endpoint and error type. Here are common formats:

**Authentication error:**

```json
{
  "authentication": "Authentication Required"
}
```

**Malformed JSON:**

```json
{
  "requestBody": "The request does not contain valid JSON."
}
```

**Validation error (field-level):**

```json
{
  "postingDetails.title": "This field is required.",
  "orderedProducts": "You must select at least one product."
}
```

**Validation error (nested):**

```json
{
  "orderedProductsSpecs": [
    {
      "credentials": {},
      "posting_requirements": {
        "location": "This value is required for the selected product."
      }
    }
  ]
}
```

Server errors (`5xx`) are rare. VONQ monitors and investigates these automatically. If you encounter persistent 5xx errors, contact support.

## Localization

HAPI supports translated responses and localized validation messages via the `Accept-Language` header.

Supported values:

| Header value | Language |
|--------------|----------|
| `en` | English |
| `nl` | Dutch |
| `de` | German |
| `fr` | French |
| `es` | Spanish |
| `it` | Italian |

```
Accept-Language: de
```

Translations apply to taxonomy labels, product names and descriptions, error messages, and other user-facing text. If the header is omitted, the API defaults to English. If a translation is unavailable, responses fall back to English or the channel's original language, depending on the endpoint.

<!-- theme: info -->
> Some translation features may need to be enabled by your VONQ account manager. Contact them if you need localized responses for a specific locale.

## Null Serialization

The handling of `null` values varies by endpoint. Some endpoints include `null` fields explicitly, while others omit them entirely. Do not assume that a missing field means "not supported" - it may simply have no value. Treat missing fields and `null` fields equivalently unless the endpoint documentation says otherwise.

## CORS

The API supports Cross-Origin Resource Sharing (CORS) with a fully open policy (`*`). All origins, methods, and headers are permitted. This allows client-side JavaScript applications to call HAPI directly when using [JWT authentication](./guides/03-authentication-and-users/authentication.md).

## Caching

Taxonomy data (job functions, education levels, seniority, industries) and product catalog data are updated regularly. If you cache this data locally, refresh it daily to avoid ordering with stale values-campaigns referencing outdated taxonomy IDs will be rejected.

## Related

- [Introduction](./01-introduction.md)-what is HAPI, getting started
- [Authentication](./guides/03-authentication-and-users/authentication.md)-secret key vs JWT, generating tokens
- [Entities](./guides/03-authentication-and-users/entities.md)-ATS, ATSUser, Company, Partner relationships
