---
title: Campaign Validation - Endpoint Reference
description: HTTP request/response details for campaign validation endpoints.
category: guides/campaigns
---

> For conceptual overview, see [Campaign Validation](./validation.md).

# Campaign Validation - Endpoint Reference

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| POST | `/campaigns/validate-vacancy-info/` | Validate vacancy fields only |
| POST | `/campaigns/validate-campaign/` | Validate complete campaign payload |
| POST | `/campaigns/order?validateOnly=true` | Dry-run validation using the order endpoint |

### POST /campaigns/validate-vacancy-info/

- **Auth**: token + `X-Customer-Id` or JWT
- **Description**: Validates the vacancy portion of a campaign-posting details, target group, and recruiter info-independently of any products or posting requirements.

```http
POST https://marketplace.api.vonq.com/campaigns/validate-vacancy-info/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
Content-Type: application/json
```

```json
{
  "vacancy": {
    "companyId": "customer-123",
    "recruiterInfo": {
      "id": "recruiter-42",
      "name": "Jane Recruiter",
      "emailAddress": "jane@acme.example.com"
    },
    "campaignName": "Senior Engineer - Amsterdam Q1",
    "postingDetails": {
      "title": "Senior Software Engineer",
      "description": "<p>We are looking for a senior engineer...</p>",
      "organization": {
        "name": "Acme Corp",
        "companyLogo": "https://example.com/logo.png"
      },
      "workingLocation": {
        "addressLine1": "Keizersgracht 100",
        "postcode": "1015 AA",
        "city": "Amsterdam",
        "country": "NL"
      },
      "contactInfo": {
        "name": "Jane Recruiter",
        "emailAddress": "jane@acme.example.com"
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
    },
    "targetGroup": {
      "educationLevel": [{ "vonqId": "2", "description": "Bachelor / Graduate" }],
      "seniority": [{ "vonqId": "3", "description": "Mid-Senior level" }],
      "industry": [{ "vonqId": "48", "description": "Academic" }],
      "jobCategory": [{ "vonqId": "11", "description": "Customer Service" }]
    }
  }
}
```

For full field reference, see [Vacancy Fields](./vacancy-fields.md).

**Success Response (200)**

```json
{
  "errors": {},
  "has_errors": false
}
```

**Error Response (422)**

Errors are nested to match the request structure, so you can map each error directly to the field that caused it:

```json
{
  "errors": {
    "postingDetails": {
      "title": ["This field is required."],
      "organization": {
        "companyLogo": ["The remote file does not appear to exist."]
      }
    },
    "targetGroup": {
      "seniority": [{ "vonqId": ["VonqId 999 not found."] }]
    }
  },
  "has_errors": true
}
```

Each field's errors are returned as an array of strings. Nested objects preserve their nesting in the error structure.

**Errors**

| Status | Cause |
|--------|-------|
| 422 | One or more vacancy fields are invalid or missing |
| 401 | Invalid or missing authentication |

### POST /campaigns/validate-campaign/

- **Auth**: token + `X-Customer-Id` or JWT
- **Description**: Validates a complete campaign payload-vacancy fields, ordered products, and all per-product posting requirements-in a single call.

The request body wraps the full campaign in a `campaign` property.

```http
POST https://marketplace.api.vonq.com/campaigns/validate-campaign/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
Content-Type: application/json
```

```json
{
  "campaign": {
    "companyId": "customer-123",
    "currency": "EUR",
    "recruiterInfo": {
      "id": "recruiter-42",
      "name": "Jane Recruiter",
      "emailAddress": "jane@acme.example.com"
    },
    "postingDetails": {
      "title": "Senior Software Engineer",
      "description": "<p>We are looking for a senior engineer...</p>",
      "organization": {
        "name": "Acme Corp",
        "companyLogo": "https://example.com/logo.png"
      },
      "workingLocation": {
        "addressLine1": "Keizersgracht 100",
        "postcode": "1015 AA",
        "city": "Amsterdam",
        "country": "NL"
      },
      "contactInfo": {
        "name": "Jane Recruiter",
        "emailAddress": "jane@acme.example.com"
      },
      "yearsOfExperience": 5,
      "employmentType": "permanent",
      "weeklyWorkingHours": { "from": 32, "to": 40 },
      "salaryIndication": {
        "period": "yearly",
        "range": { "from": 65000, "to": 85000, "currency": "EUR" }
      },
      "jobPageUrl": "https://acme.example.com/careers/senior-engineer",
      "applicationUrl": "https://acme.example.com/apply/senior-engineer"
    },
    "targetGroup": {
      "educationLevel": [{ "vonqId": "2", "description": "Bachelor / Graduate" }],
      "seniority": [{ "vonqId": "3", "description": "Mid-Senior level" }],
      "industry": [{ "vonqId": "48", "description": "Academic" }],
      "jobCategory": [{ "vonqId": "11", "description": "Customer Service" }]
    },
    "orderedProducts": [
      "d2e3f4a5-b6c7-8901-bcde-f12345678901"
    ],
    "orderedProductsSpecs": [
      {
        "productId": "d2e3f4a5-b6c7-8901-bcde-f12345678901",
        "contractId": "c4d5e6f7-a8b9-0123-def0-234567890123",
        "postingRequirements": {
          "titre": "Ingénieur Senior",
          "IGB_country": "FR"
        }
      }
    ]
  }
}
```

**Success Response (200)**

```json
{
  "errors": {
    "orderedProductsSpecs": [{}]
  },
  "has_errors": false
}
```

On success, the `orderedProductsSpecs` array contains an empty object for each product-one per entry in your request.

**Error Response (422)**

The error response combines vacancy-level errors and per-product errors. The `orderedProductsSpecs` errors are **positional**-the first object corresponds to the first product in your request, and so on.

```json
{
  "errors": {
    "postingDetails": {
      "title": ["This field is required."]
    },
    "currency": ["This field is required."],
    "orderedProductsSpecs": [
      {
        "credentials": {},
        "posting_requirements": {
          "titre": "The field \"Titre\" must have a value."
        }
      }
    ]
  },
  "has_errors": true
}
```

| Error Section | Contains |
|---------------|----------|
| Top-level fields (`postingDetails`, `targetGroup`, `currency`, etc.) | Vacancy field errors-same structure as `validate-vacancy-info` |
| `orderedProductsSpecs[n].credentials` | Missing or invalid contract credentials for this product |
| `orderedProductsSpecs[n].posting_requirements` | Invalid or missing posting requirement values for this product |

**Notes**
- The `postingRequirements` field in the request uses a flat key-value object (e.g., `{ "titre": "..." }`), not the `{ name, value }` array format used by `validate-channel-posting`.
- Credential errors indicate that the contract for a product is missing required credentials (e.g., API keys for the job board). See [Contracts-Notes](../06-contracts/notes.md).

**Errors**

| Status | Cause |
|--------|-------|
| 422 | One or more fields are invalid or missing |
| 401 | Invalid or missing authentication |

### POST /campaigns/order?validateOnly=true

- **Auth**: token + `X-Customer-Id` or JWT
- **Description**: Performs a dry-run validation of the full campaign order without actually creating it.

```http
POST https://marketplace.api.vonq.com/campaigns/order?validateOnly=true HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
Content-Type: application/json
```

The request body is identical to a real campaign order (see [Ordering](./ordering.md)). The error response format matches `validate-campaign`.
