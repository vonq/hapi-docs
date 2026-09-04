---
id: managing-contracts-endpoints
title: Managing Contracts - Endpoint Reference
description: Full request/response details for MOC, contract CRUD, and contract group endpoints.
category: guides/contracts
related:
- managing-contracts
endpoints:
- GET /products/channels/mocs/
- GET /products/channels/mocs/{id}/
- GET /contracts/
- POST /contracts/
- GET /contracts/single/{contract_id}/
- GET /contracts/multiple/{contracts_ids}/
- PATCH /contracts/single/{contract_id}/
- DELETE /contracts/{contract_id}/
- GET /igb/contracts/groups/
- POST /igb/contracts/groups/
- GET /igb/contracts/groups/{group_idx}/
- PUT /igb/contracts/groups/{group_idx}/
- PATCH /igb/contracts/groups/{group_idx}/
- DELETE /igb/contracts/groups/{group_idx}/
---

> For conceptual overview, see [Managing Contracts](./managing-contracts.md).

# Managing Contracts - Endpoint Reference

## Channel Discovery (MOC Endpoints)

### GET /products/channels/mocs/

- **Auth**: token + `X-Customer-Id`, or JWT
- **Description**: List channels that support contracts. Use this to build a channel browser or search for a specific channel.

**Query Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `search` | string | Filter by channel name |
| `limit` | integer | Results per page |
| `offset` | integer | Pagination offset |

```http
GET https://marketplace.api.vonq.com/products/channels/mocs/?search=SEEK HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
```

```json
{
  "count": 1,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 1234,
      "name": "SEEK",
      "url": "https://www.seek.com.au",
      "type": "job board",
      "mc_enabled": true
    }
  ]
}
```

---

### GET /products/channels/mocs/{id}/

- **Auth**: token + `X-Customer-Id`, or JWT
- **Description**: Retrieve full details for a channel, including credential field definitions, setup instructions, and posting requirements. This is the essential call before creating a contract - it tells you exactly what the user needs to provide.

<!-- theme: info -->
> Unlike the list endpoint, the detail endpoint is customer-scoped: `X-Customer-Id` is required (a partner-level token without it returns `403` "X-Customer-Id header required").

**Path Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | integer | The channel ID |

```http
GET https://marketplace.api.vonq.com/products/channels/mocs/1234/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
```

```json
{
  "id": 1234,
  "name": "SEEK",
  "url": "https://www.seek.com.au",
  "type": "job board",
  "mc_enabled": true,
  "manual_setup_required": true,
  "setup_instructions": "<p>To set up SEEK, you need an advertiser account...</p>",
  "feed_url": null,
  "contract_credentials": [
    {
      "name": "agency_or_company",
      "label": "Agency or Company",
      "sort": "1",
      "description": "Select whether you are an agency or a company",
      "url": null,
      "options": [
        { "key": "Agency", "label": "Agency" },
        { "key": "Company", "label": "Company" }
      ]
    },
    {
      "name": "organization_id",
      "label": "Organization ID",
      "sort": "2",
      "description": "Your SEEK advertiser ID",
      "url": null,
      "options": null
    }
  ],
  "posting_requirements": [
    {
      "name": "location",
      "label": "Location",
      "sort": "1",
      "required": true,
      "type": "AUTOCOMPLETE",
      "options": []
    },
    {
      "name": "JobCategory",
      "label": "Job Category",
      "sort": "2",
      "required": true,
      "type": "AUTOCOMPLETE",
      "options": []
    }
  ],
  "product": {
    "product_id": "d3e4f5a6-b7c8-9012-cdef-123456789012",
    "title": "SEEK Standard Ad"
  }
}
```

**Response fields**:

