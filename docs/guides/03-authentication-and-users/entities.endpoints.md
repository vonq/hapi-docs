---
title: Entities - Endpoint Reference
description: Full request/response details for ATS and ATSUser endpoints.
category: guides/authentication-and-users
---

> For conceptual overview, see [Entities](./entities.md).

# Entities - Endpoint Reference

## GET /v3/ats/ats/me/

- **Auth**: secret key (`X-Auth-Token`, without `X-Customer-Id`)
- **Description**: Retrieve the authenticated partner account information.

```http
GET https://marketplace.api.vonq.com/v3/ats/ats/me/ HTTP/1.1
X-Auth-Token: <your Partner token here>
```

```json
{
  "id": "f1a2b3c4-d5e6-7890-abcd-ef1234567890",
  "name": "Your ATS Name",
  "hapi_partner_id": "f2a3b4c5-d6e7-8901-bcde-f12345678901"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `id` | uuid | Internal ATS identifier |
| `name` | string | Partner account name |
| `hapi_partner_id` | uuid | Public partner identifier |

---

## GET /v3/ats/atsuser/me/

- **Auth**: JWT or secret key + `X-Customer-Id`
- **Description**: Retrieve the authenticated customer's account information.

```http
GET https://marketplace.api.vonq.com/v3/ats/atsuser/me/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
```

```json
{
  "id": "d5e6f7a8-b9c0-1234-ef01-345678901234",
  "ats": {
    "id": "f1a2b3c4-d5e6-7890-abcd-ef1234567890",
    "name": "Your ATS Name",
    "hapi_partner_id": "f2a3b4c5-d6e7-8901-bcde-f12345678901"
  },
  "customer_id": "customer-123",
  "settings": {
    "ats_user_jwt_expiration": 0
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `id` | uuid | Internal ATSUser identifier |
| `ats` | object | Parent ATS account (see fields above) |
| `customer_id` | string | The customer identifier you provided |
| `settings` | object | JWT settings for this user |
| `settings.ats_user_jwt_expiration` | integer | JWT expiration time in seconds (`0` = default) |

---

## GET /v3/ats/users/

- **Auth**: secret key (`X-Auth-Token`, without `X-Customer-Id`)
- **Description**: List all ATSUsers (customers) under your partner account. Supports filtering and search.

```http
GET https://marketplace.api.vonq.com/v3/ats/users/ HTTP/1.1
X-Auth-Token: <your Partner token here>
```

| Parameter | In | Type | Description |
|-----------|------|------|-------------|
| `customer_id` | query | string | Filter by customer ID |
| `search` | query | string | Search across customer IDs |
| `ordering` | query | string | Field to order results by |

**Response**: Paginated list of ATSUser objects.

| Field | Type | Description |
|-------|------|-------------|
| `id` | uuid | Internal ATSUser identifier |
| `customer_id` | string | The customer identifier |
| `created_on` | datetime | When this ATSUser was created |

---

## POST /v3/ats/users/

- **Auth**: secret key (`X-Auth-Token`, without `X-Customer-Id`)
- **Description**: Explicitly create an ATSUser. If the `customer_id` already exists, the existing ATSUser is returned.

```http
POST https://marketplace.api.vonq.com/v3/ats/users/ HTTP/1.1
X-Auth-Token: <your Partner token here>
Content-Type: application/json

{
  "customer_id": "customer-456"
}
```

| Field | In | Type | Required | Description |
|-------|------|------|----------|-------------|
| `customer_id` | body | string | Yes | Your identifier for the customer (max 255 characters) |

**Response**: `201 Created` - returns the ATSUser object.

---

## GET /v3/ats/users/{customer_id}/

- **Auth**: secret key (`X-Auth-Token`, without `X-Customer-Id`)
- **Description**: Retrieve a specific ATSUser by their customer ID. Useful when authenticating at partner level and looking up a particular customer.

```http
GET https://marketplace.api.vonq.com/v3/ats/users/customer-123/ HTTP/1.1
X-Auth-Token: <your Partner token here>
```

**Response**: ATSUser object (same fields as the list endpoint).

<!-- theme: info -->
> The `/me/` endpoints identify the entity from your authentication headers - `GET /v3/ats/ats/me/` for the ATS, `GET /v3/ats/atsuser/me/` for the ATSUser. The `/users/` endpoints let you manage ATSUsers from a partner-level perspective - listing, creating, or looking up specific customers.

---

## GET /v3/ats/atsuser/me/settings/

- **Auth**: JWT or secret key + `X-Customer-Id`
- **Description**: Retrieve settings for the authenticated customer, including payment model and feature flags.

```http
GET https://marketplace.api.vonq.com/v3/ats/atsuser/me/settings/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
```

```json
{
  "settings": {
    "ats_managed_payment": true,
    "can_use_wallets": true,
    "can_pay_with_purchase_order": true,
    "can_pay_with_direct_charge": true,
    "invoice_currency": "EUR"
  },
  "payment_settings": [
    {
      "currency": "EUR",
      "min_topup": "1.11",
      "max_purchase_order": "1500.00",
      "max_outstanding_balance": "10000.00",
      "payment_method_types": ["card", "bank_transfer", "sepa_debit", "ideal", "sofort"],
      "gateway": {
        "provider": "stripe",
        "publishable_key": "pk_...",
        "settings": { "payment_method_types": ["card", "sepa_debit", "ideal", "sofort", "customer_balance"] }
      }
    }
  ]
}
```

**`settings` object**:

| Field | Type | Description |
|-------|------|-------------|
| `ats_managed_payment` | boolean | Whether the ATS manages payments on behalf of customers |
| `can_use_wallets` | boolean | Whether wallet top-ups are enabled |
| `can_pay_with_purchase_order` | boolean | Whether purchase order payments are enabled |
| `can_pay_with_direct_charge` | boolean | Whether direct card charges are enabled |
| `invoice_currency` | string | Default billing currency (e.g. `"EUR"`) |

The response includes additional internal feature flags (e.g. `smartfill`, `campaigns`, `direct_apply`, `analytics`) that are managed internally and may change without notice.

**`payment_settings` array** - one entry per enabled currency:

| Field | Type | Description |
|-------|------|-------------|
| `currency` | string | Currency code (e.g. `"EUR"`, `"USD"`) |
| `min_topup` | string | Minimum wallet top-up amount |
| `max_purchase_order` | string | Maximum purchase order amount |
| `max_outstanding_balance` | string | Maximum outstanding balance |
| `payment_method_types` | array | Supported payment methods for this currency |
| `gateway.provider` | string | Payment gateway provider (e.g. `"stripe"`) |
| `gateway.publishable_key` | string | Public key for client-side payment integration |

Settings are read-only from the API. To adjust your account configuration, contact your VONQ account manager. For details on payment-related settings, see [Wallets & Payments](../12-wallets-and-payments.md).
