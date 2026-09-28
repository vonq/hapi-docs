---
id: direct-apply-webhooks-endpoints
title: Direct Apply-Webhooks - Endpoint Reference
description: Full payload structure, file delivery mode examples, and CPA+ webhook format for Direct Apply webhooks.
category: guides/direct-apply
related:
- direct-apply-webhooks
endpoints: []
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
    "firstName": "Alex",
    "lastName": "Example",
    "formattedName": "Alex Example",
    "source": "example-board",
    "emailAddresses": [
      { "type": "PERSONAL", "emailAddress": "alex@example.com", "isPreferred": true }
    ],
    "phoneNumbers": [
      { "number": "+10000000000", "isPreferred": true }
    ],
    "questions": [
      {
        "question": "What interests you about this role?",
        "id": "question-interest",
        "type": "text",
        "answers": [{ "answer": "The opportunity to solve practical problems." }]
      },
      {
        "question": "How much relevant experience do you have?",
        "id": "question-experience",
        "type": "choice",
        "answers": [
          { "id": "option-one-to-three-years", "answer": "1 to 3 years" }
        ]
      },
      {
        "question": "Which work arrangements suit you?",
        "id": "question-work-arrangements",
        "type": "multi-choice",
        "answers": [
          { "id": "option-hybrid", "answer": "Hybrid" },
          { "id": "option-remote", "answer": "Remote" }
        ]
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

### Field Presence

Job boards differ in how much candidate data they provide. Only a minimal core is guaranteed:

| Field | Presence |
|-------|----------|
| `payload.firstName` | Always present |
| `payload.emailAddresses` | Always present, at least one entry |
| `payload.lastName` | May be missing or empty-some boards deliver single-name candidates (e.g., `firstName: "Jef"` with `formattedName` but no surname) |
| `payload.attachments` | May be missing or empty-boards can allow applications without a resume or other files |
| All other profile fields | Optional; presence varies per job board |

Do not reject applications that lack `lastName` or attachments.

### Questionnaire Answer Types

Question `type` values are delivered exactly as defined at ordering time - HAPI does not transform them:

| Type | Description |
|------|-------------|
| `text` | Free-text answer |
| `choice` | Single-select-one answer |
| `multi-choice` | Multi-select-one or more answers |
| `date` | ISO 8601 datetime with timezone, e.g. `2026-10-01T00:00:00+00:00` |
| `file` | The `filename` of an entry in `attachments` (its `comments` carries the question `id`) |
| `int` | Whole number, delivered as a string, e.g. `"8"`. Some boards send `integer`; treat both the same. |
| `float` | Decimal number with a dot, delivered as a string, e.g. `"32.5"` |

`date`, `file`, `int` and `float` answers are validated before delivery; applications with malformed answers are rejected at the job board side.

Each question object in the webhook:

| Field | Type | Description |
|-------|------|-------------|
| `question` | string | The question text shown to the candidate |
| `id` | string | The question ID you defined when creating the questionnaire |
| `type` | string | Forwarded verbatim from the job board - normally one of the types above. Treat as an open string, not a closed enum. |
| `answers` | array | Candidate's answers. See the rules below. |

Every answer object contains `answer`, always as a string. The presence of `id` depends on the question type:

| Question type | Answer shape | Number of entries |
|---------------|--------------|-------------------|
| `text` | `{ "answer": "Within one month" }` | One |
| `choice` | `{ "id": "option-one-month", "answer": "Within one month" }` | One selected option |
| `multi-choice` | `{ "id": "option-remote", "answer": "Remote" }` | One entry per selected option |

For `choice` and `multi-choice`, `id` is the selected option ID and `answer` is the option text shown to the candidate. Use the pair of question `id` and answer `id` to identify a selected option. An answer ID may appear under more than one question, so it is not a global identifier.

Types without configured options, including `text`, do not have an answer ID. Their answer objects contain only `answer`.

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

### Attachment Filenames

`payload.attachments[].filename` and the `filename` of the matching file part are always the same string, in every delivery mode. Pair each file with its attachment entry on that value, including in the split modes where the file arrives in a later request.

Filenames are normalized before delivery: non-printable characters are removed and leading and trailing whitespace is trimmed. Some job boards embed invisible characters such as U+200E (left-to-right mark) in the name the candidate uploaded, so the filename you receive can differ from the original.

Within one application filenames are unique, so they are safe to use as a key while processing that application. They are not unique across applications, and normalization is not a substitute for your own path handling when you store the file.

## File Delivery Mode Examples

In every mode, an application without files is delivered as a single `application/json` POST (the payload request shown in Mode 2), and no file requests follow. If your endpoint can only ingest `multipart/form-data`, your account manager can enable forced multipart delivery, which sends the `json` form field without file parts instead.

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