| Field | Type | Description |
|-------|------|-------------|
| `id` | integer | Channel identifier |
| `name` | string | Channel name |
| `url` | string | Channel website URL |
| `type` | string | Channel type (e.g., `"job board"`, `"social media"`) |
| `mc_enabled` | boolean | Whether the channel supports contracts |
| `manual_setup_required` | boolean | Whether the user must follow setup steps before creating a contract |
| `setup_instructions` | string \| null | HTML setup instructions for the channel |
| `feed_url` | string \| null | Job feed URL template for this channel |
| `logo_url` | string | Channel logo image URL |
| `logo_square_url` | string | Square variant of the channel logo image URL |
| `logo_rectangle_url` | string | Rectangle variant of the channel logo image URL |
| `contract_credentials` | array | Credential field definitions (see below) |
| `posting_requirements` | array | Posting requirement facets for this channel |
| `product` | object \| null | The associated My Contract product |

**Credential field definitions** (each item in `contract_credentials`):

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | The key to use in the `credentials` object when creating a contract |
| `label` | string | Display label for the field |
| `sort` | string | Sort order for display |
| `description` | string \| null | Help text explaining what this credential is |
| `url` | string \| null | If present, the field uses an OAuth flow - redirect the user to this URL |
| `options` | array \| null | If present, render as a dropdown with `key`/`label` pairs |

---

## Contract CRUD

### GET /contracts/

- **Auth**: token + `X-Customer-Id`, or JWT
- **Description**: List all contracts for the authenticated customer.

**Query Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `limit` | integer | Results per page (default: 50, max: 100) |
| `offset` | integer | Pagination offset (default: 0) |
| `channel_id` | integer | Filter by channel ID |
| `channel_name` | string | Filter by channel name |
| `label` | object | Filter by labels - pass as `label[key]=value` query params |
| `group_id` | string | Filter by contract group ID |
| `group_idx` | string | Filter by contract group index |

```http
GET https://marketplace.api.vonq.com/contracts/?limit=10 HTTP/1.1
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
      "contract_id": "c1d2e3f4-a5b6-7890-abcd-ef1234567890",
      "customer_id": "c2d3e4f5-a6b7-8901-bcde-f12345678901",
      "channel_id": 1234,
      "alias": "SEEK - Sydney Office",
      "class_name": "seek-anz",
      "facets": [],
      "product": {
        "product_id": "d3e4f5a6-b7c8-9012-cdef-123456789012",
        "title": "SEEK Standard Ad"
      },
      "expiry_date": null,
      "credits": null,
      "purchase_price": { "amount": 350.00, "currency": "AUD" },
      "has_errors": false,
      "igb_customer_id": "a1b2c3d4",
      "group": {
        "id": "c5d6e7f8-a9b0-1234-ef01-345678901234",
        "idx": 0,
        "igb_customer_id": "a1b2c3d4",
        "name": "Default"
      },
      "channel": {
        "id": 1234,
        "name": "SEEK",
        "url": "https://www.seek.com.au",
        "type": "job board",
        "mc_enabled": true
      }
    }
  ]
}
```

**Notes**:
- The list endpoint returns summary objects without `credentials` or `posting_requirements`. Use `GET /contracts/single/{contract_id}/` for full details.
- Filter by label: `?label[team]=recruiting&label[region]=emea`.
- Contracts have no `status` field. Each list row carries a `has_errors` boolean - the health signal for the contract. When it is `true`, fetch `GET /contracts/single/{contract_id}/` and inspect the `errors` array for details (e.g. deactivated channel, invalid credentials).

---

### POST /contracts/

- **Auth**: token + `X-Customer-Id`, or JWT
- **Description**: Create a new contract with a job board channel.

