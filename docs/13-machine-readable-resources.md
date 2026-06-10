---
title: Machine-Readable Resources
description: OpenAPI schema, workflow maps, glossary, and full documentation dumps-optimized for AI agents and programmatic consumption.
category: resources
endpoints: []
prerequisites: []
concepts: []
related: [introduction, api-overview]
audience: [developer, ai-agent]
difficulty: beginner
---

# Machine-Readable Resources

> Structured files designed for AI agents, code generators, and tooling-not just human readers.

## Quick Start for AI Agents

If you are an AI agent building a HAPI integration, load these files in order:

1. **OpenAPI schema** (`schema/build/public.json`) - exact endpoints, parameters, request/response schemas, and auth requirements. This is your primary reference for constructing HTTP requests.
2. **`api-map.yaml`** - workflow sequencing: what to call, in what order, and what depends on what. The OpenAPI spec tells you *how* to call each endpoint; this file tells you *when* and *why*.
3. **`glossary.yaml`** - domain terms, aliases, and disambiguation. Use this when you encounter unfamiliar terms like "MOC", "facet", or "campaign".
4. **`llms.txt`** - concise index with concepts, endpoint table, and doc links. Good for orientation if you need narrative context beyond the schema.

## Available Resources

### OpenAPI Schema

The canonical API specification, located at `schema/build/public.json`:

| Property | Value |
|----------|-------|
| Format | OpenAPI 3.0.3 (JSON) |
| Paths | 60 |
| Component schemas | 236 |
| Auth schemes | 3 (`Partner-APIKey`, `ATSUser-APIKey`, `ATSUser-JWT`) |
| Size | ~394 KB |

This is the single source of truth for endpoint paths, HTTP methods, query/path parameters, request bodies, response shapes, and security requirements.

### Supplementary Files

These files complement the OpenAPI schema with information it cannot express. `api-map.yaml` and `glossary.yaml` live under `docs/extra/`; `llms.txt` and `llms-full.txt` live at the documentation root so AI tools can discover them directly.

| File | Format | Purpose | Size |
|------|--------|---------|------|
| `docs/extra/api-map.yaml` | YAML | Workflow topology-step-by-step integration flows, endpoint dependencies, state machines, and common pitfalls. | ~32 KB |
| `docs/extra/glossary.yaml` | YAML | Domain terms with definitions, aliases, disambiguation, and related endpoints. | ~13 KB |
| `llms.txt` | Markdown | Structured index of the entire API-concepts, endpoint table, section links. Published at the docs root for automatic AI discovery. | ~8 KB |
| `llms-full.txt` | Markdown | Complete `docs/` documentation concatenated into a single root file for context-window ingestion. | ~405 KB |

## When to Use What

**Building an integration or coding against the API?**
Start with the **OpenAPI schema**-it has everything you need to construct valid requests. Use `api-map.yaml` to understand the correct call sequence and dependencies between endpoints.

**Need to understand domain terminology?**
Load `glossary.yaml`. It maps terms like "campaign" to their aliases ("order", "posting order"), disambiguates similar concepts, and links to relevant endpoints.

**Feeding docs into an LLM context window?**
Use `llms-full.txt` for comprehensive coverage, or `llms.txt` for a concise index that fits in smaller contexts.

**Setting up AI-powered docs search or a chatbot?**
Each documentation page includes YAML frontmatter with `title`, `description`, `endpoints`, `prerequisites`, `concepts`, `related`, `audience`, and `difficulty`-enabling semantic filtering and retrieval.

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
related:                      # Related pages
  - campaign-ordering
  - campaign-status
audience: [developer, manager]
difficulty: beginner          # beginner | intermediate | advanced
---
```

This enables AI tools to filter pages by endpoint, find prerequisites, and navigate the concept graph.

## Keeping Resources Updated

When documentation changes, regenerate `llms-full.txt`:

```bash
bash bin/docs/generate-llms-full.sh
```

The OpenAPI schema is generated from the Django codebase and published here as `schema/build/public.json`.

The other files (`llms.txt`, `docs/extra/api-map.yaml`, `docs/extra/glossary.yaml`) are manually maintained and should be updated when new endpoints, workflows, or concepts are added.
