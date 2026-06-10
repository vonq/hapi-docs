---
title: Ordering with Contracts - Endpoint Reference
description: Full request/response examples for ordering campaigns with My Contract products.
category: guides/contracts
---

> For conceptual overview, see [Ordering with Contracts](./ordering.md).

# Ordering with Contracts - Endpoint Reference

## Retrieving Contract Posting Requirements

```http
GET https://marketplace.api.vonq.com/contracts/single/c1d2e3f4-a5b6-7890-abcd-ef1234567890/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
```

The response includes a `posting_requirements` array with the facet definitions for the contract's channel. See [Contract Posting Requirements](./posting-requirements.md) for details on filling these fields.

---

## Campaign Order with a My Contract Product

The `orderedProductsSpecs` array of a campaign order links each My Contract product to its contract and posting requirement values:

```json
{
  "orderedProducts": [
    "d3e4f5a6-b7c8-9012-cdef-123456789012",
    "a3e2f5d1-7c4b-4e8a-9f2d-1b3c5e7f9a0b"
  ],
  "orderedProductsSpecs": [
    {
      "productId": "d3e4f5a6-b7c8-9012-cdef-123456789012",
      "contractId": "c1d2e3f4-a5b6-7890-abcd-ef1234567890",
      "postingRequirements": {
        "location": "sydney-cbd",
        "JobCategory": "information-communication-technology"
      },
      "utm": "utm_source=vonq&utm_medium=cpc"
    }
  ]
}
```

---

## Mixing My Contract and Job Marketing Products

A single campaign can include both My Contract and Job Marketing products. My Contract entries carry a `contractId`; Job Marketing entries do not:

```json
{
  "orderedProducts": [
    "d3e4f5a6-b7c8-9012-cdef-123456789012",
    "a3e2f5d1-7c4b-4e8a-9f2d-1b3c5e7f9a0b"
  ],
  "orderedProductsSpecs": [
    {
      "productId": "d3e4f5a6-b7c8-9012-cdef-123456789012",
      "contractId": "c1d2e3f4-a5b6-7890-abcd-ef1234567890",
      "postingRequirements": {
        "location": "sydney-cbd",
        "JobCategory": "information-communication-technology"
      }
    },
    {
      "productId": "a3e2f5d1-7c4b-4e8a-9f2d-1b3c5e7f9a0b",
      "postingRequirements": {
        "country_id": "NL",
        "city": "1234"
      }
    }
  ]
}
```

In this example:
- The first product is a My Contract product (has `contractId`)
- The second is a Job Marketing product with product-level posting requirements (no `contractId`, but still has `postingRequirements`)
