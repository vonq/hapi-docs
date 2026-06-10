---
title: Scenarios-Introduction
description: End-to-end walkthroughs showing how to accomplish common tasks with the VONQ Hiring API.
category: guides/scenarios
endpoints: []
prerequisites: [authentication]
concepts: []
related: []
audience: [developer]
difficulty: beginner
---

# Scenarios

> End-to-end walkthroughs that show how the API pieces fit together in practice.

## What Are Scenarios?

The guide pages explain individual concepts-contracts, posting requirements, campaigns, webhooks. Scenarios show how those concepts combine into real workflows, step by step.

Each scenario walks you through a specific goal from start to finish: which endpoints to call, what to look for in the responses, and what to watch out for. They are not exhaustive API references-they link back to the relevant guide pages for full details.

## Who Are These For?

Scenarios are most useful when you:

- Are integrating HAPI for the first time and want to see the big picture
- Understand the individual concepts but need to see how they connect
- Want a checklist to follow when building a specific feature

## Available Scenarios

| Scenario | What it covers |
|----------|---------------|
| [Setting Up a Contract](./contract-setup.md) | Find a channel, check credentials, handle OAuth, create a contract |
| [Job Marketing Campaign](./job-marketing-ordering.md) | Search products, fill vacancy fields, validate, order, poll status |
| [Job Post Campaign](./job-post-campaign.md) | Use a contract, collect posting requirements, validate per-channel, order |
| [Mixed Campaign](./mixed-campaign.md) | JM + JP products in a single order - payload structure and constraints |
| [Screening](./screening.md) | Create a screening job, submit candidates, retrieve AI-scored dossiers |
