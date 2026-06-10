---
title: Direct Apply-Webhooks - Endpoint Reference
description: Full payload structure, file delivery mode examples, and CPA+ webhook format for Direct Apply webhooks.
category: guides/direct-apply
---

> For conceptual overview, see [Direct Apply-Webhooks](./webhooks.md).

## Payload Structure

The webhook payload contains the candidate's application data, questionnaire answers, and attachment metadata.

```json
{
  "requestId": "e3f4a5b6-c7d8-9012-cdef-123456789012",
  "campaignId": "e2f3a4b5-c6d7-8901-bcde-f12345678901",
  "productId": "d7e8f9a0-b1c2-3456-0123-567890123456",
  "payload": {
    "firstName": "Jane",
    "lastName": "Doe",
    "formattedName": "Jane Doe",
    "source": "seek",
    "emailAddresses": [
      { "type": "PERSONAL", "emailAddress": "jane.doe@example.com", "isPreferred": true }
    ],
    "phoneNumbers": [
      { "number": "+31612345678", "isPreferred": true }
    ],
    "questions": [
      {
        "question": "Why are you interested in this role?",
        "id": "q1",
        "type": "OPEN",
        "answers": [{ "answer": "I am passionate about..." }]
      },
      {
        "question": "Do you have a valid work permit?",
        "id": "q2",
        "type": "SELECT",
        "answers": [{ "answer": "Yes", "id": "yes" }]
      }
    ],
    "attachments": [
      { "filename": "resume.pdf", "type": "RESUME" },
      { "filename": "cover-letter.pdf", "type": "COVER_LETTER" }
    ]
  },
  "type": "payload"
}
```

### Top-Level Fields

| Field | Type | Description |
|-------|------|-------------|
| `requestId` | UUID | Unique ID for this delivery-use for deduplication and to correlate files in split mode |
| `campaignId` | UUID | The campaign this application belongs to |
| `productId` | UUID | The specific product (job board) the candidate applied on |
| `payload` | object | Candidate application data |
| `type` | string | `"payload"` for the main request, `"file"` for split file requests |

### Direct Apply–Specific Payload Fields

