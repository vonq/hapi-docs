---
title: Wallets & Payments - Endpoint Reference
description: Full request/response details for wallet creation, retrieval, billing portal, and the payment widget.
category: guides/wallets-and-payments
---

> For conceptual overview, see [Wallets & Payments](./12-wallets-and-payments.md).

## Endpoints

### POST /wallet

- **Auth**: token (`X-Auth-Token` + `X-Customer-Id`) or JWT
- **Description**: Create a wallet for the authenticated ATS user.

**Body Parameters**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `customerName` | string | No | Display name for the wallet holder |
| `currency` | string | No | ISO currency code (default: `USD`) |

```http
POST https://marketplace.api.vonq.com/wallet HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
Content-Type: application/json
```

```json
{
  "customerName": "Acme Corp",
  "currency": "EUR"
}
```

**Notes**
- Each ATS user can have only one wallet. Attempting to create a second returns `409`.
- Currency cannot be changed after creation. To use a different currency, create a new ATS user.

**Errors**

| Status | Cause |
|--------|-------|
| `400` | Invalid currency or customer name |
| `409` | Wallet already exists for this customer |

---

### GET /wallet

- **Auth**: token or JWT
- **Description**: Retrieve the wallet for the authenticated ATS user.

```http
GET https://marketplace.api.vonq.com/wallet HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
```

**Response:**

```json
{
  "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "balance": {
    "EUR": 125050,
    "amount": "1250.50",
    "currency": "EUR"
  },
  "created_date": "2025-05-02T05:51:33.290351Z",
  "has_billing_details": true,
  "missing_billing_keys": [],
  "min_topup": {
    "amount": "0.50",
    "currency": "EUR"
  },
  "max_purchase_order": {
    "amount": "5000.00",
    "currency": "EUR"
  },
  "max_outstanding_balance": {
    "amount": "10000.00",
    "currency": "EUR"
  }
}
```

**Response fields**

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Wallet identifier |
| `balance.amount` | string | Current balance as a decimal string (can be negative for PO accounts) |
| `balance.currency` | string | ISO currency code |
| `balance.{currency}` | integer | Balance in cents, keyed by the currency code (e.g., `"EUR": 125050`). The key name varies per wallet. |
| `has_billing_details` | boolean | Whether billing details are complete |
| `missing_billing_keys` | array | Required billing fields not yet provided (e.g., `["email", "address.country"]`) |
| `min_topup` | object | Minimum top-up amount |
| `max_purchase_order` | object | Maximum single PO amount |
| `max_outstanding_balance` | object | Maximum allowed negative balance |

---

### POST /wallet/billing-portal

- **Auth**: token or JWT
- **Description**: Generate a Stripe billing portal link for the customer to manage billing details, payment methods, and invoices.

**Query Parameters**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `partnerReturnUrl` | string | No | URL to redirect the customer back to after they finish |

```http
POST https://marketplace.api.vonq.com/wallet/billing-portal?partnerReturnUrl=https%3A%2F%2Fyour-ats.example.com%2Fsettings%2Fbilling HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
Content-Type: application/json
```

**Response:**

```json
{
  "billingPortalLink": "https://billing.stripe.com/session/live_..."
}
```

| Field | Type | Description |
|-------|------|-------------|
| `billingPortalLink` | string | Primary billing portal link |

**Notes**
- If unused, the link expires **5 minutes** after generation. Once clicked, the session expires within 1 hour of the most recent activity and can last up to 2 hours total.
- Customers must complete billing details here before they can top up, direct-charge a campaign, or use purchase orders.

---

### Payment Widget

The payment widget is a Stripe-managed iframe at `/wallet/topup.html`. It handles all card and bank input in a PCI-compliant way-no sensitive payment data touches your system.

**Iframe URL:**

```
https://marketplace.api.vonq.com/wallet/topup.html?walletId={walletId}&partnerId={partnerId}&returnUrl={returnUrl}
```

<!-- theme: info -->
> ### Iframe Dimensions
> To avoid scrollbars, the iframe should have a minimum height of **1330px** and a recommended minimum width of **520px**.

**Query Parameters**

| Parameter | Required | Description |
|-----------|----------|-------------|
| `walletId` | Yes | Wallet UUID |
| `partnerId` | Yes | Your partner ID |
| `returnUrl` | No | Redirect URL after 3DS/bank authentication |
| `campaignId` | No | For direct charge-locks amount to campaign total |
| `paymentMethodTypes` | No | Comma-separated list to filter available payment types |
| `successCallbackUrl` | No | URL called on successful payment |

<!-- theme: warning -->
> Topping up is not possible if the wallet balance is negative. Outstanding purchase order invoices must be paid before the wallet balance can become positive again. Direct charge payments are still allowed for a negative-balance wallet when the widget is loaded with a valid `campaignId`.

**Listening for Payment Events:**

The iframe communicates via `postMessage`. Listen for events on your parent window:

```javascript
window.addEventListener("message", (event) => {
  // Verify origin for security
  if (event.origin !== "https://marketplace.api.vonq.com") {
    return;
  }

  const data = JSON.parse(event.data);

  if (data.type === "payment_status") {
    switch (data.payload.value) {
      case "succeeded":
        // Payment complete-refresh wallet balance
        break;
      case "requires_action":
        // 3DS or bank redirect needed
        break;
      case "processing":
        // Payment is processing (e.g., ACH: 4–5 days)
        break;
      case "canceled":
        // Customer cancelled payment
        break;
    }
  }
});
```

**Payment Status Values:**

| Status | Description |
|--------|-------------|
| `requires_payment_method` | No payment method selected yet |
| `requires_confirmation` | Awaiting customer confirmation |
| `requires_action` | 3DS or bank redirect needed |
| `processing` | Payment processing (may take days for bank transfers/ACH) |
| `succeeded` | Payment complete-call `GET /wallet` to refresh balance |
| `canceled` | Payment cancelled by customer |

<!-- theme: warning -->
> ### One Payment Per Iframe
> Each iframe instance handles one payment attempt. Remove and re-embed the iframe for subsequent payments.
