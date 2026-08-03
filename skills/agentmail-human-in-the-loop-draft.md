---
name: Human-in-the-loop draft review
description: Have an agent compose an email as a Draft, hold it for human approval, then send the approved draft — a safe, reviewable outbound workflow.
api: openapi/agentmail-openapi-original.yml
operations:
  - POST /v0/inboxes/{inbox_id}/drafts        # create draft
  - GET  /v0/inboxes/{inbox_id}/drafts        # list drafts
  - PATCH /v0/inboxes/{inbox_id}/drafts/{draft_id}  # update draft
  - POST /v0/inboxes/{inbox_id}/drafts/{draft_id}/send  # send after approval
---

# Human-in-the-loop draft review

Keep a human in control of what an agent sends.

## Auth
- `Authorization: Bearer <am_...>`, base URL `https://api.agentmail.to`.

## Steps
1. **Compose a draft** — `POST /v0/inboxes/{inbox_id}/drafts` with `to`, `subject`, and body. Pass a `client_id` so re-running the agent does not create duplicate drafts (idempotent create). Save `draft_id`.
2. **Surface for review** — list pending drafts with `GET /v0/inboxes/{inbox_id}/drafts` and present them to a human.
3. **Edit if needed** — `PATCH /v0/inboxes/{inbox_id}/drafts/{draft_id}` to apply human edits before sending.
4. **Send on approval** — `POST /v0/inboxes/{inbox_id}/drafts/{draft_id}/send`. Set an `Idempotency-Key` header; a retry returns the original `message_id`/`thread_id` and sends no second email.

## Rules
- Never auto-send: gate step 4 behind explicit human approval.
- Respect allow/block lists (`lists/`) so the agent cannot send to blocked recipients.
- On `409 conflict` (Idempotency-Key collision) wait and retry the original request rather than generating a new send.
