# 4. The Runtime Contract

The most important interface in the system is not the portal UI. It is the call every agent makes on every customer message.

## One call, everything resolved

```text
GET  bot_runtime(bot_id)        header: x-bot-key

→ {
    "hours": { ...weekly, specials, timezone },
    "announcements": [ ... ],
    "kill_switch": { "paused": false },
    ...
    "_runtime": {
      "paused": false,
      "versions": { "hours": 5, "announcements": 2 },
      "hours": {
        "local_now": "2026-11-17T19:40",
        "is_open_now": false,
        "closes_at": null,
        "next_open": "2026-11-18T09:00",
        "today": { "windows": [], "name": "Special closing", "source": "override" }
      }
    }
  }
```

The agent does not compute anything about time. Before, "are we open?" was implemented separately in each bot, with the hours copied into several code steps, so every holiday meant editing several places and hoping none was missed. Now it is computed once, in the database, in the business's timezone:

1. A special date for this bot (closed, or different hours) wins.
2. Then a global holiday.
3. Then the weekly schedule.
4. If closed, scan forward up to 14 days for the next opening.

The response says which rule applied (`source`) and which version of each module is live, so when an agent says "we're closed" on a normal Tuesday, the reason is visible in one field.

## Per-bot keys, not the admin key

Agents do not use a service-role key. Each bot gets its own key:

- Only the **SHA-256 hash** is stored. The table has row-level security on and **no policies**, so no client can read it.
- The runtime function compares the hash, checks that the key belongs to the requested bot, records `last_used_at`, and returns that bot's config only.
- A wrong key, another bot's key and no key all return the same error.

A leaked key exposes one bot's read-only, non-secret config. It cannot read requests, users or other bots, and it cannot write anything.

## The hot path stays cheap

The call is one primary-key read of the materialized `live_config` plus some arithmetic. It never touches versions, history or requests. Admin activity in the portal has no effect on it.

## Fallback (part of the contract, not optional)

An agent must keep the last good response as a snapshot, and use it if the call fails, logging the failure. The portal being down must never mean an agent that tells customers the wrong hours or stops answering. This is a requirement every agent has to meet before it switches from its hard-coded config to the runtime call.

## "What the bot sees"

Admins have a preview tab on each bot that calls the same function and shows the exact JSON the agent would get right now. A question like "why did the bot say we're closed?" starts there, not in the agent's logs.
