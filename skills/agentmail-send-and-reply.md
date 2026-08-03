---
name: Send an email and handle the reply
description: Create an AgentMail inbox, send an initial email, then detect and reply to the inbound response in the same thread — the core two-way email agent loop.
api: openapi/agentmail-openapi-original.yml
operations:
  - POST /v0/inboxes            # create inbox
  - POST /v0/inboxes/{inbox_id}/messages/send   # send
  - GET  /v0/inboxes/{inbox_id}/messages        # list
  - POST /v0/inboxes/{inbox_id}/messages/{message_id}/reply  # reply
  - POST /v0/inboxes/{inbox_id}/webhooks        # webhook for inbound
---

# Send an email and handle the reply

Two-way email loop for an AI agent using AgentMail.

## Auth
- Send `Authorization: Bearer <am_...>` on every request. Base URL `https://api.agentmail.to`.
- Use an inbox-scoped key when the agent only needs one inbox (least privilege).

## Steps
1. **Create an inbox** — `POST /v0/inboxes`. Optionally set `username`, `display_name`, and custom `metadata` (up to 256 keys) to link the inbox to a record in your system. Save the returned `inbox_id` (prefix `inb_`).
2. **Send the first message** — `POST /v0/inboxes/{inbox_id}/messages/send` with `to`, `subject`, and `text`/`html`. Set an `Idempotency-Key` header so a retry never double-sends; the same key returns the original `message_id`/`thread_id` for 24 hours. Save `thread_id`.
3. **Receive the reply** — either register a webhook (`POST /v0/inboxes/{inbox_id}/webhooks` with event `message.received`) and verify the signature (see `conventions/`), or poll `GET /v0/inboxes/{inbox_id}/messages`.
4. **Reply in thread** — `POST /v0/inboxes/{inbox_id}/messages/{message_id}/reply` with your body. This keeps the conversation threaded. Use an `Idempotency-Key` again.

## Rules
- Handle `429 rate_limit_exceeded` by honoring `Retry-After` with exponential backoff.
- `403 domain_not_verified` means send from `@agentmail.to` or verify a custom domain first.
- Error envelope fields: `name`, `code`, `message`, `fix`, `docs` — see `errors/agentmail-error-codes.yml`.
