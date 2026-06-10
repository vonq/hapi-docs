---
title: Special Products
description: Bundles and CPA+ products.
category: guides/products
endpoints:
  - GET /products/search/
prerequisites: [marketplace]
concepts: [bundle, cpa_plus]
related: [marketplace, cpa, campaign-ordering]
audience: [developer, manager]
difficulty: intermediate
---

# Special Products

> Bundles and CPA+ are specialized product types beyond standard marketplace listings. Both are discovered through the normal product search flow, but they are not safely distinguished by `type` alone.

## Overview

Beyond standard job advertising products, HAPI offers specialized product types and tools to help partners find the best channels and optimize their spending:

- **Bundles**-curated sets of products sold together at a discounted price.
- **CPA+ products**-cost-per-application pricing where you pay for received applications instead of postings.

Both work through the same product search and campaign ordering flow described in [Marketplace](./02-marketplace.md) and [Campaign Ordering](../08-campaigns/ordering.md).

## Reliable Identification Rules

Use the following rules when classifying products:

| Product kind | Reliable check | Important caveats |
|--------------|----------------|-------------------|
| Real bundle | `bundle_products_ids` is a non-empty array and `cpa` is `null` | `type` is often `"product_bundle"`, but that value is not unique to real bundles |
| CPA+ | `cpa != null` | CPA+ products can have `type: "job board"` or `type: "product_bundle"` |

Additional notes:

- There is **no dedicated CPA+ search filter**. Search normally, then inspect `cpa`.
- `channel.type` often mirrors `product.type`, so it is also **not** a reliable discriminator.
- A title containing the word `"Bundle"` is not a reliable classification heuristic.

## Endpoints

| Endpoint | Description |
|----------|-------------|
| `GET /products/search/?isBundle=true` | Filter product search to bundles only |
See [Special Products - Endpoint Reference](./03-special-products.endpoints.md) for full request/response details.

## Bundles

A **bundle** is a curated set of products packaged and sold as a single item at a discounted price. VONQ creates and manages bundles-partners cannot create their own. Bundles are a way to order multiple channels together at a better price than ordering each individually.

### How Bundles Work

- Bundles appear in product search results like any other product. Many live bundle products use `type: "product_bundle"`, but the reliable discriminator is a non-empty `bundle_products_ids` array.
- The `bundle_products_ids` field lists the individual product IDs included in the bundle and can contain a mix of UUID strings and numeric strings.
- You order a bundle by its product ID, just like any other product-pass the bundle's `product_id` in the campaign order.
- After ordering, the campaign details return the individual bundled products alongside the bundle itself.

### Identifying Bundles

Use the `isBundle=true` search parameter to filter for bundles. On the product object:

| Field | What to check |
|-------|---------------|
| `bundle_products_ids` | Non-empty array |
| `cpa` | `null` |
| `type` | Often `"product_bundle"`, but not reliable by itself |

### Ordering a Bundle

Order a bundle the same way you order any product-pass the bundle's `product_id` in `orderedProducts`. The campaign response includes both the bundle and its individual products. If any bundled product has posting requirements, you need to provide them in `orderedProductsSpecs` for each individual product. You can also pass separate UTM parameters for each bundled product via `orderedProductsSpecs`.

When resolving bundled child products later, pass `bundle_products_ids` values through exactly as returned. Do not assume the IDs are all UUIDs.

---

## CPA+ Products

**CPA+** (Cost Per Application Plus) is VONQ's AI-powered, performance-based hiring product. Instead of paying a fixed fee for a job posting, you purchase a package of AI-screened candidate applications-each one pre-screened, interviewed, and scored by AI before delivery.

### Identifying CPA+ Products

CPA+ products have a non-null `cpa` object containing the hiring goal:

| Field | Type | Description |
|-------|------|-------------|
| `cpa.hiring_goal` | integer | Target number of reviewed applications |
| `cpa.hiring_goal_unit` | string | Unit for the hiring goal (currently `reviewed_applications`) |

Standard products have `cpa: null`.

Important classification rules:

- Treat **`cpa != null`** as the reliable CPA+ signal.
- Do **not** rely on `type` or `channel.type`. In live data, `CPA+ Netherlands - Pay for Prescreened Applicants` uses `type: "product_bundle"`, while `CPA+ Powered by LinkedIn - Pay for Reviewed Applicants - v2` uses `type: "job board"`.
- CPA+ products typically have empty `bundle_products_ids`, even when `type` is `"product_bundle"`.
- Because there is no `isCpa` filter, search normally and inspect `cpa` on each result.

### Ordering & Application Management

CPA+ products are ordered through the standard campaign ordering flow. Applications are delivered via [Direct Apply webhooks](../10-direct-apply/webhooks.md) or retrieved through polling endpoints.

For the full CPA+ guide-including endpoints, application retrieval, dossier downloads, and integration workflows-see [CPA+](../09-cpa.md).

---

## Edge Cases & Gotchas

<!-- theme: warning -->
> ### Bundle posting requirements
> When ordering a bundle, you may still need to provide posting requirements for individual products within the bundle. Check each bundled product's `has_product_specs` field and fetch specs for those that need them.

<!-- theme: warning -->
> ### Do not classify by `type`, `channel.type`, or title alone
> A real bundle and a CPA+ product can both use `type: "product_bundle"`. `channel.type` can mirror that same value, and ordinary products can still contain the word `"Bundle"` in the title. Use `bundle_products_ids` and `cpa` instead.

## Related

- [Marketplace](./02-marketplace.md)-product search, details, and the product object reference
- [Posting Requirements](./04-posting-requirements.md)-retrieving specs for products (including bundled products)
- [Campaign Ordering](../08-campaigns/ordering.md)-ordering campaigns with bundles, CPA+, and standard products
- [CPA+](../09-cpa.md)-dedicated CPA+ guide with detailed application management
- [Smartfill](../07-posting-requirements/smartfill.md)-AI-powered autofill for posting requirements and search filters
