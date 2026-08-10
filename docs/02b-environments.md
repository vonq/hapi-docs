---
title: Environments
description: Production vs Sandbox-base URLs, separate provisioning, what sandbox does and does not do, and how to test campaign lifecycles without real deliveries.
category: getting-started
endpoints: []
prerequisites: [introduction, api-overview]
concepts: [sandbox, production, environments, provisioning, status_simulation]
related: [api-overview, authentication, campaign-status]
audience: [developer]
difficulty: beginner
---

# Environments

> HAPI runs in two isolated environments. Knowing what Sandbox does-and deliberately does not-do will save you significant time during integration.

## Base URLs

| Environment | Base URL | Purpose |
|-------------|----------|---------|
| **Production** | `https://marketplace.api.vonq.com` | Live environment. Orders are processed and charged. |
| **Sandbox** | `https://marketplace-sandbox.api.vonq.com` | Testing environment. Orders are accepted but never executed or charged. |

## Separate Identity & Provisioning

Your partner account has a **distinct identity in each environment**. Sandbox is not a mirror of your production account-it is provisioned separately by VONQ:

- **Separate credentials**-you receive one secret key per environment during onboarding. A production key never authenticates against Sandbox, and vice versa.
- **Separate data**-ATS users, customers, wallets, contracts, and campaigns exist per environment. Nothing you create in Sandbox appears in Production.
- **Separate configuration**-account settings and webhook setup (callback URLs, custom authentication headers, loose validation, etc.) are configured per environment. If you want webhooks in both environments, they must be configured in both-ask your account manager.

## Product Portfolio

The Sandbox portfolio-channels and products, for both Job Marketing and Job Post-is kept in sync with Production on a **best-effort basis**. It closely mirrors Production, but do not assume exact parity: always discover products through the [marketplace search](guides/05-products/02-marketplace.md) and [contracts](guides/06-contracts/01-introduction.md) endpoints in the environment you are calling, rather than reusing identifiers across environments.

## What Sandbox Does Not Do

Sandbox accepts and validates your API calls exactly like Production, but there is **no campaign delivery** behind it:

| Capability | Production | Sandbox |
|------------|------------|---------|
| Product search, taxonomy, posting requirements | ✅ | ✅ |
| Campaign ordering + validation | ✅ | ✅ Accepted, **never executed or charged** |
| Vacancies actually posted to job boards | ✅ | ❌ Never |
| Campaign/product status transitions | Driven by real job-board responses | Only via [status simulation](guides/08-campaigns/status.md#sandbox-status-simulation)-otherwise campaigns stay in their initial status |
| `PATCH /campaigns/{campaignId}/edit` status simulation | ❌ Blocked | ✅ Sandbox-only |
| Direct Apply / Screening candidates | Real applications from job boards | ❌ No delivery means no real candidates can apply |

<!-- theme: warning -->
> ### Statuses Don't Move by Themselves in Sandbox
> A Sandbox campaign stays in its initial status indefinitely-no job board ever confirms it. If your integration seems to "hang waiting for the campaign to go online" in Sandbox, that is expected: use the [status-simulation endpoint](guides/08-campaigns/status.md#sandbox-status-simulation) to advance it through its lifecycle.

### Simulating Deliveries

Beyond the self-service status-simulation endpoint, VONQ can run additional simulations in Sandbox (for example, exercising application delivery flows). These are arranged case by case-discuss your testing needs with your account manager or VONQ support.

## Recommended Integration Flow

1. Build and test the full order → status → webhook loop in Sandbox, using [status simulation](guides/08-campaigns/status.md#sandbox-status-simulation) to drive campaigns through their lifecycle.
2. Verify your webhook endpoint and its authentication headers with Sandbox deliveries (remember: webhooks are configured per environment).
3. Switch the base URL and secret key to Production. Nothing else in your integration should need to change.

## Related

- [API Overview](02-api-overview.md)-HTTP conventions, pagination, error handling
- [Authentication](guides/03-authentication-and-users/authentication.md)-per-environment secret keys
- [Status & Lifecycle](guides/08-campaigns/status.md)-sandbox status simulation reference
- [Campaign Webhooks](guides/08-campaigns/webhooks.md)-webhook setup and authentication
