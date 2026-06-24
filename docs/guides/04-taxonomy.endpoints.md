---
title: Taxonomy & Locations - Endpoint Reference
description: Full request/response details for job titles, industries, job functions, education levels, seniority, and location search endpoints.
category: guides/taxonomy
endpoints:
  - GET /products/job-titles/
  - GET /products/industries/
  - GET /products/job-functions/
  - GET /taxonomy/education-levels
  - GET /taxonomy/seniority
  - GET /products/location/search/
prerequisites: [authentication]
concepts: [taxonomy, job_title, industry, education_level, seniority, location]
related: [products-introduction, campaign-ordering]
audience: [developer]
difficulty: beginner
---

> For conceptual overview, see [Taxonomy & Locations](./04-taxonomy.md).

# Endpoints

### GET /products/job-titles/

- **Auth**: token | jwt
- **Description**: Search for job titles by text. Returns a paginated list of matching titles, ranked by relevance by default. Use the returned `id` as the `jobTitleId` parameter when searching for products.

**Query Parameters**:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `text` | string | yes | Search text for a job title name |
| `sortBy` | string | no | Ordering applied after matching the search text. Default: `relevance`. See sort options below. |
| `limit` | integer | no | Results per page (default: 10, max: 20) |
| `offset` | integer | no | Starting index for pagination (default: 0) |
| `Accept-Language` | header | no | Language code for localized names. See [Localization](../02-api-overview.md#localization). |

**Sort Options**:

| Value | Description |
|-------|-------------|
| `relevance` | Search relevance (default) |
| `frequency.desc` | Prefer more frequently used matching titles |
| `frequency.asc` | Prefer less frequently used matching titles |
| `name.asc` | Prefer matching titles by localized name, A to Z |
| `name.desc` | Prefer matching titles by localized name, Z to A |

For `name.asc` and `name.desc`, sorting uses the language resolved from `Accept-Language`, defaulting to the project default language. Clients should only send these supported sort values.

Search for job titles matching "manager":

```http
GET https://marketplace.api.vonq.com/products/job-titles/?text=manager HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
```

```json
{
  "count": 10,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 2,
      "name": "Project Manager",
      "job_function": {
        "id": 4,
        "name": "Management"
      }
    },
    {
      "id": 4,
      "name": "Business Development Manager",
      "job_function": {
        "id": 7,
        "name": "Sales"
      }
    },
    {
      "id": 9,
      "name": "Account Manager",
      "job_function": {
        "id": 7,
        "name": "Sales"
      }
    }
  ]
}
```

**Response fields**:

| Field | Type | Description |
|-------|------|-------------|
| `count` | integer | Total number of matching job titles |
| `next` | string \| null | URL for the next page of results |
| `previous` | string \| null | URL for the previous page of results |
| `results` | array | Array of job title objects |
| `results[].id` | integer | Job title identifier-pass as `jobTitleId` in [product search](./05-products/02-marketplace.md) |
| `results[].name` | string | Localized job title name |
| `results[].job_function.id` | integer | Parent job function identifier |
| `results[].job_function.name` | string | Localized job function name |

**Notes**:
- The `text` parameter is required-this is a search endpoint, not a listing endpoint. You cannot retrieve all job titles at once.
- Each job title belongs to exactly one job function. The `job_function` object is always included.
- HAPI supports ~4,000 job titles per language. Default results are ranked by relevance relative to the search query.

---

### GET /products/industries/

- **Auth**: token | jwt
- **Description**: List all supported industries, ordered alphabetically. Use the returned `id` when filtering products by industry and when specifying the `industry` in a campaign target group.

**Query Parameters**:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `limit` | integer | no | Results per page (default: 50, max: 100) |
| `offset` | integer | no | Starting index for pagination (default: 0) |
| `Accept-Language` | header | no | Language code for localized names. See [Localization](../02-api-overview.md#localization). |

List all industries:

```http
GET https://marketplace.api.vonq.com/products/industries/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
```

```json
[
  {
    "id": 48,
    "name": "Academic"
  },
  {
    "id": 6,
    "name": "Automotive industry"
  },
  {
    "id": 8,
    "name": "Technology"
  }
]
```

**Response fields**:

| Field | Type | Description |
|-------|------|-------------|
| `id` | integer | Industry identifier-pass as `industryId` in [product search](./05-products/02-marketplace.md), or as `vonqId` in campaign target group |
| `name` | string | Localized industry name |

---

### GET /products/job-functions/

- **Auth**: token | jwt
- **Description**: Retrieve the full job functions taxonomy as a tree structure. Each job function can contain child job functions. Use the returned `id` as the `jobFunctionId` parameter when searching for products. Not to be used in conjunction with `jobTitleId`.

**Query Parameters**:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `Accept-Language` | header | no | Language code for localized names. See [Localization](../02-api-overview.md#localization). |

List all job functions:

```http
GET https://marketplace.api.vonq.com/products/job-functions/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
```

```json
[
  {
    "id": 8,
    "name": "Education",
    "children": [
      {
        "id": 5,
        "name": "Teaching",
        "children": []
      }
    ]
  },
  {
    "id": 7,
    "name": "Sales",
    "children": [
      {
        "id": 11,
        "name": "Account Management",
        "children": []
      }
    ]
  }
]
```

**Response fields**:

| Field | Type | Description |
|-------|------|-------------|
| `id` | integer | Job function identifier-pass as `jobFunctionId` in [product search](./05-products/02-marketplace.md) |
| `name` | string | Localized job function name |
| `children` | array | Child job functions in the same format. Empty array if no children. |

**Notes**:
- Returns the complete tree in a single response-no pagination or search parameters.
- Each job title (from `GET /products/job-titles/`) belongs to exactly one job function. Use either `jobFunctionId` or `jobTitleId` when searching products-not both.

---

### GET /taxonomy/education-levels

- **Auth**: token | jwt
- **Description**: List all education levels. Returns the complete list with all translations in a multilingual array. Use the returned `id` as `vonqId` in the `educationLevel` field of a campaign target group.

```http
GET https://marketplace.api.vonq.com/taxonomy/education-levels HTTP/1.1
X-Auth-Token: <your Partner token here>
```

```json
[
  {
    "id": 1,
    "name": [
      { "languageCode": "en_GB", "value": "Master / Post-Graduate / PhD" },
      { "languageCode": "de_DE", "value": "Master / PhD / Doktor" },
      { "languageCode": "nl_NL", "value": "Master / Postdoctoraal" },
      { "languageCode": "fr_FR", "value": "Master / Postgraduate / Doctorat" },
      { "languageCode": "es_ES", "value": "Máster / Postgrado / Doctorado" },
      { "languageCode": "it_IT", "value": "Laurea Magistrale / Post-Laurea / Dottorato" }
    ]
  },
  {
    "id": 2,
    "name": [
      { "languageCode": "en_GB", "value": "Bachelor / Graduate" },
      { "languageCode": "de_DE", "value": "Bachelor / Absolvent" },
      { "languageCode": "nl_NL", "value": "HBO / Bachelor" }
    ]
  },
  {
    "id": 3,
    "name": [
      { "languageCode": "en_GB", "value": "Vocational / Diploma / Associates degree" },
      { "languageCode": "de_DE", "value": "Ausbildung" },
      { "languageCode": "nl_NL", "value": "MBO / Beroepssecundair onderwijs" }
    ]
  },
  {
    "id": 4,
    "name": [
      { "languageCode": "en_GB", "value": "GCSE / A-Level / Highschool / GED" },
      { "languageCode": "de_DE", "value": "Haupt-/ Realschulabschluss" },
      { "languageCode": "nl_NL", "value": "Middelbaar / Secundair onderwijs" }
    ]
  }
]
```

**Response fields**:

| Field | Type | Description |
|-------|------|-------------|
| `id` | integer | Education level identifier-use as `vonqId` in campaign [target group](./08-campaigns/ordering.md) |
| `name` | array | All available translations |
| `name[].languageCode` | string | Locale code (`en_GB`, `de_DE`, `nl_NL`, `fr_FR`, `es_ES`, `it_IT`) |
| `name[].value` | string | Translated name |

**Notes**:
- Returns the complete list in a single response-no pagination or search parameters.
- The `name` field is an array of translations, unlike job titles and industries which return a single localized string.

---

### GET /taxonomy/seniority

- **Auth**: token | jwt
- **Description**: List all seniority levels. Returns the complete list with all translations. Use the returned `id` as `vonqId` in the `seniority` field of a campaign target group.

```http
GET https://marketplace.api.vonq.com/taxonomy/seniority HTTP/1.1
X-Auth-Token: <your Partner token here>
```

```json
[
  {
    "id": 1,
    "name": [
      { "languageCode": "en_GB", "value": "Executive/Director" },
      { "languageCode": "de_DE", "value": "Geschäftsführer / C-Level" },
      { "languageCode": "nl_NL", "value": "Executive/Director" }
    ]
  },
  {
    "id": 2,
    "name": [
      { "languageCode": "en_GB", "value": "Manager" },
      { "languageCode": "de_DE", "value": "Manager" },
      { "languageCode": "nl_NL", "value": "Manager" }
    ]
  },
  {
    "id": 3,
    "name": [
      { "languageCode": "en_GB", "value": "Mid-Senior level" },
      { "languageCode": "de_DE", "value": "Professional" },
      { "languageCode": "nl_NL", "value": "Professional" }
    ]
  },
  {
    "id": 4,
    "name": [
      { "languageCode": "en_GB", "value": "Entry level/Graduate" },
      { "languageCode": "de_DE", "value": "Berufseinsteiger" },
      { "languageCode": "nl_NL", "value": "Entry level/Graduate" }
    ]
  },
  {
    "id": 5,
    "name": [
      { "languageCode": "en_GB", "value": "Student/Trainee" },
      { "languageCode": "de_DE", "value": "Student/Trainee" },
      { "languageCode": "nl_NL", "value": "Student/Trainee" }
    ]
  }
]
```

**Response fields**:

| Field | Type | Description |
|-------|------|-------------|
| `id` | integer | Seniority level identifier-use as `vonqId` in campaign [target group](./08-campaigns/ordering.md) |
| `name` | array | All available translations |
| `name[].languageCode` | string | Locale code |
| `name[].value` | string | Translated name |

**Notes**:
- Returns the complete list-no pagination or search parameters.
- Same multilingual array format as education levels.

### GET /products/location/search/

- **Auth**: token | jwt
- **Description**: Search for geographic locations by name. Returns a ranked list of matching locations with the full place hierarchy. Use the returned `id` when filtering products by geography (`includeLocationId`, `exactLocationId`) or when specifying a working location in a campaign.

**Query Parameters**:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `text` | string | yes | Location name to search for (supports partial matching) |
| `Accept-Language` | header | no | Language code for localized place names. See [Localization](../02-api-overview.md#localization). |

Search for locations matching "amsterdam":

```http
GET https://marketplace.api.vonq.com/products/location/search/?text=amsterdam HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: customer-123
```

```json
[
  {
    "id": 20174,
    "fully_qualified_place_name": "Amsterdam, North Holland, Netherlands",
    "canonical_name": "Amsterdam",
    "place_type": ["place"],
    "area": 312.0,
    "bounding_box": [4.728759, 52.318248, 5.093288241, 52.431227621],
    "within": {
      "id": 2364,
      "fully_qualified_place_name": "Netherlands",
      "canonical_name": "Netherlands",
      "place_type": ["country"],
      "bounding_box": [],
      "within": {
        "id": 2483,
        "fully_qualified_place_name": "Europe",
        "canonical_name": "Europe",
        "place_type": ["continent"],
        "bounding_box": [],
        "within": {
          "id": 2425,
          "fully_qualified_place_name": "Welt",
          "canonical_name": "International",
          "place_type": ["world"],
          "bounding_box": []
        }
      }
    }
  },
  {
    "id": 20177,
    "fully_qualified_place_name": "Amsterdam, New York, United States",
    "canonical_name": "Amsterdam",
    "place_type": ["place"],
    "bounding_box": [],
    "within": {
      "id": 2395,
      "fully_qualified_place_name": "United States",
      "canonical_name": "United States",
      "place_type": ["country"],
      "bounding_box": [],
      "within": {
        "id": 2484,
        "fully_qualified_place_name": "North America",
        "canonical_name": "North America",
        "place_type": ["continent"],
        "bounding_box": [],
        "within": {
          "id": 2425,
          "fully_qualified_place_name": "Welt",
          "canonical_name": "International",
          "place_type": ["world"],
          "bounding_box": []
        }
      }
    }
  }
]
```

**Response fields**:

| Field | Type | Description |
|-------|------|-------------|
| `id` | integer | Location identifier-use in [product search](./05-products/02-marketplace.md) filters and campaign working location |
| `fully_qualified_place_name` | string | Full name including parent hierarchy (e.g., `"Amsterdam, North Holland, Netherlands"`) |
| `canonical_name` | string \| null | Short name (e.g., `"Amsterdam"`) |
| `place_type` | array\<string\> | Classification: `place`, `district`, `region`, `country`, `continent`, or `world` |
| `area` | number \| absent | Area in km². Present on some entries (e.g., `312.0` for Amsterdam), absent on others. |
| `bounding_box` | array | Geographic bounding box as `[west, south, east, north]`. Empty array `[]` when not available. |
| `within` | object \| absent | Parent location in the hierarchy. The chain goes place → country → continent → world. Absent on the `world`-type entry at the root. |

**Notes**:
- Results are ranked by relevance. There is no pagination-the endpoint returns a small, relevance-ranked set.
- Minimum useful input is ~3 characters. Short queries may return broad results.
- Each location nests its full geographic context via the `within` field. The hierarchy goes up to 4 levels deep: place → country → continent → world. Region-level entries (e.g., "North Holland") may be skipped.
- The `bounding_box` and `area` fields are present when geographic data is available; `bounding_box` is an empty array and `area` is absent when not.
