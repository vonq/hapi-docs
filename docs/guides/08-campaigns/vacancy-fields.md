---
title: Vacancy Fields
description: Job-level fields-recruiterInfo, postingDetails, targetGroup. Field reference with types and constraints.
category: guides/campaigns
endpoints: []
prerequisites: [campaigns-introduction, taxonomy]
concepts: [vacancy_fields, recruiterInfo, postingDetails, targetGroup, workingLocation, salaryIndication]
related: [campaign-ordering, taxonomy]
audience: [developer]
difficulty: intermediate
---

# Vacancy Fields
> Job-level details that describe the position-title, description, location, salary, and target audience. These fields end up on the job board.

## Overview

When ordering a campaign in HAPI, the request body contains several groups of fields. **Vacancy fields** are the ones that describe the job itself-the information that appears on the job board listing. They are separate from [posting requirements](../07-posting-requirements/01-introduction.md) (channel-specific fields) and product-specific configuration.

Vacancy fields are grouped into four sections:

| Section | Purpose |
|---------|---------|
| `postingDetails` | The job listing content-title, description, location, salary, employment type |
| `targetGroup` | Taxonomy classifications-education level, seniority, industry, job category |
| `recruiterInfo` | Contact details for the recruiter managing this vacancy |
| `companyId` | The company posting the job (same as your customer ID) |

See [Vacancy Fields - Endpoint Reference](./vacancy-fields.endpoints.md) for full JSON payload examples for each section.

## postingDetails

The `postingDetails` object contains the core job listing content.

```json
{
  "postingDetails": {
    "title": "Senior Software Engineer",
    "description": "<p>We are looking for a senior engineer to join our platform team...</p>",
    "organization": {
      "name": "Acme Corp",
      "companyLogo": "https://example.com/logo.png"
    },
    "workingLocation": {
      "addressLine1": "Keizersgracht 100",
      "postcode": "1015 AA",
      "city": "Amsterdam",
      "country": "NL",
      "allowsRemoteWork": true
    },
    "contactInfo": {
      "name": "Jane Recruiter",
      "emailAddress": "jane@acme.example.com",
      "phoneNumber": "+31 20 1234567"
    },
    "yearsOfExperience": 5,
    "employmentType": "permanent",
    "weeklyWorkingHours": {
      "from": 32,
      "to": 40
    },
    "salaryIndication": {
      "period": "yearly",
      "range": {
        "from": 65000,
        "to": 85000,
        "currency": "EUR"
      }
    },
    "jobPageUrl": "https://acme.example.com/careers/senior-engineer",
    "applicationUrl": "https://acme.example.com/apply/senior-engineer"
  }
}
```

