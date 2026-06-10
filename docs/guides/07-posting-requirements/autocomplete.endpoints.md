---
title: Autocomplete & Lazy-Loaded Options - Endpoint Reference
description: HTTP request/response details for autocomplete endpoints-product options, product data, and contract autocomplete.
category: guides/posting-requirements
---

> For conceptual overview, see [Autocomplete & Lazy-Loaded Options](./autocomplete.md).

# Autocomplete & Lazy-Loaded Options - Endpoint Reference

## POST /products/{product_id}/specs/facets/{facet_name}/options/

- **Auth**: token
- **Description**: Fetch filtered/searched autocomplete options for a product facet.

Use this for search-driven autocomplete on Job Marketing products. The user types a search term, and you call this endpoint to get matching options.

```http
POST https://marketplace.api.vonq.com/products/d3e4f5a6-b7c8-9012-cdef-123456789012/specs/facets/location/options/ HTTP/1.1
X-Auth-Token: <your Partner token here>
Content-Type: application/json
```

```json
{
  "term": "Amsterdam"
}
```

**Response**:

```json
[
  { "key": "1234", "label": "Amsterdam" },
  { "key": "1235", "label": "Amstetten" },
  { "key": "1236", "label": "Amsterdam-Zuidoost" }
]
```

Also supports `GET` with `?search=Amsterdam` query parameter.

## POST /contracts/posting-requirements/{channel_id_or_contract_id}/{posting-requirement-name}/

- **Auth**: token
- **Description**: Fetch autocomplete options for a contract posting requirement facet.

Covered in detail in [Contract Posting Requirements](../06-contracts/posting-requirements.md). Key difference from product endpoints: supports both `channel_id` (requires `credentials` in body) and `contract_id` (credentials already stored).

## Campaign Editing Context

When editing an existing campaign, pass `campaignId` in the autocomplete request body:

```json
{
  "term": "Sydney",
  "campaignId": "c1d2e3f4-a5b6-7890-abcd-ef1234567890"
}
```

This tells the channel that you're editing an existing job, not creating a new one. Some channels return different options for edits-for example, restricting ad type changes or offering upgrade-only options.

Use this whenever you're fetching autocomplete options in the context of a campaign edit workflow.

## Multi-Term Request Format

Some facets require multiple values instead of a single search string. Pass `term` as an array:

```json
{
  "term": ["Fullstack Engineer", "London", "Engineering"]
}
```

Each element provides different context for the channel's option selection logic. The exact meaning of each position depends on the channel implementation.

Known multi-term facets:
- `seekAnzAdvertisementType` (SEEK)-ad type depends on job title, location, and category
- `brandingId`-may depend on multiple context values
