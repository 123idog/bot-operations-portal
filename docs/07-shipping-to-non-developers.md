# 7. Shipping to Non-Developers

The users are team leads, not engineers. Most of the work after the schema was making the system safe and obvious for them.

## Three languages, one layout

The UI language is a per-user setting (Hebrew, Arabic, English). Direction follows the language, and the layout uses CSS **logical properties** (`margin-inline-start`, `padding-inline-end`) everywhere, so right-to-left is not a second stylesheet. Arabic gets its own font fallback. The browser tab title follows the language too.

## Operational copy, not marketing copy

An early version used slogans in headings. The feedback was direct: this is an operations tool. Headings became labels ("Bot overview", "Pending approvals"), and every screen states its rule in plain words ("Changes take effect after admin approval").

## A sandbox bot

A real-looking test bot in its own department, with every module and no customers. It is used to:

- demo the portal to a new team lead without touching a live agent;
- run an end-to-end test as a team lead: change the hours, approve, and confirm through "what the bot sees" that the runtime is closed until the right moment;
- test the agent-side integration with a workflow that reads the sandbox bot's runtime with its own key.

Releases are checked there first: hours change, special closed date, kill switch, a problem report with a pasted screenshot and a fake phone number, a status change and a comment. Then the outbox is checked for one event per action.

## Data first: the catalog

The catalog agent used to send customers a PDF per product, designed by hand. Prices in the PDF and prices in the agent's table drifted apart.

Now the team lead edits **structured data**: a list of price rows (not just "adult/child", because real products have per-vehicle prices, age bands and add-ons), schedule, inclusions, notes and photos. The brochure PDF is **generated** from that data with one template, and the same structured data is what the agent's catalog is built from. One source, so the agent and the brochure stop disagreeing.

Details that came from real use:

- An empty price row printed "Price: free" on a brochure. Empty rows are now dropped on the server, not only in the form.
- A content edit waiting for approval while the download button served the live version confused the team lead ("it gives me the old PDF"). The editor now shows both: the live PDF and the pending one, with a banner explaining which one customers receive.
- Team leads can create a product as a **draft**, including in a new destination, copy an existing product as a starting point, and delete a draft that was never published. Publishing requires at least a price (or "price on request") and a generated PDF.

## Loosen rules from evidence

The system launched with approval on everything. Within days, the catalog's team lead was making many small, correct edits, each waiting on approval. Approval for catalog edits was removed, and the guard against mistakes moved to where mistakes happen: a confirmation when a price changes by more than 30% ([chapter 2](02-modules-not-bots.md)).

Start strict, watch real use, and move each control to the place where it catches real errors.
