# Prototype — how to run it and how to demo it

## Run

Open `prototype/index.html` in any browser. Double-click it. That's the whole
install.

No server, no build step, no dependencies, no network. One file. It works on a
laptop, a tablet, or a phone, online or off. Data persists in `localStorage`;
**Reset demo data** puts it back to the seeded warehouse.

## What this is

A working model of §1–§6 of [`../inventory-data-model-example.md`](../inventory-data-model-example.md):

- **Trailer → pallet → cases (by lot) → slot**, with a real warehouse map
- **One append-only event ledger.** On-hand, current location, and status are
  all computed from it. There is no stored quantity anywhere in the code, which
  is the single most important thing to show.
- **Universal search.** One box over pallets, cases, lots, SKUs, locations,
  trailers, orders, customers, vendors, loads. Everything is clickable, and
  every record links to the records around it.

Seeded with 5 trailers, ~85 pallets, 22 lots, 16 SKUs, 214 rack slots, and
~220 events — including the exceptions that make the point.

## What it deliberately is not

No allocation, no partial cases, no pricing, no catch-weight billing, no
returns workflow, no users or permissions, no printing, no scanner integration,
no backend. Those are §8 of the data model — the known gaps. Don't demo around
them; name them if asked.

`localStorage` is a stand-in for a real database. The derive layer (`D()`,
`palletState()`, `lotState()`) is the part written to survive that swap.

## 90-second demo path

Run it in this order. Each step answers a question they actually have.

1. **Today** — "This is 5 trailers and 4,000 cases. Every number on this screen
   is computed, not typed."
2. Click **TRL-4471** → *Everything that came off this trailer.* One trailer,
   24 pallets, where each one is right now. **This is the whole pitch.** One
   pallet is flagged `NOT PUT AWAY`.
3. Click that pallet (**LPN-100449**) → received 12 hours ago, one event, never
   moved. "It's on your freezer dock right now. Nobody knows, because nothing
   was written down that says it isn't in a rack."
4. **Search** — type `shrimp`, then a slot like `FRZ-B-03`, then a lot number.
   "Same box for everything. Scan a label, get the record."
5. Open any **lot** → *Recall drill.* "If your supplier calls at 4pm about this
   lot code, this is the list — where it is, and who already got it."
6. **Warehouse map** — the building as it actually is. Click any cell.
7. **Exceptions** — "This screen is the reason to build the ledger. Nothing here
   was typed by anyone; it's all the absence or the shape of events."
8. **Put away / move** — do one live. Two fields. Then try putting a freezer
   pallet in a dry slot and let it warn.

Optional, if they ask "how hard is data entry": **Receive** — one pallet, four
fields, and the license plate is assigned by the system. Nobody types an LPN.

## Things to say out loud

- **It's a prototype, built to be argued with.** Every field in it is a guess
  until the walkthrough. The point is to give them something concrete to
  disagree with, which is much faster than asking them to describe their process
  from a blank page.
- **The event log is the actual product.** Everything else is a view.
- **Nothing is ever edited or deleted** — corrections are new rows. That's what
  makes it an audit trail rather than a spreadsheet with better fonts.

## Things NOT to say

Per `CLAUDE.md` §4: no dates, no prices. *"I want to write this up properly
first"* is a complete answer.

## Code map

Single file, roughly in this order:

| Section | What it does |
|---|---|
| tokens / CSS | light + dark, dense industrial layout |
| `ET` | event vocabulary — which events move a pallet |
| `seed()` | deterministic demo warehouse |
| derive layer | `linesOf` `palletState` `lotState` `itemState` `exceptions` |
| `D()` | derived-state cache, rebuilt from scratch on every append |
| `index()` / `search()` | the universal search index |
| `VIEWS` | one function per screen |
| `DETAIL` | one function per record type — the drawer |

The rule the code follows: **anything that can be derived, is.** If you find
yourself adding a stored `status` or `qty_on_hand` field, that's the bug.