### Field Reference

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | Yes | Job title |
| `description` | string | Yes | Job description (HTML allowed-see [HTML in descriptions](#html-in-descriptions)) |
| `organization.name` | string | Yes | Company name |
| `organization.companyLogo` | string | Yes | URL to the company logo |
| `workingLocation` | object | Yes | Job location (see [Working Location](#working-location)) |
| `contactInfo` | object | No | Recruiter contact shown on the listing (see [Contact Info](#contact-info)) |
| `yearsOfExperience` | integer | Loose | Required years of experience. Optional with `?loose=true`. |
| `employmentType` | string | Yes | Employment type (see [Employment Types](#employment-types)) |
| `weeklyWorkingHours` | object | Yes* | Working hours range: `from` and `to` (integers, 1–168) |
| `weeklyWorkingMinutes` | object | Yes* | Working minutes range: `from` and `to` (integers, 60–10080) |
| `salaryIndication` | object | No | Salary range (see [Salary Indication](#salary-indication)) |
| `jobPageUrl` | string | Yes | URL to the job listing on your site |
| `applicationUrl` | string | Yes | URL where candidates apply |

\* Exactly one of `weeklyWorkingHours` or `weeklyWorkingMinutes` must be provided. Use `weeklyWorkingMinutes` when you need fractional hours (e.g., 3 hours 30 minutes = 210 minutes).

<!-- theme: info -->
> ### Numeric Fields Accept Both Types
> Numeric fields (`yearsOfExperience`, `weeklyWorkingHours.from/to`, `salaryIndication.range.from/to`) accept both integers and strings. The API coerces string values to numbers internally. Both `"5"` and `5` are valid.

### Working Location

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `addressLine1` | string | Loose | Street address. Optional with `?loose=true`. |
| `addressLine2` | string | No | Secondary address line |
| `postcode` | string | Yes | Postal code |
| `city` | string | Yes | City name |
| `country` | string | Yes | Country name or ISO code |
| `allowsRemoteWork` | boolean | No | Whether the position allows remote work |

<!-- theme: info -->
> The API response also returns a `postCode` field (camelCase) as a legacy duplicate of `postcode`. Use `postcode` when sending requests - both are returned in responses.

### Contact Info

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | No | Contact person name |
| `emailAddress` | string | No | Contact email address |
| `phoneNumber` | string | No | Contact phone number |

### Employment Types

| Value | Description |
|-------|-------------|
| `permanent` | Permanent / indefinite contract |
| `temporary` | Temporary position |
| `fixed_term` | Fixed-term contract |
| `fixed_term_with_option_for_permanent` | Fixed-term with option for permanent |
| `freelance` | Freelance / independent contractor |
| `traineeship` | Traineeship |
| `internship` | Internship |

### Salary Indication

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `period` | string | Yes | `hourly`, `monthly`, or `yearly` |
| `range.from` | number | Yes | Minimum salary |
| `range.to` | number | Yes | Maximum salary |
| `range.currency` | string | No | ISO currency code (e.g., `EUR`, `USD`) |

### HTML in Descriptions

The `description` field accepts HTML. However:

- **Not all tags are supported.** The content is sanitized by HAPI and may be further sanitized by the job board.
- **Rendering varies by job board.** Each job board has its own HTML rendering implementation. The same description may look different across boards-some strip certain tags, others apply their own styling.
- **Stick to basic formatting.** Use `<p>`, `<ul>`, `<ol>`, `<li>`, `<strong>`, `<em>`, `<br>`, and `<h2>`/`<h3>` for best results across boards.

<!-- theme: warning -->
> ### Description Rendering Is Not Consistent
> Do not rely on pixel-perfect HTML rendering. Each job board processes the description differently. Test with simple, semantic HTML and expect visual differences across channels.

## targetGroup

The `targetGroup` object classifies the job using [taxonomy](../04-taxonomy.md) values. These help VONQ optimize distribution and are used by job boards for categorization.

Each field is an array containing exactly **one** taxonomy reference object with a `vonqId` (the taxonomy ID as a string) and a `description` (a human-readable label - this is a freeform string for display purposes; only `vonqId` is used for matching).

```json
{
  "targetGroup": {
    "educationLevel": [
      { "vonqId": "2", "description": "Bachelor / Graduate" }
    ],
    "seniority": [
      { "vonqId": "3", "description": "Mid-Senior level" }
    ],
    "industry": [
      { "vonqId": "48", "description": "Academic" }
    ],
    "jobCategory": [
      { "vonqId": "11", "description": "Customer Service" }
    ]
  }
}
```

| Field | Required | Taxonomy Endpoint | Description |
|-------|----------|-------------------|-------------|
| `educationLevel` | Loose | `GET /taxonomy/education-levels` | Minimum education level |
| `seniority` | Loose | `GET /taxonomy/seniority` | Career experience level |
| `industry` | Loose | `GET /products/industries/` | Business sector |
| `jobCategory` | Loose | Not resolved through taxonomy endpoints | Compatibility field |

All four fields are required by default but may be omitted with `?loose=true` when listed in your loose-validation settings.

<!-- theme: warning -->
> ### One Value Per Dimension
> Each taxonomy field accepts exactly one value. Pass an array with a single object-not multiple.

To look up available taxonomy values and their IDs, see [Taxonomy](../04-taxonomy.md).

## recruiterInfo

The `recruiterInfo` object identifies the recruiter managing this vacancy.

```json
{
  "recruiterInfo": {
    "id": "recruiter-42",
    "name": "Jane Recruiter",
    "emailAddress": "jane@acme.example.com"
  }
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Recruiter identifier (your internal ID) |
| `name` | string | Yes | Recruiter full name |
| `emailAddress` | string | Yes | Recruiter email address |

## companyId

The `companyId` identifies the company posting the job. This is the same as your customer ID (`X-Customer-Id`). It exists as a separate field for backwards compatibility.

```json
{
  "companyId": "customer-123"
}
```

## Loose Validation

By default, all fields marked "Loose" in the tables above are required. Adding `?loose=true` to the ordering endpoint allows configured fields to be omitted, so you can submit a campaign before all vacancy details are available.

Possible fields for loose validation:

- `postingDetails.yearsOfExperience`
- `postingDetails.workingLocation.addressLine1`
- `targetGroup.educationLevel`
- `targetGroup.seniority`
- `targetGroup.industry`
- `targetGroup.jobCategory`

<!-- theme: info -->
> ### Loose Validation Requires Account Configuration
> Your account must be enabled for loose validation. Use `GET /v3/ats/atsuser/me/settings/` to read `settings.campaigns.loose_validation`: `marketplace.fields` applies to Marketplace orders, `job_post.fields` applies to Job Post orders, and mixed campaigns use the union of both.

See [Ordering](./ordering.md) for the full ordering request and query parameters.

## Edge Cases & Gotchas

<!-- theme: warning -->
> ### applicationUrl and CPA+ Campaigns
> The `applicationUrl` field is always required, but for CPA+ campaigns, it is mutated internally-candidates apply through the CPA+ flow, not your application URL directly.

<!-- theme: warning -->
> ### Hours vs Minutes
> Provide exactly one of `weeklyWorkingHours` or `weeklyWorkingMinutes`. If you provide both, the API rejects the request. Use minutes when you need precision below whole hours (e.g., 22.5 hours = 1350 minutes).

### Handling Optional Fields for JP Campaigns

When ordering JP (contract-based) campaigns, some vacancy fields are required by the API but may not be relevant to your use case. If you don't have real data for these fields, you can provide placeholder values. The following table shows common dummy values used by integrations:

| Field | Dummy Value | Notes |
|-------|------------|-------|
| `recruiterInfo.id` | `"0"` | Internal identifier-not shown on job boards |
| `postingDetails.workingLocation.addressLine2` | `" "` (space) | Optional field-use a single space if needed |
| `postingDetails.contactInfo.phoneNumber` | `"+0000000000"` | Not always displayed on the job board |
| `postingDetails.yearsOfExperience` | `0` | Use with `?loose=true` to omit entirely |

<!-- theme: warning -->
> ### Dummy Values Are a Workaround
> Using dummy values is not ideal-the actual content may appear on job boards. Use `?loose=true` where possible to omit fields entirely instead of submitting placeholder data. See [Loose Validation](#loose-validation).

- **Taxonomy values are multilingual**-the taxonomy endpoints return translations in multiple languages. Use the `description` that matches your locale when building the `targetGroup`. See [Taxonomy](../04-taxonomy.md).
- **`companyId` must match `X-Customer-Id`**-these refer to the same entity. The field exists for backwards compatibility.
- **Logo URL must be publicly accessible**-the `companyLogo` URL is fetched by job boards when rendering the listing. Use a stable, publicly reachable URL.

## Related

- [Taxonomy](../04-taxonomy.md)-look up education levels, seniority, industries, and job categories
- [Ordering](./ordering.md)-full campaign order request structure
- [Validation](./validation.md)-validate vacancy fields before ordering
- [Editing](./editing.md)-which vacancy fields can be edited after ordering
- [Posting Requirements](../07-posting-requirements/01-introduction.md)-channel-specific fields (separate from vacancy fields)
