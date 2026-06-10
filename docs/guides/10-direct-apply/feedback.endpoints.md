---
title: Direct Apply-Feedback - Endpoint Reference
description: Full request/response details for the application feedback endpoint.
category: guides/direct-apply
---

> For conceptual overview, see [Direct Apply-Feedback](./feedback.md).

## Endpoints

### POST /v3/apply-applications/application-feedback/

- **Auth**: token (`X-Auth-Token`)
- **Description**: Send an application status update to the job board.

**Body Parameters**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `request_id` | string (UUID) | Yes | The `requestId` from the original Direct Apply webhook payload |
| `status` | string | Yes | One of: `delivered`, `qualified`, `cancelled`, `closed_rejected`, `closed_hired` |

Send a "delivered" status after storing the application:

```http
POST https://marketplace.api.vonq.com/v3/apply-applications/application-feedback/ HTTP/1.1
X-Auth-Token: <your Partner token here>
Content-Type: application/json
```

```json
{
  "request_id": "b4c5d6e7-f8a9-0123-def0-234567890123",
  "status": "delivered"
}
```

**Response (201 Created):**

```json
{
  "status": "OK"
}
```

**Errors**

| Status | Cause |
|--------|-------|
| `400` | Invalid `request_id` (not found, doesn't belong to your ATS, malformed UUID) or invalid `status` value |