**Request Body**:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `channel_id` | integer | yes | The channel to create a contract for. Must have `mc_enabled: true`. |
| `credentials` | object | yes | Channel-specific credentials. Fields depend on the channel (see MOC details). |
| `alias` | string | no | User-facing name for this contract. Defaults to the channel name. |
| `credentials_validation` | string | no | Set to `"if_supported"` to validate credentials against the channel. |
| `followed_instructions` | boolean | no | Set to `true` to confirm setup instructions were followed. Required when `manual_setup_required` is `true` on the channel. |
| `posting_duration_days` | integer | no | Default posting duration in days (min: 7, max: 365). Pass `-1` to use the channel's default duration. |
| `credits` | integer | no | Number of credits for this contract. |
| `expiry_date` | datetime | no | Expiration date for this contract. Must be a full ISO 8601 datetime (e.g. `2027-06-30T00:00:00Z`) - a date-only value such as `2027-06-30` is rejected. |
| `group_id` | string (UUID) | no | Contract group to assign to. Defaults to the default group. |
| `purchase_price` | object | no | Purchase price - `amount` (number) and `currency` (string). |
| `labels` | object | no | Key-value pairs for filtering (max 50 pairs, each key/value max 32 chars). |
| `posting_requirements_defaults` | object | no | Default posting requirement values. Stored for the partner to read and apply manually - not automatically applied during ordering. |
| `facets` | array | no | Additional channel configuration parameters. |

Create a contract for SEEK with credential validation:

```http
POST https://marketplace.api.vonq.com/contracts/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
Content-Type: application/json

{
  "channel_id": 1234,
  "alias": "SEEK - Sydney Office",
  "credentials": {
    "agency_or_company": "Company",
    "organization_id": "org-12345"
  },
  "credentials_validation": "if_supported",
  "followed_instructions": true,
  "posting_duration_days": 30
}
```

```json
{
  "contract_id": "c1d2e3f4-a5b6-7890-abcd-ef1234567890",
  "customer_id": "c2d3e4f5-a6b7-8901-bcde-f12345678901",
  "channel_id": 1234,
  "alias": "SEEK - Sydney Office",
  "credentials": {
    "agency_or_company": "Company",
    "organization_id": "EXvFEpB9RMqfpX..."
  },
  "class_name": "seek-anz",
  "facets": {},
  "product": {
    "product_id": "d3e4f5a6-b7c8-9012-cdef-123456789012",
    "title": "SEEK Standard Ad"
  },
  "posting_requirements": [
    {
      "name": "location",
      "label": "Location",
      "sort": "1",
      "required": true,
      "type": "AUTOCOMPLETE",
      "options": [],
      "autocomplete": {
        "url": "/contracts/posting-requirements/c1d2e3f4-a5b6-7890-abcd-ef1234567890/location/"
      }
    }
  ],
  "expiry_date": null,
  "credits": null,
  "purchase_price": null,
  "posting_duration_days": 30,
  "group": {
    "id": "c5d6e7f8-a9b0-1234-ef01-345678901234",
    "idx": 0,
    "igb_customer_id": "a1b2c3d4",
    "name": "Default"
  },
  "labels": {},
  "posting_requirements_defaults": {},
  "errors": []
}
```

**Contract Response Fields**:

| Field | Type | Description |
|-------|------|-------------|
| `contract_id` | string (UUID) | Contract identifier |
| `customer_id` | string | Customer/ATSUser identifier |
| `channel_id` | integer | Channel identifier |
| `alias` | string | User-facing name for this contract |
| `credentials` | object | Credential values - text-input fields are returned as encrypted base64 strings, dropdown-selection fields as plaintext |
| `class_name` | string | Internal channel class identifier (e.g., `"seek-anz"`, `"stepstone"`) |
| `facets` | object \| array | Additional channel configuration |
| `product` | object | Associated My Contract product - `product_id` and `title` |
| `posting_requirements` | array | Posting requirement facets for this channel (only on single/detail endpoints) |
| `expiry_date` | string \| null | Contract expiration date (ISO 8601) |
| `credits` | integer \| null | Remaining credits |
| `purchase_price` | object \| null | Purchase price - `amount` and `currency` |
| `posting_duration_days` | integer | Default posting duration in days |
| `group` | object | Contract group - `id`, `idx`, `igb_customer_id`, `name` |
| `labels` | object | Key-value label pairs |
| `posting_requirements_defaults` | object | Default posting requirement values |
| `errors` | array | Contract errors (e.g., deactivated channel, invalid credentials) |

