---
title: Bundles in Campaigns - Endpoint Reference
description: JSON request/response examples for ordering bundles and reading bundle campaign responses.
category: guides/campaigns
---

> For conceptual overview, see [Bundles in Campaigns](./bundles.md).

# Bundles in Campaigns - Endpoint Reference

Bundles are ordered via `POST /campaigns/order` and appear in `GET /campaigns/{campaignId}` responses. See [Campaign Ordering - Endpoint Reference](./ordering.endpoints.md) for the base request/response format.

## Ordering a Bundle

Include the bundle product ID in `orderedProducts` like any other product:

```json
{
  "orderedProducts": [
    "bundle-abc-123"
  ]
}
```

You can mix bundles with regular and contract-based (My Contract) products in the same campaign:

```json
{
  "orderedProducts": [
    "bundle-abc-123",
    "regular-product-456",
    "contract-product-789"
  ],
  "orderedProductsSpecs": [
    {
      "productId": "contract-product-789",
      "contractId": "c4d5e6f7-a8b9-0123-def0-234567890123",
      "postingRequirements": { "location": "berlin-mitte" }
    }
  ]
}
```

## Campaign Response After Ordering a Bundle

When you retrieve the campaign (`GET /campaigns/{campaignId}`), the `orderedProducts` array contains **both** the bundle ID and the individual bundled product IDs. If a bundle contains three products, you see four product IDs in total:

```json
{
  "orderedProducts": [
    "bundle-abc-123",
    "bundled-product-1",
    "bundled-product-2",
    "bundled-product-3"
  ],
  "orderedProductsSpecs": [
    {
      "productId": "bundle-abc-123",
      "status": "online"
    },
    {
      "productId": "bundled-product-1",
      "status": "online",
      "jobBoardLink": "https://www.jobboard-a.com/jobs/12345"
    },
    {
      "productId": "bundled-product-2",
      "status": "in progress",
      "jobBoardLink": null
    },
    {
      "productId": "bundled-product-3",
      "status": "online",
      "jobBoardLink": "https://www.jobboard-c.com/vacancy/67890"
    }
  ]
}
```

Each bundled product has its own status, delivery date, and job board link. They transition independently just like any other product in a campaign. There are no bundle-specific fields in the response-the bundled products appear as regular entries in `orderedProductsSpecs` and `postings`.
