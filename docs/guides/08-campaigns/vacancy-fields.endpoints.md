---
title: Vacancy Fields - Endpoint Reference
description: JSON payload examples for vacancy field sections used in campaign ordering and validation.
category: guides/campaigns
---

> For conceptual overview and field reference tables, see [Vacancy Fields](./vacancy-fields.md).

# Vacancy Fields - Endpoint Reference

Vacancy fields are submitted as part of `POST /campaigns/order`, `POST /campaigns/validate-vacancy-info/`, and `POST /campaigns/validate-campaign/`. See [Campaign Ordering - Endpoint Reference](./ordering.endpoints.md) and [Campaign Validation - Endpoint Reference](./validation.endpoints.md) for full endpoint examples.

## postingDetails Example

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

## targetGroup Example

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

## recruiterInfo Example

```json
{
  "recruiterInfo": {
    "id": "recruiter-42",
    "name": "Jane Recruiter",
    "emailAddress": "jane@acme.example.com"
  }
}
```

## companyId Example

```json
{
  "companyId": "customer-123"
}
```