| Field | Type | Description |
|-------|------|-------------|
| `payload.source` | string | Job board identifier (e.g., `seek`, `indeed`, `linkedin`) |
| `payload.formattedName` | string | Full name as formatted by the job board (optional) |
| `payload.jobboardResumeId` | string | Job board's internal resume/profile ID (optional) |
| `payload.jobboardApplyDateTime` | string | When the candidate applied on the job board (ISO 8601, optional) |
| `payload.questions` | array | Questionnaire answers (only if questionnaire was configured) |
| `payload.attachments` | array | File metadata-each entry has `filename` and `type` |
| `payload.consents` | array | Consent information provided by the candidate (optional) |
| `payload.cpa` | object | CPA+ flag-present only for CPA+ applications (see [CPA+ Applications](#cpa-applications)) |

### Questionnaire Answer Field Mapping

| Input Type (ordering) | Webhook Type (delivery) | Description |
|----------------------|------------------------|-------------|
| `text` | `OPEN` | Free-text answer |
| `choice` | `SELECT` | Single-select-one answer |
| `multi-choice` | `MULTISELECT` | Multi-select-one or more answers |

Each question object in the webhook:

| Field | Type | Description |
|-------|------|-------------|
| `question` | string | The question text shown to the candidate |
| `id` | string | The question ID you defined when creating the questionnaire |
| `type` | string | `OPEN`, `SELECT`, or `MULTISELECT` |
| `answers` | array | Candidate's answers. Each has `answer` (text) and optionally `id` (matching the option ID you defined) |

### Attachment Types

| Type | Description |
|------|-------------|
| `RESUME` | Candidate's resume / CV |
| `COVER_LETTER` | Cover letter |
| `SUMMARY` | Profile summary |
| `CERTIFICATE` | Certificate or qualification |
| `MOTIVATION` | Motivation letter |
| `PHOTO` | Candidate photo |
| `DOSSIER` | Screening dossier (CPA+ applications) |
| `OTHER` | Other file type |
| `null` | Unclassified-not all job boards classify files |

## File Delivery Mode Examples

### Mode 1: Single Multipart Request

JSON payload and all files in one `multipart/form-data` POST. The JSON is in a form field named `json`; files are additional form fields.

<!-- theme: info -->
> ### Recommended
> Single request mode allows you to atomically process the entire application in one request.

<details>
<summary>Example: Single multipart request</summary>

```text
POST https://your-ats.example.com/webhooks/vonq
Content-Type: multipart/form-data; boundary=----Boundary

------Boundary
Content-Disposition: form-data; name="json"
Content-Type: application/json

{"requestId":"a5e17923-...","campaignId":"6bd0c43c-...","productId":"657ca9a1-...","payload":{...},"type":"payload"}

------Boundary
Content-Disposition: form-data; name="resume.pdf"; filename="resume.pdf"
Content-Type: application/pdf

<binary file content>

------Boundary
Content-Disposition: form-data; name="cover-letter.pdf"; filename="cover-letter.pdf"
Content-Type: application/pdf

<binary file content>

------Boundary--
```

</details>

### Mode 2: Split Requests

The payload is sent first as `application/json`. Each file follows as a separate `multipart/form-data` request. All requests share the same `requestId`.

The payload request always arrives first. File requests are sent only after your endpoint returns `2xx` for the payload, so you can be sure the application data is stored before files arrive.

<details>
<summary>Example: Split requests</summary>

**First request-payload:**

```json
{
  "requestId": "e3f4a5b6-c7d8-9012-cdef-123456789012",
  "campaignId": "e2f3a4b5-c6d7-8901-bcde-f12345678901",
  "productId": "d7e8f9a0-b1c2-3456-0123-567890123456",
  "payload": { "firstName": "Jane", "lastName": "Doe", "..." : "..." },
  "type": "payload"
}
```

**Subsequent request-file:**

```text
POST https://your-ats.example.com/webhooks/vonq
Content-Type: multipart/form-data

requestId=e3f4a5b6-c7d8-9012-cdef-123456789012
campaignId=e2f3a4b5-c6d7-8901-bcde-f12345678901
productId=d7e8f9a0-b1c2-3456-0123-567890123456
type=file

file=<<resume.pdf>>
```

</details>

### Mode 3: Base64 Single Request

Everything in one JSON POST. File content is base64-encoded in each attachment's `base64Content` field.

Best for JSON-only systems.

<details>
<summary>Example: Base64 single request</summary>

```json
{
  "requestId": "e3f4a5b6-c7d8-9012-cdef-123456789012",
  "campaignId": "e2f3a4b5-c6d7-8901-bcde-f12345678901",
  "productId": "d7e8f9a0-b1c2-3456-0123-567890123456",
  "payload": {
    "firstName": "Jane",
    "lastName": "Doe",
    "emailAddresses": [{ "emailAddress": "jane.doe@example.com", "isPreferred": true }],
    "attachments": [
      {
        "filename": "resume.pdf",
        "type": "RESUME",
        "base64Content": "JVBERi0xLjQKJeLjz9MK..."
      }
    ]
  },
  "type": "payload"
}
```

</details>

### Mode 4: Base64 Split Requests

Payload first without file content, then each file as a separate JSON request with `base64Content`.

Best for JSON-only systems with request size limits.

<details>
<summary>Example: Base64 split file request</summary>

```json
{
  "requestId": "e3f4a5b6-c7d8-9012-cdef-123456789012",
  "campaignId": "e2f3a4b5-c6d7-8901-bcde-f12345678901",
  "productId": "d7e8f9a0-b1c2-3456-0123-567890123456",
  "type": "file",
  "filename": "resume.pdf",
  "contentType": "application/pdf",
  "base64Content": "JVBERi0xLjQKJeLjz9MK..."
}
```

</details>

## CPA+ Applications

CPA+ candidate applications are delivered through the same Direct Apply webhook. The payload is identical, with one addition: a `cpa` object indicating the application passed CPA review before delivery.

```json
{
  "requestId": "...",
  "campaignId": "...",
  "productId": "...",
  "payload": {
    "firstName": "John",
    "lastName": "Smith",
    "cpa": {
      "reviewed": true
    },
    "attachments": [
      { "filename": "dossier.pdf", "type": "DOSSIER" },
      { "filename": "resume.pdf", "type": "RESUME" }
    ]
  },
  "type": "payload"
}
```

CPA+ and Direct Apply are always different products-you will never find a single product with both enabled. But from a webhook perspective, they use the same infrastructure and payload format.
