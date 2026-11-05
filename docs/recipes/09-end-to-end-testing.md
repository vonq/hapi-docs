---
id: recipe-end-to-end-testing
title: Recipe-End-to-End Testing
description: Prove the integration before go-live. What Sandbox can and cannot test, how to run every recipe's scenarios as an automated suite, how to drive a campaign lifecycle, and what to switch for Production.
category: recipes
endpoints:
- GET /v3/ats/atsuser/me/
- POST /campaigns/order
- PATCH /campaigns/{campaignId}/edit
- GET /campaigns/{campaignId}
- PUT /campaigns/{campaignId}/product_cancellation
prerequisites:
- recipes-introduction
- environments
related:
- environments
- campaign-status
- campaign-webhooks
- recipe-checkout-and-order
- recipe-campaign-tracking
audience:
- developer
difficulty: intermediate
keywords:
- sandbox
- test suite
- integration tests
- status simulation
- test customer
- go-live
- duplicate order
- rate limit
---

# End-to-End Testing

> Turn every recipe's **Validate** table into an automated suite that runs against Sandbox, drive a campaign through its whole lifecycle there, and know which checks only Production can answer.

## What Sandbox Tests, and What It Cannot

Sandbox accepts and validates every call like Production, but it never sends an order to a job board. That holds for every board, including boards you have your own account with. See [Environments](../02b-environments.md).

| You want to test | Sandbox | How |
|------------------|---------|-----|
| Customers, settings, JWT | Yes | `ACC-*` scenarios |
| Taxonomy, product search, bundles, recommendations | Yes | `VAC-*`, `CHN-*` scenarios. Sandbox's catalog mirrors Production on a best-effort basis, with different IDs |
| Contracts and credential forms | Yes | `CON-*` scenarios. Boards that check credentials (SEEK) need a real board account |
| Posting requirement forms and validation | Yes | `REQ-*` scenarios with a contract-based board such as SEEK and a marketplace product with `has_product_specs: true` |
| Validation and ordering | Yes, free of charge | `ORD-*` scenarios |
| Status changes, webhooks, takedowns | Yes, with status simulation | `TRK-*` scenarios. Without simulation, campaigns stay `in progress` |
| A real posting on a real board | No | Only in Production. Agree with your VONQ account manager how to verify your first live orders |
| Direct Apply and screening candidates | No | VONQ can arrange simulations case by case |

## Integration Variants

| Decision | Options | What changes |
|----------|---------|--------------|
| Which tests place orders | Every run, or a separate suite run on demand | Validation scenarios are cheap and safe to run on every build; orders create Sandbox campaigns you then simulate and take offline |
| Webhook testing | A public HTTPS tunnel to your development machine, or a deployed test environment | Private and `localhost` URLs are rejected |
| Test customers | One per developer or branch, or one per run | Separate customers keep favorites, contracts and duplicate-order protection from interfering between runs |

## Build

### Step 1: Set up the test environment

- Sandbox base URL `https://marketplace-sandbox.api.vonq.com` and the Sandbox secret key from environment variables. Never commit the key.
- A dedicated test customer ID that no real customer uses, for example `e2e-<branch>`. Customers are created on first use (see [Customer Accounts](./02-customer-accounts.md)).
- No IDs copied from Production. Discover products, channels and locations in Sandbox at the start of each run.

### Step 2: Turn the Validate tables into tests

Give each test the scenario ID as its name (`CHN-03 bundle contents resolve`), so failures map straight to a recipe. Split them into layers:

| Layer | Runs | Contents |
|-------|------|----------|
| Unit | Every build, no network | Job-to-vacancy mapping, order body assembly, error parsing (including positional product errors) |
| Sandbox contract tests | Every build | All scenarios that only read or validate: `ACC-*`, `VAC-*`, `CHN-*`, `REQ-*`, and the validation-only `ORD-*` |
| Sandbox order suite | On demand or nightly | Scenarios that create contracts or orders: `CON-*`, ordering `ORD-*`, `TRK-*` |
| UI end-to-end | Before release | The wizard happy path, plus one failure on each step |

### Step 3: Drive a full lifecycle

