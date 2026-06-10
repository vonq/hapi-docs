---
title: Contract Posting Requirements - Endpoint Reference
description: Full request/response details for the contract posting requirements autocomplete endpoint.
category: guides/contracts
---

> For conceptual overview, see [Contract Posting Requirements](./posting-requirements.md).

# Contract Posting Requirements - Endpoint Reference

## POST /contracts/posting-requirements/{channel_id_or_contract_id}/{posting-requirement-name}/

- **Auth**: token
- **Description**: Search for autocomplete options for a contract posting requirement facet.

This endpoint is used for facets that have a non-null `autocomplete` object - typically `AUTOCOMPLETE` type facets but also some `SELECT` and `HIER` facets with lazy-loaded options.

**Path Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| `channel_id_or_contract_id` | string | A numeric channel ID (before contract exists) or a contract UUID (after contract is created) |
| `posting-requirement-name` | string | The facet `name` - case-sensitive (e.g., `location`, `JobCategory`) |

**Request Body**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `term` | string or string[] | Conditional | Search text. Pass as array for multi-term facets. Can be omitted (send empty `{}` body) for facets with a small fixed option set - see note below |
| `credentials` | object | Conditional | Required when using a **channel_id**. Omit when using a **contract_id** (credentials are already stored) |
| `campaignId` | string | No | Pass during campaign editing to get context-aware options |

<!-- theme: info -->
> **Listing all options without a search term.** Some facets with a small, fixed set of options (e.g., `applicationMethod`) accept an empty request body `{}` and return all available options. Most facets still require a non-empty `term` - if you get a `400` response, include a `term`.

---

### Using a contract ID (credentials already stored)

```http
POST https://marketplace.api.vonq.com/contracts/posting-requirements/c1d2e3f4-a5b6-7890-abcd-ef1234567890/location/ HTTP/1.1
X-Auth-Token: <your Partner token here>
Content-Type: application/json
```

```json
{
  "term": "Sydney"
}
```

---

### Using a channel ID (credentials not yet stored - e.g., during contract setup)

```http
POST https://marketplace.api.vonq.com/contracts/posting-requirements/1234/location/ HTTP/1.1
X-Auth-Token: <your Partner token here>
Content-Type: application/json
```

```json
{
  "term": "Sydney",
  "credentials": {
    "agency_or_company": "Company",
    "organization_id": "org-12345"
  }
}
```

---

### Response

Array of matching options:

```json
[
  { "key": "sydney-cbd", "label": "Sydney CBD" },
  { "key": "sydney-north", "label": "North Sydney" },
  { "key": "sydney-west", "label": "Western Sydney" }
]
```

Each option has:

| Field | Type | Description |
|-------|------|-------------|
| `key` | string | The value to submit in `postingRequirements` when ordering |
| `label` | string | Human-readable display text |

---

### Notes

- Facet names are **case-sensitive**: `JobCategory` ≠ `jobcategory`. Use the exact `name` from the facet definition.
- Credential field names vary by channel (e.g., SEEK uses `agency_or_company` + `organization_id`, Stepstone uses `organization_id` + `email`). Check the [MOC details endpoint](managing-contracts.endpoints.md#get-productschannelsmocs) for the correct fields.

### Errors

| Status | Cause |
|--------|-------|
| 400 | Missing `credentials` when using a channel ID, or invalid credential values |
| 404 | Channel/contract not found, or facet name does not exist on this channel |

---

## Multi-term Autocomplete

Some facets depend on multiple input values rather than a single search string. For these, pass `term` as an array:

```json
{
  "term": ["Fullstack Engineer", "London", "Engineering"]
}
```

Each element filters the results differently depending on the channel's configuration. The facet's `autocomplete.required_parameters` array indicates when multi-term is expected.