**Notes**:
- Each channel requires different credential fields. Always check the channel MOC details first.
- Common credential fields by channel:

| Channel | Credential Fields |
|---------|-------------------|
| SEEK | `agency_or_company`, `organization_id` |
| Stepstone | `organization_id`, `email` |
| Infojobs | `soapUsername`, `soapPassword` |

- Error responses may include a structured `errorDetails` array alongside the standard field-level errors:

```json
{
  "group_id": ["The contract for the same channel already exists in this group."],
  "errorDetails": [
    {
      "path": "group_id",
      "code": "invalid",
      "message": "The contract for the same channel already exists in this group.",
      "params": []
    }
  ]
}
```

- The one-contract-per-channel rule applies **per group**. To hold a second contract for the same channel (e.g. separate credentials per brand or business unit), first create a new [contract group](#contract-groups) and pass its `group_id` when creating the contract.

---

### GET /contracts/single/{contract_id}/

- **Auth**: token + `X-Customer-Id`, or JWT
- **Description**: Retrieve full details for a single contract, including credentials (masked), posting requirements, and any errors.

**Path Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `contract_id` | string (UUID) | The contract identifier |

```http
GET https://marketplace.api.vonq.com/contracts/single/c1d2e3f4-a5b6-7890-abcd-ef1234567890/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
```

Returns the full contract object (same structure as the create response). The `posting_requirements` array contains the facet definitions for this contract's channel - use these when building the posting form for campaigns. See [Posting Requirements](./posting-requirements.md).

**Notes**:
- The `errors` array is populated when there are issues with the contract (e.g., the channel has been deactivated or credentials are invalid).

---

### GET /contracts/multiple/{contracts_ids}/

- **Auth**: token + `X-Customer-Id`, or JWT
- **Description**: Retrieve full details for multiple contracts in a single request.

**Path Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `contracts_ids` | string | Comma-separated list of contract IDs (max 50) |

**Query Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `limit` | integer | Results per page (default: 50, max: 100) |
| `offset` | integer | Pagination offset |

```http
GET https://marketplace.api.vonq.com/contracts/multiple/c1d2e3f4-a5b6-7890-abcd-ef1234567890,c6d7e8f9-a0b1-2345-f012-567890123456/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
```

Returns a paginated list of full contract objects.

---

### PATCH /contracts/single/{contract_id}/

- **Auth**: token + `X-Customer-Id`, or JWT
- **Description**: Update an existing contract. Only the fields you include are updated.

**Path Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `contract_id` | string (UUID) | The contract identifier |

**Updatable fields**:

| Field | Type | Description |
|-------|------|-------------|
| `alias` | string | Updated display name |
| `credentials` | object | Updated credentials |
| `credentials_validation` | string | Set to `"if_supported"` to validate new credentials |
| `labels` | object | Updated labels |
| `posting_duration_days` | integer | Updated default posting duration (min: 7, max: 365; `-1` for the channel default) |
| `posting_requirements_defaults` | object | Updated default posting requirement values |

<!-- theme: warning -->
> ### Limited updatable fields
> Only `alias`, `credentials`, `credentials_validation`, `labels`, `posting_duration_days`, and `posting_requirements_defaults` can be changed after creation. Other fields fall into two behaviors:
> - **Rejected with `400`**: `group_id` - the response is `{"group_id": ["Cannot change the contract group once it is set."]}`.
> - **Silently ignored**: `expiry_date`, `credits`, and `purchase_price` - the request returns `200` but the values are left unchanged. Do not rely on the `200` status alone; compare the response body to confirm the change was applied.

Update a contract's alias and credentials:

```http
PATCH https://marketplace.api.vonq.com/contracts/single/c1d2e3f4-a5b6-7890-abcd-ef1234567890/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
Content-Type: application/json

{
  "alias": "SEEK - Melbourne Office",
  "credentials": {
    "agency_or_company": "Agency",
    "organization_id": "org-67890"
  },
  "credentials_validation": "if_supported"
}
```

---

### DELETE /contracts/{contract_id}/

- **Auth**: token + `X-Customer-Id`, or JWT
- **Description**: Delete a contract.

**Path Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `contract_id` | string (UUID) | The contract identifier |

**Response**: `204 No Content`

```http
DELETE https://marketplace.api.vonq.com/contracts/c1d2e3f4-a5b6-7890-abcd-ef1234567890/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
```

**Notes**:
- Deletion is synchronous - once the `204` is returned, the contract is gone. Repeating the request (or deleting an unknown contract ID) returns `404`, so a `404` on retry means the earlier delete succeeded.

<!-- theme: danger -->
> ### Deleting contracts with active campaigns
> Deleting a contract removes the stored credentials. Active campaigns using this contract may no longer be editable or stoppable programmatically. Always check for active campaigns before deleting. This action is irreversible.

---

## Contract Groups

### GET /igb/contracts/groups/

- **Auth**: token + `X-Customer-Id`, or JWT
- **Description**: List all contract groups for the authenticated customer.

**Query Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `ordering` | string | Field to order results by |
| `search` | string | Search term to filter groups |

```http
GET https://marketplace.api.vonq.com/igb/contracts/groups/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
```

```json
[
  {
    "id": "c5d6e7f8-a9b0-1234-ef01-345678901234",
    "name": "Default",
    "idx": 0,
    "igb_customer_id": "a1b2c3d4"
  },
  {
    "id": "b4f3a6e2-8d5c-4f9b-a03e-2c4d6f8a1b2c",
    "name": "Premium Channels",
    "idx": 1,
    "igb_customer_id": "a1b2c3d4"
  }
]
```

**Response fields**:

| Field | Type | Description |
|-------|------|-------------|
| `id` | string (UUID) | Group identifier |
| `name` | string | Group display name |
| `idx` | integer | Numeric index. `0` is always the default group. |
| `igb_customer_id` | string | The customer ID associated with contracts in this group |

---

### POST /igb/contracts/groups/

- **Auth**: token + `X-Customer-Id`, or JWT
- **Description**: Create a new contract group.

**Request Body**:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Group name. Must be unique per customer. |

```http
POST https://marketplace.api.vonq.com/igb/contracts/groups/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
Content-Type: application/json

{
  "name": "APAC Channels"
}
```

```json
{
  "id": "c5d6e7f8-9a0b-4c1d-2e3f-4a5b6c7d8e9f",
  "name": "APAC Channels",
  "idx": 2,
  "igb_customer_id": "a1b2c3d4"
}
```

---

### GET /igb/contracts/groups/{group_idx}/

- **Auth**: token + `X-Customer-Id`, or JWT
- **Description**: Retrieve details for a single contract group.

**Path Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `group_idx` | integer or UUID | The group index (e.g. `0`). The group's UUID `id` is also accepted. |

---

### PUT /igb/contracts/groups/{group_idx}/

- **Auth**: token + `X-Customer-Id`, or JWT
- **Description**: Update a contract group's name.

**Path Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `group_idx` | integer or UUID | The group index (e.g. `0`). The group's UUID `id` is also accepted. |

**Request Body**:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | New group name |

---

### PATCH /igb/contracts/groups/{group_idx}/

- **Auth**: token + `X-Customer-Id`, or JWT
- **Description**: Partially update a contract group. Same as PUT, but all fields are optional.

**Path Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `group_idx` | integer or UUID | The group index (e.g. `0`). The group's UUID `id` is also accepted. |

**Request Body**:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | no | New group name |

---

### DELETE /igb/contracts/groups/{group_idx}/

- **Auth**: token + `X-Customer-Id`, or JWT
- **Description**: Delete a contract group.

**Path Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `group_idx` | integer or UUID | The group index (e.g. `0`). The group's UUID `id` is also accepted. |

**Response**: `204 No Content`

**Notes**:
- The default group (idx `0`) cannot be deleted.
- Groups with associated contracts cannot be deleted. Move or delete the contracts first.
