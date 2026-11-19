# 2. Modules, Not Bots

The first design instinct is a screen per bot. With six bots that is six screens, and each new bot is a project.

Instead, the portal knows a fixed set of **modules**:

| Module | What a team lead does with it |
|---|---|
| `hours` | Weekly hours, special dates, timezone |
| `announcements` | Temporary messages the agent should mention ("closed for the holiday on...") |
| `kb` | Knowledge entries the agent can use |
| `kb_approvals` | Approve or reject knowledge suggested from real conversations |
| `handoff` | Who receives handoffs, and when |
| `kill_switch` | Pause the agent immediately |
| `reports` / `requests` | Report a wrong answer, ask for a change, follow its status |
| `catalog` | Structured product data (for the catalog agent) |

## A bot is a row plus a list of modules

```text
bots            id, department, channel, language, status
bot_modules     bot_id, module, enabled, policy, schema
```

- `schema` describes the module's fields for that bot. The portal **renders the form from the schema**, so two bots can use the same module with different fields without new UI code.
- `policy` says how a change goes live: `admin` (an admin approves it) or `auto` (it applies directly). It is data, so it can change per bot and module without a deploy.

Adding a bot is an insert: one row in `bots`, a few rows in `bot_modules`. It appears in the portal for its department with exactly the screens it needs. All six bots were added that way.

## Module keys are an enum, on purpose

Module names are a database enum, not free text. A typo in a module name fails at insert time instead of producing a bot with a screen nobody can find. Adding a module is a migration, which is the right amount of friction for something that also needs UI and agent support.

## Approval policy as data

On day one, every change needed the admin's approval. That was the right default for a new system with customer-facing effects.

Within days, real use showed the case it was designed for. The product catalog (built later, with its own tables and versioning) had approval hard-wired into its submit function. The team lead made many small, correct edits, and each one waited for approval for no benefit. Removing it needed a migration, and the guard against mistakes moved into the editor: a **confirmation** when a price changes by more than 30%.

The lesson went back into the design: the right amount of control is not known on day one, so every area that can change state should read its approval rule from data. Moving the catalog onto the module policy is on the list.
