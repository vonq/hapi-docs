---
id: machine-readable-resources
title: Machine-Readable Resources
description: Selective discovery with the documentation manifest, OpenAPI schema, workflow map, glossary, and llms.txt.
category: resources
endpoints: []
prerequisites: []
related:
- introduction
- api-overview
audience:
- developer
- ai-agent
difficulty: beginner
---

# Machine-Readable Resources

> Find the relevant guides and API contracts without preloading unrelated files.

## Quick Start for AI Agents

Clone [`vonq/hapi-docs`](https://github.com/vonq/hapi-docs), then work from the
repository root:

1. Read [`llms.txt`](../llms.txt) to choose a guide or scenario.
2. Query [`docs-manifest.jsonl`](../docs-manifest.jsonl) by document metadata
   or OpenAPI operation ID.
3. Search the selected area with `rg` and open only matching pages.
4. Select one operation or component from the OpenAPI schema with `jq`.

[`AGENTS.md`](../AGENTS.md) contains ready-to-run commands and the authority
order for resolving source differences.

## Available Resources

The published repository contains the canonical downloadable files. Clone it so
links stay relative and local search remains fast.

### OpenAPI Schema

The canonical API specification is
[`schema/build/public.json`](../schema/build/public.json). It defines endpoint
paths, methods, parameters, payloads, responses, and authentication.

Select one operation by `operationId`:

```bash
jq --arg id 'hapi-OrderCampaign' '
  .paths | to_entries[] as $path
  | $path.value | to_entries[]
  | select(.value.operationId? == $id)
  | {method: .key, path: $path.key, operation: .value}
' schema/build/public.json
```

Select one component schema:

```bash
jq '.components.schemas.HAPICampaignCreateRequest' schema/build/public.json
```

### Webhook Payload Schemas

Webhook models describe requests **HAPI sends to your system**. They are generated into the same [OpenAPI schema](../schema/build/public.json) as the REST API on every schema build.

| Model name | Outgoing request | Guide |
|------------|------------------|-------|
| `DirectApplyWebhookData` | Direct Apply or CPA+ application | [Direct Apply contract](guides/10-direct-apply/webhooks.endpoints.md#openapi-contract) |
| `DirectApplyWebhookFile` | Direct Apply or CPA+ Base64 file | [File delivery](guides/10-direct-apply/webhooks.endpoints.md#openapi-contract) |
| `ScreeningApplicationWebhookData` | Screening application result | [Screening contract](guides/11-screening/webhooks.md#openapi-contract) |
| `ScreeningApplicationWebhookFile` | Screening Base64 file | [Screening contract](guides/11-screening/webhooks.md#openapi-contract) |
| `ScreeningJobWebhookData` | Screening job event | [Job events](guides/11-screening/webhooks.md#job-events) |

On **Stoplight**, open the API reference and find the exact name above in its models. On **GitHub**, open the JSON file and search for that name under `components.schemas`, or extract it locally:

```bash
jq '.components.schemas.DirectApplyWebhookData' schema/build/public.json
```

These links use published relative file paths and ordinary Markdown headings so the guides work on both sites. A JSON pointer such as `#/components/schemas/DirectApplyWebhookData` identifies a model inside OpenAPI, but is not a GitHub page anchor. Use the model names above instead of relying on a renderer-specific deep link or a generated Stoplight URL.

The `DirectApplyApplication` callback on campaign ordering covers Direct Apply and CPA+. The `ScreeningApplication` and `ScreeningJob` callbacks on screening job creation cover application results and job events, including account-level destinations when no per-campaign or per-job URL override is supplied. Their request bodies describe JSON and, where applicable, multipart delivery. These are partner receivers, not additional HAPI endpoints to call.

### Supplementary Files

These files complement the OpenAPI schema:

| File | Format | Purpose |
|------|--------|---------|
| [`docs-manifest.jsonl`](../docs-manifest.jsonl) | JSON Lines | Document metadata and a compact index of OpenAPI operations. |
| [`llms.txt`](../llms.txt) | Markdown | Small router to guides, scenarios, workflow data, and optional reference pages. |
| [`docs/extra/api-map.yaml`](extra/api-map.yaml) | YAML | Call sequences, dependencies, state changes, and decision points. |
| [`docs/extra/glossary.yaml`](extra/glossary.yaml) | YAML | Domain terms, aliases, disambiguation, and related endpoints. |

## When to Use What

**Finding a guide or operation?**
Start with `llms.txt` for browsing. Query `docs-manifest.jsonl` when you know a
concept, keyword, document ID, or operation ID.

**Need to understand domain terminology?**
Search `glossary.yaml`. It maps terms such as "campaign" to aliases and related
endpoints.

**Building an HTTP request?**
Select the operation from `schema/build/public.json`. Then read its linked guide
for meaning and examples.

**Setting up AI-powered docs search or a chatbot?**
Index the manifest first. It contains document metadata and byte size without
documentation bodies.

## llms.txt Convention

`llms.txt` follows the [llms.txt proposal](https://llmstxt.org/)-a convention for making websites AI-accessible. Many AI tools and agents automatically look for this file at the root of a documentation site, similar to `robots.txt`.

If you host these docs, serve `llms.txt` at your docs root URL:

```
https://your-docs-domain/llms.txt
```

## Frontmatter Schema

Every documentation page includes structured YAML frontmatter:

```yaml
---
id: page-id
title: Page Title
description: One-line summary of what this page covers.
category: guides/section-name
endpoints:                    # API endpoints documented on this page
  - METHOD /resource/
prerequisites:                # Pages to read first
  - authentication
  - products-introduction
concepts:                     # Domain terms used (keys from glossary.yaml)
  - campaign
  - product
keywords:                     # Free-form search terms
  - ordering
related:                      # Related pages
  - campaign-ordering
  - campaign-status
audience: [developer, manager]
difficulty: beginner          # beginner | intermediate | advanced
---
```

This enables AI tools to filter pages by endpoint, find prerequisites, and navigate the concept graph.

## Keeping Resources Updated

When documentation or the public OpenAPI schema changes, regenerate the AI
resources:

```bash
./bin/docs/generate-ai-docs
```

The OpenAPI schema is generated from the Django codebase and published here as `schema/build/public.json`.

The generator writes `llms.txt` and `docs-manifest.jsonl`. The workflow map and
glossary remain maintained sources.
