---
title: CPA+ in Campaigns
description: CPA+ product-specific details within the campaigns context-orderedCpa, cpaResults.
category: guides/campaigns
endpoints: []
prerequisites: [campaign-ordering]
concepts: [cpa_plus, orderedCpa, cpaResults, hiring_goal]
related: [cpa, campaign-ordering, campaign-status]
audience: [developer]
difficulty: intermediate
---

# CPA+ in Campaigns
> How CPA+ products appear in campaign responses-tracking progress, understanding restrictions, and receiving updates.

## Overview

CPA+ products are ordered through HAPI's standard campaign ordering flow-include the CPA+ product ID in `orderedProducts` like any other product. The differences emerge after ordering: the campaign response includes CPA+-specific fields for tracking application progress, and certain campaign operations (like editing) are restricted.

For a full introduction to CPA+-what it is, how AI screening works, application delivery, and the polling endpoints-see [CPA+](../09-cpa.md).

## CPA+ Fields in Campaign Responses

When you retrieve a campaign containing a CPA+ product (`GET /campaigns/{campaignId}`), two additional fields appear:

- **`orderedCpa`** in `orderedProductsSpecs[]`-records the hiring goal purchased
- **`cpaResults`** in `postings[]`-tracks real-time application progress

See [CPA+ in Campaigns - Endpoint Reference](./cpa.endpoints.md) for full JSON response examples and field descriptions.

Compare `cpaResults.reviewed_applications` against `orderedCpa.hiring_goal` to show progress in your UI (e.g., "8 of 20 reviewed applications received").

## applicationUrl Behavior

<!-- theme: warning -->
> ### applicationUrl Is Overwritten for CPA+ Products
> When a CPA+ product is in the campaign, the `applicationUrl` you provide is replaced internally with a VONQ URL that points to the candidate journey-the AI screening flow candidates go through before their application reaches you. You still must provide a valid `applicationUrl` in the order request, but candidates will not be directed to it.

## Webhooks and Stats Updates

As candidates apply and are processed through the AI screening pipeline, HAPI updates the `cpaResults` counters on the campaign. Each time stats are updated, the campaign's `modifiedOn` timestamp changes.

If you have [campaign webhooks](./webhooks.md) enabled, these stats updates trigger a webhook notification. Use this as a signal to:
- Refresh your progress display (`reviewed_applications` vs `hiring_goal`)
- Poll for new applications via `GET /v3/cpacampaigns/{hapi_campaign_id}/applications/`

See [CPA+-Application Delivery](../09-cpa.md#application-delivery) for the full application retrieval flow.

## Restrictions

### Cannot Edit CPA+ Campaigns

Campaigns containing CPA+ products **cannot be edited**. The `isEditable` flag is always `false` for these campaigns. This applies to the entire campaign-even if other non-CPA+ products in the same campaign would normally support editing.

If you need to change vacancy details for a CPA+ campaign, cancel it and create a new one.

### Cannot Mix in Bundles with Editing

If a bundle contains a CPA+ product, the campaign is not editable after ordering-the same restriction applies.

## Edge Cases & Gotchas

- **CPA+ products are identified by a non-null `cpa` field** on the product object. Check this when building your UI to show appropriate progress tracking. See [Special Products](../05-products/03-special-products.md).
- **The campaign runs until the hiring goal is met or the campaign is stopped.** Unlike regular products with a fixed posting duration, CPA+ products stay active until all reviewed applications are delivered.
- **All applications are delivered**-both reviewed and not reviewed (for compliance). Filter with `?reviewed=true` in the polling endpoint to show only qualified candidates.

## Related

- [CPA+](../09-cpa.md)-full CPA+ guide: AI screening, application delivery, polling endpoints, file downloads
- [Webhooks](./webhooks.md)-receiving campaign update notifications (including stats changes)
- [Editing](./editing.md)-why CPA+ campaigns are not editable
- [Special Products](../05-products/03-special-products.md)-identifying CPA+ products in the catalog
- [Direct Apply-Webhooks](../10-direct-apply/webhooks.md)-CPA+ application delivery via webhooks