1. Order a cheap marketplace product without posting requirements (`POST /campaigns/order`).
2. Move it online with [status simulation](../guides/08-campaigns/status.md#sandbox-status-simulation): `PATCH /campaigns/{campaignId}/edit` with a top-level `status: "online"` and the product `online`, `deliveredOn` and `jobBoardLink`.
3. Check your UI, your webhook receiver (if webhooks are enabled) and your incremental sync.
4. Take the product offline with `PUT /campaigns/{campaignId}/product_cancellation`, then confirm with `GET /campaigns/{campaignId}`.
5. Repeat with a product simulated as `not processed` and check the reorder path.

Include a top-level `status` whenever you want the campaign itself to change: simulating only product statuses leaves the campaign status and `modifiedOn` as they were.

### Step 4: Keep test runs clean

- Make every order body unique, for example with a run ID in the title and a label. Accounts can reject a byte-identical order placed within a configured window.
- Run order tests one after another, within 60 requests per 60 seconds, and back off on `429`. Its body may not be JSON.
- Only take offline the campaigns your run created; find them by their label.
- Log every HAPI call your ATS makes: method, path, status, duration, and the request and response bodies with the secret key masked. That log makes test failures and support requests much faster to resolve.

### Step 5: Switch to Production

1. Ask VONQ to configure in Production whatever you rely on in Sandbox: webhooks and their headers, loose validation, payment methods, feature settings. Settings are per environment.
2. Switch the base URL and the secret key. Nothing else in your code should change.
3. Discover product, channel and location IDs again; they differ from Sandbox.
4. Remove status simulation from every code path outside tests. It returns `403` in Production.
5. Watch the first live orders until each product is `online` or `not processed`.

## Store

| What | Why |
|------|-----|
| Test customer IDs per developer or branch | Isolates runs |
| Campaign IDs and labels created by each run | Cleanup and takedowns of your own test campaigns only |
| The call log of failed runs | Debugging and support requests |

## Rules

1. A Production key does not work on Sandbox and the other way round; customers, contracts, wallets and settings exist separately in each.
2. Sandbox statuses never move by themselves. A test that waits for `online` without simulation hangs.
3. Status simulation works only with the `X-Customer-Id` that placed the order, and only in Sandbox.
4. Sandbox location search only finds places Sandbox already knows. Do not fail a test because a small town is missing there.
5. Product and location IDs differ between environments. Never hard-code them in tests.
6. Webhooks are configured per environment and need a public HTTPS URL.

## Validate

| ID | Given | When | Then |
|----|-------|------|------|
| `E2E-01` | A Production key | Any call to the Sandbox base URL | `401` |
| `E2E-02` | A new test customer ID | `GET /v3/ats/atsuser/me/` with that `X-Customer-Id` | `200` with the same `customer_id` |
| `E2E-03` | A valid Sandbox order | Read the campaign status every few minutes for 10 minutes | Still `in progress`; nothing is posted |
| `E2E-04` | `E2E-03` | Steps 2 to 4 of the lifecycle | Product `online`, then `offline`; the campaign follows when it was its last online product; a webhook arrives within a few minutes when enabled |
| `E2E-05` | An account with duplicate-order protection | Two orders with byte-identical bodies, then one with a new run ID in the title | The second is rejected; the third is accepted |
| `E2E-06` | A Production campaign | `PATCH /campaigns/{campaignId}/edit` with a status | `403` |
| `E2E-07` | The full suite | Run every scenario of every recipe | All pass, or are skipped with a recorded reason (feature disabled for the account, webhooks not enabled, no real board account) |

## Done When

- [ ] Every scenario ID from the recipes exists as a test, passes, or has a recorded reason for being skipped.
- [ ] One full lifecycle runs in Sandbox: order, online, webhook or sync, takedown, offline.
- [ ] Test runs use dedicated test customers and unique order bodies, and clean up only their own campaigns.
- [ ] HAPI calls are logged with the secret key masked.
- [ ] Production settings match what the integration relies on, and status simulation is absent from production code paths.

## Related

- [Environments](../02b-environments.md): Sandbox and Production differences
- [Status & Lifecycle](../guides/08-campaigns/status.md#sandbox-status-simulation): status simulation reference
- [Campaign Webhooks](../guides/08-campaigns/webhooks.md): webhook setup per environment
- [Recipes](./01-introduction.md): all recipes and their scenario IDs
