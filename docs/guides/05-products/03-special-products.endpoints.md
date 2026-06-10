---
title: Special Products - Endpoint Reference
description: Full request/response details for bundle filtering and CPA+ discovery.
category: guides/products
---

> For conceptual overview, see [Special Products](./03-special-products.md).

# Special Products - Endpoint Reference

## GET /products/search/ (bundle and CPA+ discovery)

Use the standard product search endpoint to discover both real bundles and CPA+ products.

- **Auth**: token + `X-Customer-Id`, or JWT
- There is **no dedicated `isCpa` query parameter**
- Reliable identification rules:
  - real bundle -> `bundle_products_ids` is non-empty
  - CPA+ -> `cpa != null`
  - `type` and `channel.type` are not sufficient on their own

### Bundle filter

Use `isBundle=true` to return actual bundles only.

```http
GET https://marketplace.api.vonq.com/products/search/?isBundle=true HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
```

See [Marketplace - Endpoint Reference](./02-marketplace.endpoints.md) for the full query parameter list.

Important behavior:

- `isBundle=true` filters for **actual bundled products**
- it does **not** return every product whose `type` happens to be `"product_bundle"`
- CPA+ products with `type: "product_bundle"` and empty `bundle_products_ids` are excluded by this filter

**Live bundle example**:

```json
{
  "product_id": "d8e9f0a1-b2c3-4567-1234-678901234567",
  "title": "Product Bundles - Standard Netherlands (49% discount) - Nationale Vacaturebank, Monsterboard, NederlandVacature - Bundle",
  "type": "product_bundle",
  "cpa": null,
  "allow_orders": true,
  "vonq_price": [{ "amount": 1010.0, "currency": "EUR" }],
  "ratecard_price": [{ "amount": 1722.22, "currency": "EUR" }],
  "duration": { "range": "days", "period": 30 },
  "bundle_products_ids": [
    "d9e0f1a2-b3c4-5678-2345-789012345678",
    "386",
    "dae1f2b3-c4d5-6789-3456-890123456789",
    "2121"
  ]
}
```

Note that `bundle_products_ids` can contain both UUID strings and numeric strings. Use the values exactly as returned when calling single-product or multi-product lookup endpoints.

### Excluding real bundles

`isBundle=false` is also supported as an exclusion mode:

```http
GET https://marketplace.api.vonq.com/products/search/?name=cpa&isBundle=false HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
```

This excludes real bundles, but it can still return CPA+ products whose `type` is `"product_bundle"`. In other words, `isBundle` behaves like a real-bundle filter, not a raw `type == "product_bundle"` filter.

### CPA+ discovery

Because there is no `isCpa` filter, discover CPA+ products by searching normally and checking whether `cpa` is non-null.

```http
GET https://marketplace.api.vonq.com/products/search/?name=cpa HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
```

**Live CPA+ results example**:

```json
[
  {
    "product_id": "dbe2f3a4-c5d6-7890-4567-901234567890",
    "title": "CPA+ Netherlands - Pay for Prescreened Applicants",
    "type": "product_bundle",
    "channel": {
      "name": "CPA+ Netherlands",
      "type": "product_bundle"
    },
    "cpa": {
      "hiring_goal": 15,
      "hiring_goal_unit": "reviewed_applications"
    },
    "bundle_products_ids": []
  },
  {
    "product_id": "dce3f4a5-c6d7-8901-5678-012345678901",
    "title": "CPA+ Powered by LinkedIn - Pay for Reviewed Applicants - v2",
    "type": "job board",
    "channel": {
      "name": "CPA+ Powered by LinkedIn",
      "type": "job board"
    },
    "cpa": {
      "hiring_goal": 2,
      "hiring_goal_unit": "reviewed_applications"
    },
    "bundle_products_ids": []
  }
]
```

**Ordering a bundle** - pass the bundle's `product_id` in `orderedProducts`:

```json
{
  "orderedProducts": ["d8e9f0a1-b2c3-4567-1234-678901234567"],
  "orderedProductsSpecs": []
}
```
