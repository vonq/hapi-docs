---
title: Marketplace - Endpoint Reference
description: Full request/response details for the HAPI marketplace product endpoints.
category: guides/products
---

> For conceptual overview, see [Marketplace](./02-marketplace.md).

# Marketplace - Endpoint Reference

## GET /products/search/

- **Auth**: token + `X-Customer-Id`, or JWT
- **Description**: Search and filter the product catalog. Returns a paginated list of products ranked by relevance to the search criteria.

Localized results are available via the `Accept-Language` header. See [Localization](../../02-api-overview.md#localization).

**Query Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `name` | string | Filter by product or channel name (partial match) |
| `includeLocationId` | string | Location ID-includes products for this and nearby locations. Comma-separated for multiple. |
| `exactLocationId` | string | Location ID-strict match only |
| `jobTitleId` | string | Job title ID. Mutually exclusive with `jobFunctionId`-the API returns a `400` error if both are provided. |
| `jobFunctionId` | string | Job function ID. Mutually exclusive with `jobTitleId`-the API returns a `400` error if both are provided. |
| `industryId` | string | Industry ID. Comma-separated for multiple. |
| `durationFrom` | integer | Minimum posting duration in days |
| `durationTo` | integer | Maximum posting duration in days |
| `currency` | string | ISO-4217 currency code for price filtering (e.g., `EUR`, `USD`) |
| `sortBy` | string | Sort order (default: `relevant`). See sort options below. |
| `recommended` | boolean | Only return recommended products |
| `excludeRecommended` | boolean | Exclude recommended products |
| `mcEnabled` | boolean | Filter for products on channels that support contracts |
| `isBundle` | boolean | `true` for bundles only, `false` to exclude bundles |
| `injectFavorites` | boolean | **ATS users only, page 1 only.** When `true` and `offset=0`, prepends the authenticated ATS user's favorite products (most-recently-favorited first). Capped at 10 products. Keep this parameter on later pagination requests so promoted favorites stay excluded from the regular result stream. Ignored for non-ATS users. See [Personalizing page 1](./02-marketplace.md#personalizing-page-1-favorites--top-ordered). |
| `injectTopOrdered` | string | **ATS users only, page 1 only.** When set to `1w`, `1m`, `6m`, or `12m` and `offset=0`, prepends the authenticated ATS user's most-ordered products for that time window (highest count first). Capped at 10 products. Combines with `injectFavorites` (favorites first, deduped). Keep this parameter on later pagination requests so promoted products stay excluded from the regular result stream. Ignored for non-ATS users. |
| `limit` | integer | Results per page (default: 50) |
| `offset` | integer | Pagination offset (default: 0) |

To find taxonomy IDs for the `jobTitleId`, `jobFunctionId`, `industryId`, `includeLocationId`, and `exactLocationId` filters, see [Taxonomy & Locations](../04-taxonomy.md).

**Sort Options**:

| Value | Description |
|-------|-------------|
| `relevant` | Most relevant (default) |
| `recent` | Most recently added |
| `order_frequency.desc` | Most frequently ordered first |
| `order_frequency.asc` | Least frequently ordered first |
| `created.desc` | Newest first |
| `created.asc` | Oldest first |
| `list_price.desc` | Highest price first |
| `list_price.asc` | Lowest price first |

Search for products available in the Netherlands for the Technology industry:

```http
GET https://marketplace.api.vonq.com/products/search/?includeLocationId=5678&industryId=8&limit=10 HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
```

```json
{
  "count": 142,
  "next": "/products/search/?includeLocationId=5678&industryId=8&limit=10&offset=10",
  "previous": null,
  "results": [
    {
      "product_id": "a3e2f5d1-7c4b-4e8a-9f2d-1b3c5e7f9a0b",
      "title": "Indeed Standard Job Posting",
      "description": "Reach millions of job seekers on the world's largest job site.",
      "homepage": "https://www.indeed.com",
      "type": "job board",
      "allow_orders": true,
      "mc_enabled": true,
      "mc_only": false,
      "has_product_specs": true,
      "product_specs": {
        "validation_disabled": false,
        "validation_optional": false
      },
      "duration": { "range": "days", "period": 30 },
      "time_to_process": { "range": "hours", "period": 24 },
      "time_to_setup": { "range": "hours", "period": 24 },
      "vonq_price": [{ "amount": 299.00, "currency": "EUR" }],
      "ratecard_price": [{ "amount": 349.00, "currency": "EUR" }],
      "channel": {
        "id": 100,
        "name": "Indeed",
        "url": "https://www.indeed.com",
        "type": "job board"
      },
      "locations": [
        { "id": 5678, "canonical_name": "Netherlands" }
      ],
      "job_functions": [],
      "industries": [
        { "id": 8, "name": "Technology" }
      ],
      "audience_group": "generic",
      "cross_postings": [],
      "logo_url": [{ "url": "https://static.vonq.com/indeed-logo.png" }],
      "logo_square_url": [{ "url": "https://static.vonq.com/indeed-square.png", "size": "68x68" }],
      "logo_rectangle_url": [{ "url": "https://static.vonq.com/indeed-rect.png", "size": "270x90" }],
      "cpa": null,
      "bundle_products_ids": []
    }
  ]
}
```

**Notes**:
- `includeLocationId` broadens results to nearby locations. Use `exactLocationId` when you need a strict geographic match.
- `jobFunctionId` and `jobTitleId` are mutually exclusive. The API returns a `400` error if both are provided.
- Products with `allow_orders: false` are visible in search but cannot be ordered-filter these out before presenting them as orderable options.
- The response uses standard pagination with `count`, `next`, and `previous` fields.

---

## GET /products/single/{product_id}/

- **Auth**: token + `X-Customer-Id`, or JWT
- **Description**: Retrieve full details for a single product.

**Path Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `product_id` | string | The product identifier (UUID or numeric string) |

```http
GET https://marketplace.api.vonq.com/products/single/a3e2f5d1-7c4b-4e8a-9f2d-1b3c5e7f9a0b/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
```

Returns a single Product object (same structure as search results).

---

## GET /products/multiple/{products_ids_or_portfolio_id}/

- **Auth**: token + `X-Customer-Id`, or JWT
- **Description**: Retrieve product details from a list of product IDs, the authenticated user's favorites, the user's top-ordered products, or a pre-configured portfolio selector.

**Path Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `products_ids_or_portfolio_id` | string | Comma-separated product IDs, `favorites`, `orders-top-{window}`, or a portfolio selector UUID. Maximum 50 product IDs. |

**Query Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `limit` | integer | Results per page |
| `offset` | integer | Pagination offset |

**Selector Values**:

| Value | Description |
|-------|-------------|
| Comma-separated product IDs | Retrieve full product details for up to 50 product IDs. Duplicates are removed while preserving the first occurrence. |
| `favorites` | Retrieve full product details for the authenticated ATS user's favorite products. Manage the favorite IDs through [Product Favorites](./05.favorites.md). |
| `orders-top-1w` | Retrieve products most ordered by the authenticated ATS user in the last week. |
| `orders-top-1m` | Retrieve products most ordered by the authenticated ATS user in the last month. |
| `orders-top-6m` | Retrieve products most ordered by the authenticated ATS user in the last 6 months. |
| `orders-top-12m` | Retrieve products most ordered by the authenticated ATS user in the last 12 months. |
| Portfolio selector UUID | Retrieve products belonging to a pre-configured portfolio selector. |

Retrieve details for two products at once:

```http
GET https://marketplace.api.vonq.com/products/multiple/a3e2f5d1-7c4b-4e8a-9f2d-1b3c5e7f9a0b,b4f3a6e2-8d5c-4f9b-a03e-2c4d6f8a1b2c/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
```

```json
{
  "count": 2,
  "next": null,
  "previous": null,
  "results": [
    {
      "product_id": "a3e2f5d1-7c4b-4e8a-9f2d-1b3c5e7f9a0b",
      "title": "Indeed Standard Job Posting",
      "allow_orders": true,
      "vonq_price": [{ "amount": 299.00, "currency": "EUR" }],
      "ratecard_price": [{ "amount": 349.00, "currency": "EUR" }],
      "duration": { "range": "days", "period": 30 },
      "channel": { "id": 100, "name": "Indeed", "url": "https://www.indeed.com", "type": "job board" }
    },
    {
      "product_id": "b4f3a6e2-8d5c-4f9b-a03e-2c4d6f8a1b2c",
      "title": "LinkedIn Job Slot",
      "allow_orders": true,
      "vonq_price": [{ "amount": 499.00, "currency": "EUR" }],
      "ratecard_price": [{ "amount": 549.00, "currency": "EUR" }],
      "duration": { "range": "days", "period": 30 },
      "channel": { "id": 200, "name": "LinkedIn", "url": "https://www.linkedin.com", "type": "social media" }
    }
  ]
}
```

**Notes**:
- This endpoint may return products with `allow_orders: false`. Filter these out before presenting them as orderable.
- Maximum 50 product IDs per request.
- Pass `products_ids_or_portfolio_id` only in the URL path. Requests that also send `products_ids_or_portfolio_id` as a query parameter are rejected. The legacy typo `product_ids_or_portfolio_id` is rejected the same way.

---

## GET /products/delivery-time/{products_ids}/

- **Auth**: token + `X-Customer-Id`, or JWT
- **Description**: Retrieve combined delivery timing for one or more selected products.

**Path Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `products_ids` | string | Comma-separated list of product IDs |

```http
GET https://marketplace.api.vonq.com/products/delivery-time/a3e2f5d1-7c4b-4e8a-9f2d-1b3c5e7f9a0b,b4f3a6e2-8d5c-4f9b-a03e-2c4d6f8a1b2c/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
```

Use this endpoint when you need the delivery estimate for a basket of products before ordering.

**Response (200)**

The response is an aggregate for the selected products; it is not paginated.

| Field | Type | Description |
|-------|------|-------------|
| `days_to_process` | integer | Maximum processing time across the selected products, rounded up to days |
| `days_to_setup` | integer | Maximum setup time across the selected products, rounded up to days |
| `total_days` | integer | Sum of `days_to_process` and `days_to_setup` |

```json
{
  "days_to_process": 2,
  "days_to_setup": 1,
  "total_days": 3
}
```
