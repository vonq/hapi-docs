---
id: screening-jobs-and-applications-endpoints
title: Screening-Jobs & Applications - Endpoint Reference
description: Full request/response details for all screening job and application endpoints.
category: guides/screening
related:
- screening-jobs-and-applications
endpoints:
- POST /v3/screening/jobs/
- GET /v3/screening/jobs/
- GET /v3/screening/jobs/{id}/
- DELETE /v3/screening/jobs/{id}/
- POST /v3/screening/jobs/{job_id}/applications/
- GET /v3/screening/jobs/{job_id}/applications/
- GET /v3/screening/jobs/{job_id}/applications/{id}/
- DELETE /v3/screening/jobs/{job_id}/applications/{id}/
- GET /v3/screening/jobs/{job_id}/applications/{id}/attachments/
- GET /v3/screening/jobs/{job_id}/applications/{id}/attachments/{file_type}/
---

> For conceptual overview, see [Screening-Jobs & Applications](./jobs-and-applications.md).

For results pushed to your system, see [Screening webhooks](./webhooks.md#openapi-contract). The outgoing request models **`ScreeningApplicationWebhookData`** and **`ScreeningJobWebhookData`** are included in the [public OpenAPI schema](../../../schema/build/public.json); they differ from the API responses documented below.

## Endpoints

### POST /v3/screening/jobs/

- **Auth**: token (`X-Auth-Token` + `X-Customer-Id`) or JWT (`X-Authorization: Bearer`)
- **Description**: Create a screening job.
- **Success**: `201 Created`

**Body Parameters**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `data.job.title` | string | Yes | Job title (max 255 characters) |
| `data.job.description` | string | Yes | Job description, HTML supported (max 10,000 characters) |
| `data.company.name` | string | Yes | Company name (max 255 characters) |
| `data.company.logo_url` | string | Yes | URL to the company logo |
| `settings.webhook_url` | string | No | Override account-level webhook URL |
| `settings.finalization_time_hours` | integer | No | Hours before auto-finalization (1–168, default 168) |
| `settings.duration_days` | integer | No | Days the screening job stays active before it expires (1–365, default 60) |
| `settings.allow_public_applications` | boolean | No | Enable `job_screening_url` for HAPI-collected pattern (default `true`) |
| `settings.skip_phone_validation` | boolean | No | Skip country code validation on phone numbers (default `true`) |
| `settings.allow_duplicate_applies` | boolean | No | Allow same candidate to apply multiple times (default `true`) |
| `metadata` | object | No | Custom key-value pairs (max 50 keys, 255 chars per value) |

A job is created without requirements: the AI writes them from the job description, and you add your own once they are ready. See [POST /v3/screening/jobs/{id}/requirements/](#post-v3screeningjobsidrequirements).

```http
POST https://marketplace.api.vonq.com/v3/screening/jobs/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
Content-Type: application/json
```

```json
{
  "data": {
    "job": {
      "title": "Senior Backend Engineer",
      "description": "<p>We are looking for a senior backend engineer with experience in distributed systems.</p>"
    },
    "company": {
      "name": "Acme Corp",
      "logo_url": "https://example.com/logo.png"
    }
  },
  "settings": {
    "finalization_time_hours": 72,
    "allow_public_applications": true
  },
  "metadata": {
    "ats_job_id": "job-456"
  }
}
```

**Response fields**

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique job identifier |
| `status` | string | Job status: `created` or `deleted` |
| `job_title` | string | The job title |
| `job_screening_url` | string \| null | Public URL for HAPI-collected applications. `null` if `allow_public_applications` is `false`. |
| `company_name` | string | Company name |
| `company_logo_url` | string | Company logo URL as provided in `data.company.logo_url` |
| `stats` | object | Application statistics breakdown (see below) |
| `requirements_ready_at` | string \| null | ISO 8601 timestamp when the screening requirements became final. `null` means not yet ready (typically within a few minutes of creation), or the job predates this field. |
| `created_on` | string | ISO 8601 timestamp |
| `modified_on` | string | ISO 8601 timestamp |

The `stats` object provides a breakdown of application counts:

```json
{
  "applications_total_count": 0,
  "status": {
    "applications_created_count": 0,
    "applications_deleted_count": 0,
    "applications_screened_success_count": 0,
    "applications_screened_timeout_count": 0
  },
  "stage": {
    "applications_applied_count": 0,
    "applications_screened_count": 0,
    "applications_interviewed_count": 0
  }
}
```

**Errors**

| Status | Cause |
|--------|-------|
| `400` | Missing required field, duplicate requirement summary/question, or invalid `finalization_time_hours` range |
| `403` | Screening not enabled for this account, or missing `X-Customer-Id` header |

---

### GET /v3/screening/jobs/

- **Auth**: token or JWT
- **Description**: List screening jobs.

**Query Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| `status` | string | Filter by job status: `created` or `deleted` |
| `requirements_ready` | boolean | Filter by whether the screening requirements are ready (`true` = `requirements_ready_at` is set) |
| `limit` | integer | Results per page (default: 25) |
| `offset` | integer | Starting index for pagination (default: 0) |

```http
GET https://marketplace.api.vonq.com/v3/screening/jobs/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
```

The response uses `limit`/`offset` pagination with a default page size of 25:

```json
{
  "count": 42,
  "next": "https://marketplace.api.vonq.com/v3/screening/jobs/?limit=25&offset=25",
  "previous": null,
  "results": [
    {
      "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "status": "created",
      "job_title": "Senior Backend Engineer",
      "job_screening_url": "https://screening.vonq.com/jobs/a1b2c3d4",
      "company_name": "Acme Corp",
      "company_logo_url": "https://example.com/logo.png",
      "stats": {},
      "requirements_ready_at": "2025-03-01T10:05:00Z",
      "created_on": "2025-03-01T10:00:00Z",
      "modified_on": "2025-03-01T10:00:00Z"
    }
  ]
}
```

---

### GET /v3/screening/jobs/{id}/

- **Auth**: token or JWT
- **Description**: Retrieve full details of a screening job, including `data`, `requirements`, `interview_questions`, and `settings`.

```http
GET https://marketplace.api.vonq.com/v3/screening/jobs/a1b2c3d4-e5f6-7890-abcd-ef1234567890/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
```

In addition to all list-response fields, the details response includes the job's `data`, `requirements`, `interview_questions`, `settings`, and `metadata`.

The `requirements` list is empty until the AI finishes preparing it. Once ready it holds the screening list the AI wrote from the job description (`source: "ai"`), each entry with a stable unique `id`, plus whatever you have since added or reworded (`source: "customer"`).

**`requirements` fields**

| Field | Type | Description |
|-------|------|-------------|
| `requirements[].id` | integer \| null | Stable unique id of the requirement |
| `requirements[].summary` | string \| null | Short label |
| `requirements[].description` | string \| null | Detailed context |
| `requirements[].question` | string \| null | Question the AI evaluates against |
| `requirements[].source` | string | `"customer"` if you added or edited the requirement, `"ai"` if it was AI-generated and untouched |

The `interview_questions` list holds the questions the AI interview agent asks during the screening conversation. They are generated from the job's requirements and appear at the same time as the final `requirements`. The list is empty until then, and stays empty for jobs where the interview agent is disabled.

**`interview_questions` fields**

| Field | Type | Description |
|-------|------|-------------|
| `interview_questions[].id` | integer \| null | Stable unique id of the interview question |
| `interview_questions[].question` | string \| null | The question the interview agent asks |
| `interview_questions[].requirement_summary` | string \| null | `summary` of the requirement this question assesses |

**Errors**

| Status | Cause |
|--------|-------|
| `404` | Job not found or belongs to a different customer |

---

### PATCH /v3/screening/jobs/{id}/requirements/

- **Auth**: token or JWT
- **Description**: Reword the job's requirements.
- **Success**: `200 OK` - returns the full job, same shape as the retrieve endpoint.

Send only the entries you are changing, each with its `id`. An omitted entry is left alone and an omitted field keeps its stored value, so this endpoint only edits: add with `POST` and remove with `DELETE`. Available once `requirements_ready_at` is set.

**Body Parameters**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `requirements` | array | Yes | The requirements you are changing, max 50 |
| `requirements[].id` | integer | Yes | Id of an existing requirement on this job |
| `requirements[].summary` | string | No | Short label (max 255, must be unique within the job, cannot be blanked) |
| `requirements[].description` | string | No | Detailed context for the AI (max 1,000). Send `""` to clear it |
| `requirements[].question` | string | No | Question the AI evaluates against (max 1,000) |

At least one of `summary`, `description` or `question` is required per entry.

```http
PATCH https://marketplace.api.vonq.com/v3/screening/jobs/a1b2c3d4-e5f6-7890-abcd-ef1234567890/requirements/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
Content-Type: application/json
```

```json
{
  "requirements": [
    {
      "id": 22524,
      "question": "Do you confirm you have production experience with Python 3.11?"
    }
  ]
}
```

**Notes**

- Editing an AI-generated requirement makes it yours: its `source` becomes `"customer"`.
- Only applications created after the change are assessed against the updated wording. Candidates who already completed screening are not re-assessed.
- Renaming a requirement's `summary` also relinks the interview questions generated from it.
- Sending wording identical to what is stored is a no-op; nothing is sent upstream.
- If one entry fails to save, the entries saved before it are kept and the request returns `400`. Re-send the remaining edits to retry.

**Errors**

| Status | Cause |
|--------|-------|
| `400` | Unknown `id`, an entry with no editable field, a blank or duplicate `summary`, or the change could not be saved |
| `404` | Job not found or belongs to a different customer |
| `409` | The job is not `created`, or its questions are not ready yet |

---

### POST /v3/screening/jobs/{id}/requirements/

- **Auth**: token or JWT
- **Description**: Add one requirement to the job.
- **Success**: `201 Created` - returns the full job, same shape as the retrieve endpoint.

Available once `requirements_ready_at` is set. The requirement is stored exactly as you write it.

**Body Parameters**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `summary` | string | Yes | Short label (max 255, must be unique within the job) |
| `description` | string | No | Detailed context for the AI (max 1,000). Left empty when omitted |
| `question` | string | Yes | Question the AI evaluates against (max 1,000) |

```http
POST https://marketplace.api.vonq.com/v3/screening/jobs/a1b2c3d4-e5f6-7890-abcd-ef1234567890/requirements/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
Content-Type: application/json
```

```json
{
  "summary": "Work permit",
  "description": "Only candidates who may already work in the EU can be hired.",
  "question": "Do you hold a EU work permit?"
}
```

**Notes**

- The new requirement is yours: its `source` is `"customer"`.
- The AI does not react to it. Nothing is regenerated, no interview question is created for the new requirement, and no `ai_requirements_ready` webhook is sent. Add its interview question yourself if the job runs the interview agent.
- Only applications created after the change are assessed against it.

**Errors**

| Status | Cause |
|--------|-------|
| `400` | Missing `summary` or `question`, a `summary` already used on this job, or the requirement could not be saved |
| `404` | Job not found or belongs to a different customer |
| `409` | The job is not `created`, or its questions are not ready yet |

---

### DELETE /v3/screening/jobs/{id}/requirements/{requirement_id}/

- **Auth**: token or JWT
- **Description**: Delete one requirement, together with the interview questions assessing it.
- **Success**: `200 OK` - returns the full job, same shape as the retrieve endpoint.

**Path Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | UUID | The screening job |
| `requirement_id` | integer | Id of the requirement, as listed under `requirements` on the job |

```http
DELETE https://marketplace.api.vonq.com/v3/screening/jobs/a1b2c3d4-e5f6-7890-abcd-ef1234567890/requirements/22524/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
```

**Notes**

- AI-generated requirements can be deleted like your own, and are not regenerated.
- Applications already screened keep the scores they were given.
- A job can be left with no requirements at all.

**Errors**

| Status | Cause |
|--------|-------|
| `400` | The deletion could not be saved |
| `404` | Job not found, belongs to a different customer, or has no such requirement |
| `409` | The job is not `created`, or its questions are not ready yet |

---

### PATCH /v3/screening/jobs/{id}/interview-questions/

- **Auth**: token or JWT
- **Description**: Reword the job's interview questions.
- **Success**: `200 OK` - returns the full job, same shape as the retrieve endpoint.

Same rules as the requirements endpoint: send only the questions you are changing, each with its `id`. Add with `POST` and remove with `DELETE`.

**Body Parameters**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `interview_questions` | array | Yes | The interview questions you are changing |
| `interview_questions[].id` | integer | Yes | Id of an existing interview question on this job |
| `interview_questions[].question` | string | Yes | The question the interview agent asks (max 2,000) |

```http
PATCH https://marketplace.api.vonq.com/v3/screening/jobs/a1b2c3d4-e5f6-7890-abcd-ef1234567890/interview-questions/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
Content-Type: application/json
```

```json
{
  "interview_questions": [
    {
      "id": 84120,
      "question": "Walk me through a Python service you designed and shipped end to end."
    }
  ]
}
```

**Errors**

| Status | Cause |
|--------|-------|
| `400` | Unknown `id`, or the change could not be saved |
| `404` | Job not found or belongs to a different customer |
| `409` | The job is not `created`, or its questions are not ready yet |

---

### POST /v3/screening/jobs/{id}/interview-questions/

- **Auth**: token or JWT
- **Description**: Add one interview question, assessing one of the job's requirements.
- **Success**: `201 Created` - returns the full job, same shape as the retrieve endpoint.

Available once `requirements_ready_at` is set, on jobs that run the interview agent. A requirement holds a single interview question, so `requirement_id` must name a requirement that has none yet; reword the existing one instead.

**Body Parameters**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `requirement_id` | integer | Yes | Id of the requirement this question assesses, as listed under `requirements` on the job |
| `question` | string | Yes | The question the interview agent asks (max 2,000) |

```http
POST https://marketplace.api.vonq.com/v3/screening/jobs/a1b2c3d4-e5f6-7890-abcd-ef1234567890/interview-questions/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
Content-Type: application/json
```

```json
{
  "requirement_id": 22526,
  "question": "Which work permit do you hold, and until when is it valid?"
}
```

**Errors**

| Status | Cause |
|--------|-------|
| `400` | Unknown `requirement_id`, a requirement that already has an interview question, or the question could not be saved |
| `404` | Job not found or belongs to a different customer |
| `409` | The job is not `created`, its questions are not ready yet, or the interview agent is disabled for it |

---

### DELETE /v3/screening/jobs/{id}/interview-questions/{question_id}/

- **Auth**: token or JWT
- **Description**: Delete one interview question. Its requirement stays, and candidates are still screened against it.
- **Success**: `200 OK` - returns the full job, same shape as the retrieve endpoint.

**Path Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | UUID | The screening job |
| `question_id` | integer | Id of the interview question, as listed under `interview_questions` on the job |

```http
DELETE https://marketplace.api.vonq.com/v3/screening/jobs/a1b2c3d4-e5f6-7890-abcd-ef1234567890/interview-questions/84120/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
```

**Errors**

| Status | Cause |
|--------|-------|
| `400` | The deletion could not be saved |
| `404` | Job not found, belongs to a different customer, or has no such interview question |
| `409` | The job is not `created`, or its questions are not ready yet |

---

### DELETE /v3/screening/jobs/{id}/

- **Auth**: token or JWT
- **Description**: Soft-delete a screening job. Sets `status` to `deleted`.

```http
DELETE https://marketplace.api.vonq.com/v3/screening/jobs/a1b2c3d4-e5f6-7890-abcd-ef1234567890/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
```

**Notes**

- Soft-delete only-the job data is retained but the job is no longer active.
- Creating an application on a deleted job fails.

---

### POST /v3/screening/jobs/{job_id}/applications/

- **Auth**: token or JWT
- **Description**: Create an application (candidate screening session) for a job.
- **Success**: `201 Created`

This endpoint accepts two content types:

- **`multipart/form-data`**-for direct file uploads. Send the JSON body in a field named `json`, and files as additional form fields.
- **`application/json`**-for remote file URLs. Send everything in a single JSON body.

**Body Parameters**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `initial_payload.firstName` | string | Yes | Candidate's first name |
| `initial_payload.lastName` | string | Yes | Candidate's last name |
| `initial_payload.emailAddresses` | array | Yes | Exactly one object: `{ "emailAddress": "...", "isPreferred": true }` |
| `initial_payload.phoneNumbers` | array | Yes | Exactly one object: `{ "number": "...", "type": "personal", "isPreferred": true }` |
| `initial_payload.attachments` | array | No | `[{ "filename": "resume.pdf", "type": "RESUME" }]` |
| `files` | binary | No | File uploads (multipart only). Each filename must match an entry in `attachments`. |
| `remote_files` | array | No | `[{ "filename": "resume.pdf", "remoteUrl": "https://..." }]` (JSON only). Each filename must match an entry in `attachments`. |
| `notes` | string | No | Recruiter context for the AI (max 5,000 characters) |
| `metadata` | object | No | Custom key-value pairs (max 50 keys) |

Accepted attachment file extensions: `.pdf`, `.docx`, `.jpg`, `.jpeg`, `.png`, `.txt`. Remote files have a 10-second download timeout and a 10 MB size limit.

<!-- theme: warning -->
> **`remoteUrl` must be publicly reachable from the internet.** The file is downloaded by VONQ's servers, not by the client submitting the request. URLs on private networks, behind a VPN, or on `localhost` in a development environment will fail to download even if they open fine in your own browser. Use the multipart `files` upload when the file is not publicly hosted.

Create an application with a remote resume file:

```http
POST https://marketplace.api.vonq.com/v3/screening/jobs/a1b2c3d4-e5f6-7890-abcd-ef1234567890/applications/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
Content-Type: application/json
```

```json
{
  "initial_payload": {
    "firstName": "Jane",
    "lastName": "Doe",
    "emailAddresses": [
      { "emailAddress": "jane.doe@example.com", "isPreferred": true }
    ],
    "phoneNumbers": [
      { "number": "+31612345678", "type": "personal", "isPreferred": true }
    ],
    "attachments": [
      { "filename": "jane-doe-cv.pdf", "type": "RESUME" }
    ]
  },
  "remote_files": [
    { "filename": "jane-doe-cv.pdf", "remoteUrl": "https://example.com/resumes/jane-doe-cv.pdf" }
  ],
  "notes": "Referred by CTO. Strong distributed systems background."
}
```

<details>
<summary>Multipart form-data example</summary>

```http
POST https://marketplace.api.vonq.com/v3/screening/jobs/a1b2c3d4-e5f6-7890-abcd-ef1234567890/applications/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
Content-Type: multipart/form-data; boundary=----FormBoundary

------FormBoundary
Content-Disposition: form-data; name="json"
Content-Type: application/json

{
  "initial_payload": {
    "firstName": "Jane",
    "lastName": "Doe",
    "emailAddresses": [
      { "emailAddress": "jane.doe@example.com", "isPreferred": true }
    ],
    "phoneNumbers": [
      { "number": "+31612345678", "type": "personal", "isPreferred": true }
    ],
    "attachments": [
      { "filename": "jane-doe-cv.pdf", "type": "RESUME" }
    ]
  }
}
------FormBoundary
Content-Disposition: form-data; name="files"; filename="jane-doe-cv.pdf"
Content-Type: application/pdf

<binary file content>
------FormBoundary--
```

</details>

**Response fields**

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique application identifier |
| `job_id` | string | Parent job identifier |
| `application_url` | string | Candidate-specific screening URL (resumable) |
| `full_name` | string | Candidate's full name |
| `is_external_application` | boolean | `true` if candidate applied via `job_screening_url` |
| `status` | string | Current application status |
| `screened_at` | string \| null | ISO 8601 timestamp when screening completed |
| `screened_stage` | string \| null | Stage reached during screening |
| `screened_score` | number \| null | AI evaluation score |
| `attachments.initial_files_count` | integer | Number of files submitted |
| `attachments.screened_files_count` | integer | Number of files in the dossier |
| `initial_payload` | object | Candidate data as submitted |
| `screened_payload` | object \| null | Enriched candidate data. `null` until finalization. Same shape as `initial_payload`-no new fields added. |
| `notes` | string \| null | Recruiter notes as submitted. `null` if not provided. |
| `timeout_at` | string | ISO 8601 timestamp when auto-finalization triggers |

**Errors**

| Status | Cause |
|--------|-------|
| `400` | Missing required field, filename mismatch between `attachments` and files, or `allow_duplicate_applies` is `false` and candidate already applied |
| `404` | Job not found or has been deleted |
| `413` | Remote file exceeds 10 MB |
| `504` | Remote file download timed out (10-second limit) |

---

### GET /v3/screening/jobs/{job_id}/applications/

- **Auth**: token or JWT
- **Description**: List applications for a job with filtering and sorting.

**Query Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| `status` | string | Filter by application status |
| `screened_stage` | string | Filter by screening stage |
| `is_external_application` | boolean | Filter by application source |
| `ordering` | string | Sort field. Prefix with `-` for descending. Default: `-created_on`. Options: `created_on`, `modified_on`, `status`, `screened_at`, `screened_stage`, `timeout_at`, `screened_score` |
| `limit` | integer | Results per page (default: 25) |
| `offset` | integer | Starting index for pagination (default: 0) |

List applications sorted by score (highest first):

```http
GET https://marketplace.api.vonq.com/v3/screening/jobs/a1b2c3d4-e5f6-7890-abcd-ef1234567890/applications/?ordering=-screened_score HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
```

The response uses `limit`/`offset` pagination with a default page size of 25.

---

### GET /v3/screening/jobs/{job_id}/applications/{id}/

- **Auth**: token or JWT
- **Description**: Retrieve full details of a single application.

```http
GET https://marketplace.api.vonq.com/v3/screening/jobs/a1b2c3d4-e5f6-7890-abcd-ef1234567890/applications/f9e8d7c6-b5a4-3210-fedc-ba9876543210/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
```

---

### DELETE /v3/screening/jobs/{job_id}/applications/{id}/

- **Auth**: token or JWT
- **Description**: Soft-delete an application.

```http
DELETE https://marketplace.api.vonq.com/v3/screening/jobs/a1b2c3d4-e5f6-7890-abcd-ef1234567890/applications/f9e8d7c6-b5a4-3210-fedc-ba9876543210/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
```

---

### GET /v3/screening/jobs/{job_id}/applications/{id}/attachments/

- **Auth**: token or JWT
- **Description**: List all attachments for an application, split into initial and screened files.

```http
GET https://marketplace.api.vonq.com/v3/screening/jobs/a1b2c3d4-e5f6-7890-abcd-ef1234567890/applications/f9e8d7c6-b5a4-3210-fedc-ba9876543210/attachments/ HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
```

```json
{
  "initial_files": [
    {
      "filename": "jane-doe-cv.pdf",
      "contentType": "application/pdf",
      "size": 245760,
      "md5": "d41d8cd98f00b204e9800998ecf8427e",
      "url": "https://marketplace.api.vonq.com/v3/screening/jobs/.../attachments/initial/?filename=jane-doe-cv.pdf",
      "url_public": "https://marketplace.api.vonq.com/v3/screening/applications-public/.../files/initial?filename=jane-doe-cv.pdf&expires=1710100000&signature=abc123"
    }
  ],
  "screened_files": [
    {
      "filename": "jane-doe-dossier.pdf",
      "contentType": "application/pdf",
      "size": 512000,
      "md5": "a3c2b1d4e5f67890abcdef1234567890",
      "url": "https://marketplace.api.vonq.com/v3/screening/jobs/.../attachments/screened/?filename=jane-doe-dossier.pdf",
      "url_public": "https://marketplace.api.vonq.com/v3/screening/applications-public/.../files/screened?filename=jane-doe-dossier.pdf&expires=1710100000&signature=def456"
    }
  ]
}
```

**Response fields**

| Field | Type | Description |
|-------|------|-------------|
| `initial_files` | array | Files submitted with the application |
| `screened_files` | array | Files generated by the screening process (available after `screened_success`) |
| `*.filename` | string | Original filename |
| `*.contentType` | string | MIME type |
| `*.size` | integer | File size in bytes |
| `*.md5` | string | MD5 checksum |
| `*.url` | string | Authenticated download URL (requires auth headers) |
| `*.url_public` | string | Signed URL, no auth needed. Expires after 1 hour. |

---

### GET /v3/screening/jobs/{job_id}/applications/{id}/attachments/{file_type}/?filename={name}

- **Auth**: token or JWT
- **Description**: Download a specific attachment file.

**Path/Query Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| `fileType` | string | `initial` or `screened` |
| `filename` | string | Exact filename to download |

```http
GET https://marketplace.api.vonq.com/v3/screening/jobs/a1b2c3d4-e5f6-7890-abcd-ef1234567890/applications/f9e8d7c6-b5a4-3210-fedc-ba9876543210/attachments/screened/?filename=jane-doe-dossier.pdf HTTP/1.1
X-Auth-Token: <your Partner token here>
X-Customer-Id: <customer-id>
```

Returns binary content with `Content-Type` and `Content-Disposition` headers.
