# VONQ Hiring API (HAPI) - Public Documentation

> Single-API access to 2000+ job boards, social platforms, and niche channels - so ATS partners can offer job advertising as a native feature without building individual integrations.

This repository hosts the public documentation for the **VONQ Hiring API (HAPI)**, including written guides, the OpenAPI schema, and machine-readable resources for AI agents and code generators.

The rendered docs are available on [Stoplight](https://vonq.stoplight.io/docs/hapi-docs-v2/e9ee9e37358b5-hapi). This repository is the underlying source.

## Quick Start

**New to HAPI?** Start with [Introduction](docs/01-introduction.md), then [API Overview](docs/02-api-overview.md).

**Building an integration?** Jump to the [OpenAPI schema](schema/build/public.json) and the [guides](docs/guides/).

**AI agent or code generator?** See [Machine-Readable Resources](docs/13-machine-readable-resources.md), [`llms.txt`](llms.txt), and [`llms-full.txt`](llms-full.txt).

## Table of Contents

The high-level layout:

- **Getting Started**
  - [Introduction](docs/01-introduction.md) - what HAPI is, credentials, first call
  - [API Overview](docs/02-api-overview.md) - environments, headers, pagination, errors, rate limits
- **Guides**
  - [Authentication & Users](docs/guides/03-authentication-and-users/01-introduction.md) - secret keys, JWT, ATS/ATSUser/Company entities
  - [Taxonomy](docs/guides/04-taxonomy.md) - job functions, titles, education, seniority, industries, locations
  - [Products](docs/guides/05-products/01-introduction.md) - marketplace, special products, posting requirements
  - [Contracts](docs/guides/06-contracts/01-introduction.md) - managing contracts, ordering, contract groups
  - [Posting Requirements](docs/guides/07-posting-requirements/01-introduction.md) - facets, autocomplete, validation, smartfill
  - [Campaigns](docs/guides/08-campaigns/01-introduction.md) - vacancy fields, ordering, status, editing, webhooks, bundles
  - [CPA+](docs/guides/09-cpa.md) - cost-per-application campaigns, applications, attachments
  - [Direct Apply](docs/guides/10-direct-apply/01-introduction.md) - questionnaires, application webhooks, feedback
  - [Screening](docs/guides/11-screening/01-introduction.md) - AI screening, jobs, applications, dossiers
  - [Wallets & Payments](docs/guides/12-wallets-and-payments.md) - prepaid wallets, top-ups, billing portal
  - [Scenarios](docs/guides/14-scenarios/01-introduction.md) - end-to-end integration walkthroughs
- **Reference**
  - [Machine-Readable Resources](docs/13-machine-readable-resources.md) - OpenAPI, glossary, API map, llms.txt
  - [API Schema](schema/build/public.json) - OpenAPI 3.0.3 spec

## Repository Layout

```text
.
├── docs/                      # Markdown documentation (guides, scenarios, references)
│   ├── 01-introduction.md
│   ├── 02-api-overview.md
│   ├── 13-machine-readable-resources.md
│   ├── guides/                # Per-domain guides (auth, products, campaigns, ...)
│   └── extra/                 # Machine-readable resources for AI/tooling
│       ├── api-map.yaml       # Workflow topology and call sequences
│       └── glossary.yaml      # Domain terms with aliases and disambiguation
├── schema/
│   └── build/
│       └── public.json        # OpenAPI 3.0.3 spec (source of truth for endpoints)
├── llms.txt                   # LLM-friendly index for automatic AI discovery
├── llms-full.txt              # Generated docs/ concatenation for direct ingestion
├── CHANGELOG.md               # Notable changes per release
└── README.md                  # This file
```

## OpenAPI Schema

The canonical API specification lives at [`schema/build/public.json`](schema/build/public.json):

| Property | Value |
|----------|-------|
| Format | OpenAPI 3.0.3 (JSON) |
| Auth schemes | `Partner-APIKey`, `ATSUser-APIKey`, `ATSUser-JWT` |

Use it directly with any OpenAPI-aware tool (Postman, Insomnia, openapi-generator, swagger-codegen, etc.) or browse the rendered version on [Stoplight](https://vonq.stoplight.io/docs/hapi-docs-v2/e9ee9e37358b5-hapi).

## For AI Agents

Load these in order to build an integration:

1. [`schema/build/public.json`](schema/build/public.json) - exact endpoints, parameters, request/response schemas, auth.
2. [`docs/extra/api-map.yaml`](docs/extra/api-map.yaml) - workflow sequencing: what to call, in what order, with what dependencies.
3. [`docs/extra/glossary.yaml`](docs/extra/glossary.yaml) - domain terms, aliases, disambiguation.
4. [`llms.txt`](llms.txt) - concise structured index for orientation.
5. [`llms-full.txt`](llms-full.txt) - generated `docs/` content in a single file for context-window ingestion.

Every documentation page also ships YAML frontmatter (`title`, `description`, `endpoints`, `prerequisites`, `concepts`, `related`, `audience`, `difficulty`) for semantic filtering and retrieval.

## Environments

| Environment | Base URL | Purpose |
|-------------|----------|---------|
| **Production** | `https://marketplace.api.vonq.com` | Live; orders are processed and charged. |
| **Sandbox** | `https://marketplace-sandbox.api.vonq.com` | Testing; orders accepted but not executed or charged. |

Credentials are environment-specific. Contact your VONQ account manager to receive Sandbox credentials.

## Other Integration Paths

These docs cover the **REST API**. If you want a faster integration with embeddable UI, see [HAPI Elements](https://docs.elements.hapi.vonq.com/) - production-ready, framework-agnostic JavaScript components for product search, contract management, and campaign ordering.

## Contributing

This repository is **read-only**. It is published from the internal source of truth at VONQ; do not open pull requests against it directly.

If you spot an error, an unclear passage, or want to request additional documentation, please:

- Email your VONQ account manager, or
- Open an issue in this repository describing the problem and the page (or endpoint) it relates to.

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for notable changes.

## License & Support

These docs are published by VONQ for the use of HAPI integration partners. For technical support, integration questions, or credential issues, contact your VONQ account manager.

## Links

- [Rendered docs on Stoplight](https://vonq.stoplight.io/docs/hapi-docs-v2/e9ee9e37358b5-hapi)
- [HAPI Elements docs](https://docs.elements.hapi.vonq.com/)
- [VONQ website](https://www.vonq.com/)
