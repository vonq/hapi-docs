---
id: screening-jobs-and-applications
title: Screening-Jobs & Applications
description: Create screening jobs, submit applications, manage candidates, download dossier PDFs.
category: guides/screening
endpoints:
- POST /v3/screening/jobs/
- GET /v3/screening/jobs/
- GET /v3/screening/jobs/{id}/
- PATCH /v3/screening/jobs/{id}/requirements/
- PATCH /v3/screening/jobs/{id}/interview-questions/
- DELETE /v3/screening/jobs/{id}/
- POST /v3/screening/jobs/{job_id}/applications/
- GET /v3/screening/jobs/{job_id}/applications/
- GET /v3/screening/jobs/{job_id}/applications/{id}/
- DELETE /v3/screening/jobs/{job_id}/applications/{id}/
- GET /v3/screening/jobs/{job_id}/applications/{id}/attachments/
- GET /v3/screening/jobs/{job_id}/applications/{id}/attachments/{file_type}/
prerequisites:
- screening-introduction
related:
- screening-introduction
- screening-webhooks
audience:
- developer
difficulty: intermediate
keywords:
- screening_job
- application
- interview_question
- dossier
- finalization_time_hours
- application_status
---

# Jobs & Applications

> Create screening jobs, submit candidate applications, and retrieve AI-enriched dossiers.

## Overview

A **screening job** defines what you are hiring for-the title, description, company info, and evaluation requirements. The job's own content is fixed once created: you cannot change the title, description or company, only soft-delete the job. The wording of its requirements and interview questions _can_ be edited once the AI has finished preparing them.

An **application** represents a single candidate screening session tied to a job. You submit the candidate's contact details and resume, the candidate completes the screening flow, and HAPI returns an enriched dossier with scores and parsed data.

For background on how screening fits into HAPI, see [Screening-Introduction](./01-introduction.md).

## Key Concepts

- **Requirements**-criteria the AI uses to evaluate candidates. Requirements are _not_ shown to candidates. Each requirement has a `question` (required) and optional `summary` and `description`.
- **Screening requirements**-shortly after job creation, the AI expands your requirements into the final set (it may add AI-generated requirements). Once ready, `requirements_ready_at` is set and the job's `requirements` list is updated to the final version. Opt-in [job event webhooks](./webhooks.md#job-events) can notify you when this happens.
- **Editing questions**-once `requirements_ready_at` is set, you can reword the requirements and interview questions to be more specific, stricter, or to fix a typo. Send only the entries you are changing, each with its `id`; omitted entries and omitted fields are left untouched, so entries cannot be added or removed. Only applications created after the change are assessed against the updated wording-candidates who already went through screening are not re-assessed.
- **Interview questions**-the questions the AI interview agent asks during the screening conversation. They are generated from your requirements and become available on the job at the same time as the final `requirements`, in the job's `interview_questions` list. Jobs with the interview agent disabled return an empty list.
- **Finalization**-the point at which screening completes and the dossier becomes available. Controlled by `finalization_time_hours` (default: 168 hours / 7 days).
- **Dossier**-the enriched candidate profile available after successful screening. Includes `screened_payload` (same shape as `initial_payload`, enriched) and `screened_files`.
- **Attachments**-optional files submitted with an application (e.g. resume, cover letter). Accepted extensions: `.pdf`, `.docx`, `.jpg`, `.jpeg`, `.png`, `.txt`. Upload directly via `multipart/form-data` (`files`) or reference remote URLs via JSON (`remote_files`, 10 MB / 10-second download limit). Every entry in `initial_payload.attachments` must correspond to an uploaded or remote file, and vice versa.
- **ATS-collected pattern**-your ATS collects candidate data and submits it via the API. The candidate receives an `application_url` to complete screening.
- **HAPI-collected pattern**-you share the `job_screening_url` and candidates apply directly. These applications have `is_external_application: true`.

## Endpoints

| Endpoint | Description |
|----------|-------------|
| `POST /v3/screening/jobs/` | Create a screening job with title, description, company info, requirements, and settings |
| `GET /v3/screening/jobs/` | List screening jobs (paginated), filterable by `status` and `requirements_ready` |
| `GET /v3/screening/jobs/{id}/` | Retrieve full job details including `data`, `requirements`, `interview_questions`, and `settings` |
| `PATCH /v3/screening/jobs/{id}/requirements/` | Edit the wording of the job's requirements |
| `PATCH /v3/screening/jobs/{id}/interview-questions/` | Edit the wording of the job's interview questions |
| `DELETE /v3/screening/jobs/{id}/` | Soft-delete a screening job |
| `POST /v3/screening/jobs/{job_id}/applications/` | Create a candidate screening application (supports multipart upload or remote file URLs) |
| `GET /v3/screening/jobs/{job_id}/applications/` | List applications with filtering by `status`, `screened_stage`, `is_external_application`, and sorting |
| `GET /v3/screening/jobs/{job_id}/applications/{id}/` | Retrieve full application details including `screened_payload` and `screened_score` |
| `DELETE /v3/screening/jobs/{job_id}/applications/{id}/` | Soft-delete an application |
| `GET /v3/screening/jobs/{job_id}/applications/{id}/attachments/` | List initial and screened files with authenticated and signed download URLs |
| `GET /v3/screening/jobs/{job_id}/applications/{id}/attachments/{file_type}/?filename=<name>` | Download a specific attachment file (`initial` or `screened`), for example `?filename=resume.pdf` |

