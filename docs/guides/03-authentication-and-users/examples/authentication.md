---
title: Authentication Examples
description: Request/response examples for each authentication level-secret key, JWT, partner-level, and customer-level.
category: guides/authentication-and-users
endpoints:
  - GET /v3/ats/ats/me/
  - GET /v3/ats/atsuser/me/
prerequisites: [authentication]
concepts: [secret_key, jwt, x_auth_token, x_customer_id]
related: [authentication, entities]
audience: [developer]
difficulty: beginner
---

# Authentication Examples

> Real request/response examples showing how authentication levels work, JWT generation, and common permission errors.

This page contains copy-paste-ready request/response examples for each authentication level: partner (ATS), customer (ATSUser), and JWT. Each section includes both successful calls and common permission errors so you can see exactly what to expect.

## Partner-Level Authentication (ATS)

Authenticate with only `X-Auth-Token` (no `X-Customer-Id`) to access partner-level endpoints.

```http
GET https://marketplace.api.vonq.com/v3/ats/ats/me/ HTTP/1.1
X-Auth-Token: <your secret key>
Content-Type: application/json
```

```json
{
  "id": "f1a2b3c4-d5e6-7890-abcd-ef1234567890",
  "name": "Example Partner",
  "hapi_partner_id": "f2a3b4c5-d6e7-8901-bcde-f12345678901"
}
```

### Wrong Level: Calling a Partner Endpoint as an ATSUser

Adding `X-Customer-Id` to a partner-only endpoint returns `403 Forbidden`:

```http
GET https://marketplace.api.vonq.com/v3/ats/ats/me/ HTTP/1.1
X-Auth-Token: <your secret key>
X-Customer-Id: test
Content-Type: application/json
```

```json
{
  "detail": "You do not have permission to perform this action."
}
```

## Customer-Level Authentication (ATSUser)

Add `X-Customer-Id` to scope the request to a specific customer.

```http
GET https://marketplace.api.vonq.com/v3/ats/atsuser/me/ HTTP/1.1
X-Auth-Token: <your secret key>
X-Customer-Id: test
Content-Type: application/json
```

```json
{
  "id": "f3a4b5c6-d7e8-9012-cdef-123456789012",
  "ats": {
    "id": "f1a2b3c4-d5e6-7890-abcd-ef1234567890",
    "name": "Example Partner",
    "hapi_partner_id": "f2a3b4c5-d6e7-8901-bcde-f12345678901"
  },
  "customer_id": "test",
  "settings": {
    "ats_user_jwt_expiration": 0
  }
}
```

### Wrong Level: Calling a Customer Endpoint as an ATS

Omitting `X-Customer-Id` on a customer-scoped endpoint returns `403 Forbidden`:

```http
GET https://marketplace.api.vonq.com/v3/ats/atsuser/me/ HTTP/1.1
X-Auth-Token: <your secret key>
Content-Type: application/json
```

```json
{
  "detail": "You do not have permission to perform this action."
}
```

## JWT Generation and Usage

### Step 1: Generate a JWT (Server-Side)

Use your secret key to generate a JWT scoped to a customer. This call is made from your server.

```http
POST https://marketplace.api.vonq.com/v3/ats/users/test/generate-jwt-token/ HTTP/1.1
X-Auth-Token: <your secret key>
Content-Type: application/json
```

```json
{
  "token": "<partner-id>::<jwt-header>.<jwt-payload>.<jwt-signature>"
}
```

### Step 2: Use the JWT (Client-Side)

Pass the JWT in the `X-Authorization` header. No secret key or `X-Customer-Id` needed-the customer identity is embedded in the token.

```http
GET https://marketplace.api.vonq.com/v3/ats/atsuser/me/ HTTP/1.1
X-Authorization: Bearer <partner-id>::<jwt-header>.<jwt-payload>.<jwt-signature>
Content-Type: application/json
```

```json
{
  "id": "f3a4b5c6-d7e8-9012-cdef-123456789012",
  "ats": {
    "id": "f1a2b3c4-d5e6-7890-abcd-ef1234567890",
    "name": "Example Partner",
    "hapi_partner_id": "f2a3b4c5-d6e7-8901-bcde-f12345678901"
  },
  "customer_id": "test",
  "settings": {
    "ats_user_jwt_expiration": 0
  }
}
```

## Endpoints That Accept Both Auth Levels

Some endpoints work with either ATS-level or ATSUser-level authentication. For example, searching channels:

**As ATS (partner-level):**

```http
GET https://marketplace.api.vonq.com/products/channels/mocs/?search=seek HTTP/1.1
X-Auth-Token: <your secret key>
Content-Type: application/json
```

**As ATSUser (customer-level):**

```http
GET https://marketplace.api.vonq.com/products/channels/mocs/?search=seek HTTP/1.1
X-Auth-Token: <your secret key>
X-Customer-Id: test
Content-Type: application/json
```

Both return the same response:

```json
{
  "count": 5,
  "next": null,
  "previous": null,
  "results": [...]
}
```

## Related

- [Authentication](../authentication.md)-secret key vs JWT, endpoint reference
- [Entities](../entities.md)-ATS and ATSUser relationships, scoping rules
