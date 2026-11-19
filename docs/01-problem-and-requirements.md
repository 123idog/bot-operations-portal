# 1. The Problem and the Requirements

## What it looked like before

Six AI agents across several departments: customer service, a product catalog with bookings, recruitment, and agents for other brands and languages. Each one was a workflow with prompts, tools and a few hundred lines of code.

Operational facts lived **inside the code**:

- Opening hours and holiday exceptions were constants in code steps, copied into every step that needed them (handoff decisions, the "we're closed" reply) and into every bot. A holiday meant editing several places.
- The product catalog was a set of PDFs edited by hand, sent to the engineer, uploaded, and run through an extraction workflow into a table the agent read. A price change needed two people and a manual run.
- Problem reports ("the bot told a customer X") often arrived as screenshots in chat, without the customer's number or time. Finding the conversation came before diagnosing it.

Every change went through one person. That person was the bottleneck, and also the single point of failure.

## Requirements

Written down before any design:

1. **Team leads manage their own department's agents.** Hours, announcements, knowledge, catalog, problem reports. Nobody sees another department.
2. **Every change can require approval**, at first all of them. That rule has to be loosened later per area without code changes.
3. **At least 4 to 5 agents**, each with a different mix of features. Adding one must not mean building a new screen.
4. **Three UI languages** (Hebrew, Arabic, English), right-to-left or left-to-right per user.
5. **The agents must not slow down or break** because of the portal. They read config on every customer message.
6. **Everything is audited**: who changed what, when, from which version.
7. **Long-term**: one engineer has to be able to run it.

## What I ruled out

| Option | Why not |
|---|---|
| Admin pages served by the workflow engine | The engine was already near its limits (memory, disk, concurrency). Adding interactive traffic to it would put customer conversations at risk |
| A spreadsheet as the config store | No permissions per department, no versions, no validation, and a slow read on the agents' hot path |
| One generic "settings" JSON per bot | Easy to build, impossible to permission or approve per area, and every edit risks the whole object |
| A document database | Would need its own API server, auth and permission layer, all built and secured by one person |

The next chapters describe what was built instead.