See [Screening-Jobs & Applications - Endpoint Reference](./jobs-and-applications.endpoints.md) for full request/response details, body parameters, response field tables, and code examples.

## Workflows

### ATS-Collected Pattern

Your ATS collects the candidate's data and resume, then submits everything to HAPI. The candidate receives a link to complete the screening flow.

```mermaid
%%{init: {'sequence': {'useMaxWidth': true}}}%%
sequenceDiagram
    participant ATS
    participant HAPI
    participant Candidate

    ATS->>HAPI: POST /v3/screening/jobs/ (create job)
    HAPI-->>ATS: job id

    ATS->>HAPI: POST /v3/screening/jobs/{job_id}/applications/ (candidate data + resume)
    HAPI-->>ATS: application id + application_url

    ATS->>Candidate: Share application_url
    Candidate->>HAPI: Complete screening flow

    alt Polling
        ATS->>HAPI: GET /v3/screening/jobs/{job_id}/applications/{id}/
        HAPI-->>ATS: status: screened_success
    else Webhook
        HAPI-->>ATS: Webhook notification
    end

    ATS->>HAPI: GET /v3/screening/jobs/{job_id}/applications/{id}/attachments/
    HAPI-->>ATS: Dossier files (screened_files)
```

### HAPI-Collected Pattern

You share the job's public screening URL. Candidates apply directly-no application creation via the API is needed.

```mermaid
%%{init: {'sequence': {'useMaxWidth': true}}}%%
sequenceDiagram
    participant ATS
    participant HAPI
    participant Candidate

    ATS->>HAPI: POST /v3/screening/jobs/ (create job)
    HAPI-->>ATS: job id + job_screening_url

    ATS->>Candidate: Share job_screening_url (career page, email, etc.)
    Candidate->>HAPI: Apply + complete screening flow

    alt Polling
        ATS->>HAPI: GET /v3/screening/jobs/{job_id}/applications/?is_external_application=true
        HAPI-->>ATS: New applications with status updates
    else Webhook
        HAPI-->>ATS: Webhook notification
    end

    ATS->>HAPI: GET /v3/screening/jobs/{job_id}/applications/{id}/attachments/
    HAPI-->>ATS: Dossier files (screened_files)
```

## Edge Cases & Gotchas

<!-- theme: warning -->
> ### Phone Number Validation
> Phone numbers must include a country code (e.g. `+31612345678`) unless `skip_phone_validation` is enabled on the job. The default is `true` (validation skipped), but if set to `false`, missing country codes cause a `400` error.

<!-- theme: warning -->
> ### File and Attachment Matching
> Every filename listed in `initial_payload.attachments` must have a corresponding file in `files` (multipart) or `remote_files` (JSON), and every uploaded/remote file must be listed in `attachments`. A mismatch in either direction returns `400`.

<!-- theme: warning -->
> ### Remote File Limits
> Remote files have a 10-second download timeout and a 10 MB size limit. If the remote server is slow or the file is too large, the request fails.

<!-- theme: warning -->
> ### Duplicate Applications
> When `allow_duplicate_applies` is `false` on a job, submitting a second application from the same candidate returns `400`. The default is `true`.

<!-- theme: warning -->
> ### Dossier Availability
> The `screened_payload` and `screened_files` are only available when the application status is `screened_success`. Applications that time out (`screened_timeout`) do not produce a dossier-no collected information is returned.

<!-- theme: warning -->
> ### Deleted Jobs
> Creating an application on a deleted job fails. Always check job `status` before submitting applications if your workflow allows job deletion.

<!-- theme: warning -->
> ### Data Retention
> Candidate data is removed 90 days after the screening job's campaign is closed (deleted through the API, or ended in the screening platform). After that, application payloads and notes come back empty, the attachment list is empty, and attachment downloads return `410 Gone`. Scores, stage, status and timestamps are kept. Retrieve and store anything you need in your ATS before the window closes; reopening a campaign suspends the countdown.

## Related

- [Screening-Introduction](./01-introduction.md)-overview and key concepts
- [Screening-Webhooks](./webhooks.md)-real-time notifications for screening events
- [Authentication](../03-authentication-and-users/authentication.md)-token and JWT authentication details
