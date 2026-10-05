---
id: cpa-endpoints
title: CPA+ - Endpoint Reference
description: Full request/response details for CPA+ polling endpoints-list applications, retrieve an application, download attachments.
category: guides/cpa
related:
- cpa
endpoints:
- GET /v3/cpacampaigns/{hapi_campaign_id}/applications/
- GET /v3/cpacampaigns/{hapi_campaign_id}/applications/{application_id}/
- GET /v3/cpacampaigns/{hapi_campaign_id}/applications/{application_id}/attachment/
---

> For conceptual overview, see [CPA+](./09-cpa.md).

## Endpoints

### GET /v3/cpacampaigns/{hapi_campaign_id}/applications/

- **Auth**: token (`X-Auth-Token` + `X-Customer-Id`) or JWT
- **Description**: List all applications for a CPA+ campaign.

**Path Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| `campaignId` | UUID | The HAPI campaign ID that contains the CPA+ product |

**Query Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| `reviewed` | boolean | Filter by review status-`true` for AI-reviewed, `false` for not reviewed |

```http
GET https://marketplace.api.vonq.com/v3/cpacampaigns/f1a2b3c4-5d6e-7f8a-9b0c-1d2e3f4a5b6c/applications/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
```

**Response (200):**

```json
{
  "count": 2,
  "results": [
    {
      "id": "1001",
      "reviewed": true,
      "name": "Jane Doe",
      "email": "jane.doe@example.com",
      "phone": "+31 6 12345678",
      "files": [
        {"type": "DOSSIER", "mimeType": "application/pdf", "filename": "dossier.pdf"},
        {"type": "RESUME", "mimeType": "application/pdf", "filename": "resume.pdf"}
      ],
      "updated": "2026-03-10T14:30:00Z",
      "experiences": [
        {
          "organizationName": "Example Logistics",
          "title": "Backend Developer",
          "location": "Amsterdam, Netherlands",
          "dateStart": "2021-03-01T00:00:00+00:00",
          "currentlyWorking": true,
          "description": "Builds and maintains Python services."
        }
      ],
      "educations": [
        {
          "instituteName": "Example University",
          "educationName": "Computer Science",
          "degree": "Master",
          "dateStart": "2015-09-01T00:00:00+00:00",
          "dateEnd": "2017-07-01T00:00:00+00:00"
        }
      ],
      "skills": [
        {"competenceName": "Python"}
      ],
      "certifications": [
        {
          "name": "Example Cloud Practitioner",
          "issuingAuthorityName": "Example Institute",
          "dateIssued": "2022-05-01T00:00:00+00:00"
        }
      ]
    },
    {
      "id": "1002",
      "reviewed": false,
      "name": "John Smith",
      "email": "john.smith@example.com",
      "phone": "+49 170 9876543",
      "files": [
        {"type": "DOSSIER", "mimeType": "application/pdf", "filename": "dossier.pdf"},
        {
          "type": "RESUME",
          "mimeType": "application/vnd.openxmlformats-officedocument.wordprocessingml.document",
          "filename": "resume.docx"
        }
      ],
      "updated": "2026-03-09T09:15:00Z",
      "experiences": [],
      "educations": [],
      "skills": [],
      "certifications": []
    }
  ]
}
```

The list is not paginated: the response contains only `count` and `results`.

**Response fields**

| Field | Type | Description |
|-------|------|-------------|
| `count` | integer | Total number of applications |
| `results` | array | List of application objects |
| `results[].id` | string | Application identifier |
| `results[].reviewed` | boolean | Whether the application was reviewed by AI |
| `results[].name` | string | Candidate's full name |
| `results[].email` | string | Candidate's email address |
| `results[].phone` | string | Candidate's phone number |
| `results[].files` | array | Downloadable files, each an object with `type` (`DOSSIER` or `RESUME`), `mimeType` and `filename` |
| `results[].updated` | datetime | When the application was last updated |
| `results[].experiences` | array | Work experience from the candidate's profile |
| `results[].educations` | array | Education from the candidate's profile |
| `results[].skills` | array | Skills from the candidate's profile |
| `results[].certifications` | array | Certifications from the candidate's profile |

**Profile arrays**

`experiences`, `educations`, `skills` and `certifications` use the same item fields as the Direct Apply payload. Every item field is optional except `competenceName` on skills, and dates are ISO 8601 strings. The table lists the common fields; the API schema lists all of them.

| Array | Common item fields |
|-------|-------------|
| `experiences` | `title`, `organizationName`, `location`, `description`, `dateStart`, `dateEnd`, `currentlyWorking`, `level`, `jobCategories`, `industries` |
| `educations` | `instituteName`, `educationName`, `degree`, `level`, `location`, `dateStart`, `dateEnd`, `currentlyEnrolled`, `status` |
| `skills` | `competenceName`, `proficiencyLevel` |
| `certifications` | `name`, `issuingAuthorityName`, `description`, `dateStarted`, `dateIssued`, `code` |

<!-- theme: info -->
> ### Profile arrays can be empty
> On the list endpoint the profile arrays can be `[]` until the candidate's profile data has been populated, which happens in the background after the first request. An empty array can also mean the candidate has no data of that kind. Request the list again later, or retrieve the application, which loads the profile data if needed.

Filter to only reviewed applications:

```http
GET https://marketplace.api.vonq.com/v3/cpacampaigns/f1a2b3c4-.../applications/?reviewed=true HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
```

---

### GET /v3/cpacampaigns/{hapi_campaign_id}/applications/{application_id}/

- **Auth**: token (`X-Auth-Token` + `X-Customer-Id`) or JWT
- **Description**: Retrieve details for a specific application.

```http
GET https://marketplace.api.vonq.com/v3/cpacampaigns/f1a2b3c4-.../applications/app-001/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
```

Returns a single application object with the same fields as a list item, including `files` and the four profile arrays (`experiences`, `educations`, `skills`, `certifications`).

**Errors**

| Status | Cause |
|--------|-------|
| `403` | JWT doesn't match the campaign's ATS user |
| `404` | Campaign or application not found |

---

### GET /v3/cpacampaigns/{hapi_campaign_id}/applications/{application_id}/attachment/

- **Auth**: token (`X-Auth-Token` + `X-Customer-Id`) or JWT
- **Description**: Get a temporary download URL for a file attached to an application.

**Query Parameters**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `filename` | string | Yes | The file to download (must match a `filename` from the application's `files` array) |

**Available files**

| Filename | Description |
|----------|-------------|
| `dossier.pdf` | AI screening report-interview transcript, evaluation scores, skills-vs-requirements summary |
| `resume.pdf` | Candidate's resume in PDF format |
| `resume.docx` | Candidate's resume in Word format |

```http
GET https://marketplace.api.vonq.com/v3/cpacampaigns/f1a2b3c4-.../applications/app-001/attachment?filename=dossier.pdf HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
```

**Response (200):**

```json
{
  "filename": "dossier.pdf",
  "url": "https://storage.vonq.com/temp/abc123-dossier.pdf?token=xyz",
  "generated_at": "2026-03-10T15:00:00Z"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `filename` | string | The requested filename |
| `url` | string | Temporary signed download URL |
| `generated_at` | datetime | When the file was generated |

<!-- theme: warning -->
> ### Download URLs Expire
> The `url` is a temporary signed link. Download the file promptly-do not store the URL for later use. Request a new URL from the endpoint when you need to download again.

**Errors**

| Status | Cause |
|--------|-------|
| `400` | Invalid or unsupported filename |
| `403` | JWT doesn't match the campaign's ATS user |
| `404` | Campaign or application not found |
