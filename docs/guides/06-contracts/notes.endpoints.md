---
title: Contract Caveats & Notes - Endpoint Reference
description: Request/response examples for credential validation when creating contracts.
category: guides/contracts
---

> For conceptual overview, see [Contract Caveats & Notes](./notes.md).

# Contract Caveats & Notes - Endpoint Reference

## Credential Validation Example

Use `credentials_validation: "if_supported"` when creating a contract to validate credentials against the job board:

```http
POST https://marketplace.api.vonq.com/contracts/ HTTP/1.1
X-Auth-Token: <your Partner token here>
Content-Type: application/json
```

```json
{
  "channel_id": 1234,
  "credentials": {
    "agency_or_company": "Company",
    "organization_id": "org-12345"
  },
  "credentials_validation": "if_supported"
}
```

If validation fails (on a channel that supports it), the response includes credential-level errors:

```json
{
  "credentials": {
    "organization_id": ["Invalid advertiser ID"]
  }
}
```
