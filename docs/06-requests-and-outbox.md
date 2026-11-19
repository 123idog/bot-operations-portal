# 6. Requests and the Outbox

## Problem reports that can be diagnosed

Before the portal, a report was a screenshot in a chat: "the bot answered something weird". The first hour went to finding the conversation.

A report in the portal requires, **enforced by the database function, not the form**:

| Field | Why |
|---|---|
| The bot | Reports are routed and permissioned per bot |
| The customer's phone (WhatsApp bots) or a customer reference (other channels) | To pull the exact conversation |
| When it happened (pre-filled with now) | To find the right turn |
| Screenshots (up to 5, paste with Ctrl+V) | What the team lead saw |

Phone numbers are normalized in the database: digits only, international format. A local number without a country code is accepted only where the country is unambiguous for that bot's market. Otherwise the function rejects it with a clear error instead of guessing. A wrong guess sends the diagnosis to the wrong customer.

Requests have a status, comments and a full event history. Team leads see their own department's requests. The design keeps the older chat-based request bot as a second channel into the same store, so nobody has to change habits on day one.

## The outbox

Some changes need something to happen elsewhere: notify the admin of a pending approval, start an AI triage of a new problem report, check an agent after its config changed.

The portal **never calls the workflow engine directly**. Each function that changes state also writes an event to an `outbox` table in the same transaction:

```text
outbox   id, event_type, payload, created_at, delivered_at, attempts, last_error
```

A worker on the workflow side is meant to read undelivered events, handle them and mark them delivered. Today the portal writes every event; the consumers are being added one at a time. The design's consequences:

- If the workflow engine is down, nothing in the portal fails. Events wait.
- An event exists **only if the change committed**. No notification about an approval that rolled back.
- Retrying is safe: the worker records attempts and the last error, and handlers are idempotent per event id.

## AI triage (next)

The point of collecting phone and time is that a triage job can fetch the conversation around that moment, look at what the agent saw and said, and attach a first diagnosis to the request ("the agent answered from an outdated announcement", "the customer's message arrived while the agent was paused"). The person who opens the request starts from evidence, not from a screenshot.
