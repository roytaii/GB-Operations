# Warehouse Inventory — Data Model

**Working reference for the food wholesale distributor engagement**
Worked example: one lot of frozen shrimp, from trailer arrival to customer delivery.

> **Status of this document.** Structure is ours; every *number* is an assumption
> until the walkthrough. Assumptions are marked ⚠. Do not let them harden into
> requirements.

---

## 1. The physical hierarchy

This is what actually moves. The data model exists to mirror it.

```
INBOUND TRAILER            arrives at the gate, gets a door or a yard spot
  └─ PALLET (LPN)          the license-plated unit — what a forklift moves
      └─ CASE              the sellable unit; may or may not be serialized
          └─ each/bag      inside the case; only matters if they break cases

LOCATION                   dock · slot · staging lane · yard spot · truck
  └─ holds pallets, and only pallets, at any moment in time
```

Two rules that fall out of this, and drive everything below:

- **A pallet is in exactly one location. A case is on exactly one pallet.**
  Anything else — SKU, lot, expiry, weight, cost — is an *attribute*, not a
  place. Attributes live on master records; places live in the event log.
- **The license plate is the join.** Scan one barcode on a pallet and you get
  its contents, its lot codes, its history, and where it is. That single fact is
  most of the value of the system.

### Where the levels come from

| Level | Created when | Destroyed when |
|---|---|---|
| Trailer | It shows up at the gate | It leaves |
| Inbound pallet | It comes off the trailer | It's fully picked / broken down |
| Case | Received as part of a pallet | Shipped, dumped, or damaged out |
| Outbound pallet | A picker starts building an order | It's delivered and signed for |
| Location | Once, when the warehouse is mapped | Effectively never |

---

## 2. A day in the data

What gets captured, when, by whom, and what paper it lives on **today**. This
table is the real deliverable of this document — the schema is downstream of it.

| Time ⚠ | Step | Who | Paper today | Record created |
|---|---|---|---|---|
| 05:00 | Trailer checks in at gate | Receiver / guard | Driver's BOL | `INBOUND_SHIPMENT` |
| 05:10 | Assigned door or yard spot | Receiver | Whiteboard | `EVENT: ARRIVED` |
| 05:15 | Seal broken, temp probed | Receiver | Receiving log | Temp + seal on shipment |
| 05:20–07:00 | Unload, pallet by pallet | Lumper / forklift | Packing list tally | `PALLET` + `RECEIVED` × N |
| — | Short / over / damaged / wrong | Receiver | Margin note on BOL | `EXCEPTION` |
| 07:15 | Signed BOL back to driver, departs | Receiver | BOL copy | `EVENT: DEPARTED` |
| 07:00–11:00 | Putaway to slots | Forklift | Nothing, or a scribble | `EVENT: PUTAWAY` |
| Ongoing | Cycle count, damage, dump, hold | Anyone | Sticky note | `COUNTED` `DAMAGED` `DISPOSED` `HOLD` |
| 13:00 | Orders released | Office | Printed pick tickets | `SALES_ORDER` + lines |
| 14:00–20:00 | Pick cases onto order pallets | Picker | Pick ticket check-marks | `PICKED` + new `PALLET` |
| 20:00 | Wrap, label, stage by route | Picker | Lane chalk | `STAGED` |
| 21:00–23:00 | Load route trucks | Loader | Load sheet | `LOADED` |
| 23:00 | Truck departs | Driver | BOL | `OUTBOUND_LOAD` + `DEPARTED` |
| Next AM | Deliveries, refusals, returns | Driver | Signed POD | `DELIVERED` `RETURNED` |
| End of day | Someone types all of it into Excel | Office | — | *this is the job we delete* |

**The through-line:** every row above is a moment where a person already knows
something true and writes it on paper. The system's only job at v1 is to catch
it there instead of at 9pm in a spreadsheet.

### Estimated daily volume ⚠

Sizing drives the grain decision in §4.3. Confirm all six numbers on site.

| Quantity | Assumed | Confirm by asking |
|---|---|---|
| Inbound trailers / day | 3–6 | "How many trucks do you receive?" |
| Pallets per trailer | 20–26 | "Floor-loaded or palletized?" |
| Cases per pallet | 40–70 | Ti/Hi on the top 10 SKUs |
| **Cases received / day** | **2,500–9,000** | — |
| Customer orders / day | 20–40 | "How many stops on a route? Routes?" |
| Lines per order | 15–60 | Show me yesterday's pick tickets |

Resulting event rows per day: **~600–1,200 at pallet grain, ~15,000–40,000 at
case grain.** Same warehouse, same trucks, 25× the labor and the data. That
ratio is the whole argument in §4.3.

---

## 3. Master data — the records that barely change

### 3.1 ITEM (one row per SKU)

| Field | Value |
|---|---|
| SKU | `SEA-SHR-1620-40` |
| Description | Shrimp, Raw, Shell-On Tail-On, Extra Jumbo, 16-20 ct |
| Brand | Waterfront Bistro |
| Category | Frozen Seafood |
| Pack Config | `20/2LB` (20 bags × 2 lb per case) |
| Net Weight | 40 lb |
| Gross Weight | 42.5 lb |
| Case Dimensions | 20 × 13 × 10 in |
| Ti/Hi | 6 × 10 (6 cases per layer, 10 layers = 60/pallet) |
| Temp Zone | Freezer (−10°F to 0°F) |
| Shelf Life | 24 months frozen |
| Catch Weight | No — fixed 40 lb, sold by case not actual lb |
| Reorder Point | 45 cases |
| Standard Cost | $178.40 / case |

**Brand ≠ vendor ≠ customer.** "Waterfront Bistro" is a private label. Three
separate fields. Conflating them is a common early mistake that gets expensive.

**Ti/Hi is not decoration.** It's how the system predicts a full pallet, catches
a miscount at receiving, and tells a picker whether an order fills a pallet.

### 3.2 LOCATION (the warehouse map)

The record type the original draft was missing, and the one that answers *"where
is it."* Nothing else in the system works until the building has addresses.

| Field | Example | Notes |
|---|---|---|
| Location ID | `FRZ-B-03-2` | Zone-Aisle-Bay-Level |
| Type | `SLOT` | `SLOT` `DOCK` `STAGING` `YARD` `TRUCK` `CUSTOMER` |
| Temp Zone | Freezer | Inherited by whatever sits here |
| Capacity | 2 pallets | Single-deep rack |
| Pickable | Yes | Reserve vs. forward pick face |
| Active | Yes | Blocked slots exist (leaking, broken beam) |

Types worth having on day one: `DOCK` (`FRZ-DOCK-01`), `STAGING` (route lanes),
`YARD` (parking spots — the hook for the trailer-parking module in CLAUDE.md
§1), `TRUCK`, `CUSTOMER`. Making receiving, staging, and the yard *real
locations* rather than statuses means one query answers "where is it" for
everything in the building, including the things sitting on the floor.

> **Walkthrough question that gates this whole table:** are there labels on the
> racks today, or does everyone just know? If there are no labels, step one of
> the build is a weekend with a label printer, not software.

### 3.3 LOT (one row per production lot received)

| Field | Value |
|---|---|
| Lot ID | `LOT-2026-08-0441` |
| Traceability Lot Code | `VN26158-A3` (supplier's TLC — carried forward, never renamed) |
| SKU | `SEA-SHR-1620-40` |
| Vendor | Pacific Rim Seafood Imports (`VEN-1042`) |
| PO Number | `PO-88214` |
| Pack Date | 2026-06-07 |
| Best By | 2028-06-07 |
| Country of Origin | Vietnam |
| Production Method | Farm-raised |
| Qty Received | 1,440 cases (24 pallets × 60) |
| Qty On Hand | *derived from event log* |
| Receiving Temp | −8°F ✓ within spec |

**Why this level matters:** recalls run on lot codes, not case IDs. Rotation is
FEFO (first-expired-first-out), which is a lot-level decision. Two identical
cases of the same SKU are *different inventory* if their lot codes differ, and
picking the wrong one is how you ship expired product.

---

## 4. Operational data — the records created every day

### 4.1 INBOUND SHIPMENT (the trailer)

| Field | Value |
|---|---|
| Shipment ID | `TRL-4471` |
| Carrier | Coastline Refrigerated |
| Trailer / Container # | `CRLU-772841` |
| Seal # | `SL-0099412` — intact on arrival ✓ |
| Driver | S. Whitfield |
| PO(s) on board | `PO-88214`, `PO-88220` |
| Appointment | 2026-08-16 05:00 |
| Arrived | 2026-08-16 05:07 |
| Assigned to | `FRZ-DOCK-01` |
| Unload start / end | 05:22 / 06:58 |
| Departed | 07:14 |
| Trailer temp on open | −8°F ✓ |
| Pallets expected / received | 24 / 24 |
| Cases expected / received | 1,440 / 1,437 (3 short — see exceptions) |

**Expected vs. received on the same row.** The variance *is* the record. If the
system only stores what was received, nobody can ever prove a vendor shorted
them, and the vendor-claim conversation stays a memory contest.

**Arrival, door assignment, and departure timestamps are the trailer-parking
module,** already paid for by receiving. Yard spots as locations + these four
timestamps = dwell time per carrier, door utilization by hour, and which
carriers are chronically late. That is priority #2 delivered as a byproduct of
priority #1, which is a good thing to be able to say in the meeting.

### 4.2 PALLET / LPN (the unit that moves)

| Field | Value |
|---|---|
| LPN | `LPN-100417` |
| Type | Inbound / received |
| Source | `TRL-4471` |
| Contents | 60 cases · `LOT-2026-08-0441` · `SEA-SHR-1620-40` |
| Built | 2026-08-16 05:41 |
| Current Location | *derived from event log* |
| Status | *derived from event log* |
| Hold Flag | None |

A pallet's contents are **lines, not a single value** — one line per (SKU, lot).
Inbound pallets are usually single-SKU single-lot; outbound order pallets are
always mixed. Same table, one to many lines.

| LPN | SKU | Lot | Cases |
|---|---|---|---|
| `LPN-200884` | `SEA-SHR-1620-40` | `LOT-2026-08-0441` | 12 |
| `LPN-200884` | `PRO-ONI-YEL-50` | `LOT-2026-08-0512` | 4 |
| `LPN-200884` | `DRY-RIC-JAS-25` | `LOT-2026-07-0388` | 30 |

**Status is computed, never stored.** A status field that someone updates by
hand drifts out of sync with reality inside a week. Derive it from the most
recent event. Same for current location and on-hand quantity.

### 4.3 CASE — optional, and the biggest open question in this document

| Field | Value |
|---|---|
| Case ID | `CS-000184` |
| Parent LPN | `LPN-100417` |
| Lot ID | `LOT-2026-08-0441` |
| Catch Weight | n/a (fixed 40 lb) |

Serializing every case means someone prints and applies a label to **2,500–9,000
cases a day, every day, forever.** ⚠ That is at minimum a full-time position, in
a freezer, where label adhesive fails and gloves don't work touchscreens.

Lot-level tracking inside a pallet is sufficient for recalls, FEFO, and vendor
claims. Case serialization only earns its keep if they need to distinguish two
cases *from the same lot* — realistically only for catch-weight product, where
each case genuinely has a different weight and therefore a different price.

> **Ask directly:** *Do you need to know which specific case, or just how many
> cases of this lot on this pallet?*

**Recommended default:** pallet-level license plates, lot-level quantities,
case-level serialization enabled per-SKU for catch-weight items only. The
prototype implements exactly this — the case table exists and is optional.

### 4.4 OUTBOUND LOAD

| Field | Value |
|---|---|
| Load ID | `LOAD-08-17-B` |
| Truck | `TRUCK-255` — refrigerated, setpoint −10°F |
| Route | `RT-08-17-B` |
| Driver | mreyes |
| Pallets | 14 |
| Stops | `CUST-2501`, `CUST-2588`, `CUST-2601` |
| Departed / Returned | 2026-08-17 23:26 / 2026-08-18 09:40 |

---

## 5. The event log

One append-only table. Same column shape on every row, so on-hand is
`SUM(qty_delta)` rather than a number someone has to remember to change.

```
timestamp | event_type | entity_type | entity_id | qty_delta | from_loc | to_loc | user_id | ref_doc | note
```

### Worked example — one pallet of shrimp

| Timestamp (PDT) | Event | Entity | Δ cases | From | To | User | Ref |
|---|---|---|---|---|---|---|---|
| 2026-08-16 05:07 | ARRIVED | `TRL-4471` | — | gate | `FRZ-DOCK-01` | jsantos | BOL-CR-5581 |
| 2026-08-16 05:41 | RECEIVED | `LPN-100417` | +60 | `TRL-4471` | `FRZ-DOCK-01` | jsantos | PO-88214 |
| 2026-08-16 06:12 | PUTAWAY | `LPN-100417` | 0 | `FRZ-DOCK-01` | `FRZ-B-03-2` | akim | — |
| 2026-08-17 14:22 | PICKED | `LPN-100417` | −12 | `FRZ-B-03-2` | `LPN-200884` | rlopez | SO-40912 |
| 2026-08-17 20:15 | STAGED | `LPN-200884` | 0 | pick floor | `STG-LANE-B` | rlopez | SO-40912 |
| 2026-08-17 22:56 | LOADED | `LPN-200884` | 0 | `STG-LANE-B` | `TRUCK-255` | jsantos | BOL-77301 |
| 2026-08-17 23:26 | DEPARTED | `LOAD-08-17-B` | — | `TRUCK-255` | route | mreyes | BOL-77301 |
| 2026-08-18 06:14 | DELIVERED | `LPN-200884` | −12 | `TRUCK-255` | `CUST-2501` | mreyes | POD-77301 |

**Derived state:** `LPN-100417` — 48 cases remaining, slot `FRZ-B-03-2`, status
Available. `LOT-2026-08-0441` — 1,425 cases on hand across 24 pallets.

### Design points

- **Every event carries a location.** Location implies temperature zone, which
  is how you prove the cold chain held. Receipt lands at a freezer dock, not in
  a limbo state called "received."
- **Timestamps include a timezone.** Non-negotiable.
- **User IDs, not names.** There will be two Johns.
- **Every event references a document** where one exists (PO, SO, BOL, POD).
  This is what makes the log auditable and a vendor claim winnable.
- **Container events don't fan out.** Loading a 14-pallet truck writes 14 rows,
  not 800. Cases inherit through parentage. Flatten it into one timeline for
  display — good UX — but don't *store* it flat, or a pallet pulled off the
  truck at the last second makes the log lie.
- **Picking is a transformation,** not a move: cases leave one LPN and join
  another. It's the only common event where a case changes parent, and it's
  also a regulated tracking event (§7).

### Event vocabulary

Inbound: `ARRIVED` `RECEIVED` `PUTAWAY`
Outbound: `ALLOCATED` `PICKED` `STAGED` `LOADED` `DEPARTED` `DELIVERED`
Corrections: `MOVED` `COUNTED` `ADJUSTED` `DAMAGED` `DISPOSED` `RETURNED`
`HOLD_PLACED` `HOLD_RELEASED`

---

## 6. Exceptions — the data that only exists when something goes wrong

Currently this data is a margin note on a BOL, and it evaporates. It's also the
data with the highest dollar value per row in the entire system.

| Exception | Captured at | Answers |
|---|---|---|
| Short / over shipment | Receiving | Vendor credit claims |
| Damaged on arrival | Receiving | Carrier claim, who pays |
| Temp out of spec | Receiving | Reject or accept-under-protest |
| Wrong item shipped | Receiving | Vendor performance |
| Slot mismatch at pick | Picking | Where the drift started |
| Cycle count variance | Counting | Shrink, by zone and by SKU |
| Damaged in warehouse | Anytime | Internal shrink vs. vendor shrink |
| **Pallet received, never put away** | Derived | The pallet sitting on the dock at 6pm |
| Customer refusal / return | Delivery | Credits, re-stock vs. dump |

That bolded row is worth calling out in the meeting. "Received but never put
away" requires **zero new data entry** — it's the absence of a second event — and
it catches the single most common way inventory goes missing in a paper
warehouse. It is the cheapest feature in the system and possibly the most
valuable. The prototype implements it.

The theft question in CLAUDE.md §3 resolves here too: you don't detect theft
with a theft feature. You detect it because every case has a last known location,
a last known handler, and a timestamp, and one of those three stops making sense.

---

## 7. Regulatory hook — verify, don't assume ⚠

FDA's Food Traceability Rule (FSMA §204) requires records for **Critical
Tracking Events** on foods on the **Food Traceability List**. Shrimp is a
crustacean; crustaceans are on the FTL. For a distributor the relevant CTEs are
**receiving, shipping, and transformation** — which map exactly onto `RECEIVED`,
`LOADED`/`DEPARTED`, and `PICKED` above. That alignment is not a coincidence; it
is why the event log is shaped this way.

The required Key Data Elements — traceability lot code, quantity and UOM,
location identifiers, dates, and the reference document — are the columns already
in §5.

**Two things to verify before treating any of this as a requirement:**

1. **The compliance date has moved.** FDA has proposed/announced an extension
   past the original January 2026 date. Check the current status rather than
   quoting a deadline in the meeting.
2. **Which of their SKUs are actually on the FTL.** Seafood, soft cheeses, fresh
   leafy greens, cut fruit, shell eggs, nut butters, and some produce are; most
   dry grocery is not.

This is a talking point, not a scare tactic. If they're already keeping lot
codes on paper for this reason, the system is a compliance upgrade they were
going to have to fund anyway. Confirm with whoever owns food safety there.

---

## 8. Known gaps

**Allocation.** `SO-40912` appears in the log at pick time, but nothing ties
inventory to an order line *before* picking. Without it you can't answer "what's
promised but not yet shipped," which is the difference between on-hand and
available-to-promise. Needed before anyone trusts the numbers to sell against.

**Partial cases.** The model assumes whole cases. The moment someone breaks a
case for a small restaurant order, quantity has to express bags-within-case.
Pack config (`20/2LB`) is there to support it; the event schema is not yet.
**Ask whether they break cases at all** — many wholesalers refuse to, and if so
this gap closes for free.

**Catch weight.** Flagged `No` for shrimp, but if they carry meat, cheese, or
produce sold by actual weight, every case of a SKU has a different weight and
therefore a different price. This is the one requirement that reshapes the
schema significantly, and it's the one real argument for case serialization.
**Ask early.**

**Returns and credits.** `RETURNED` is in the vocabulary but there's no model
for what happens next — restock, re-inspect, dump, or credit — and it touches
A/R, which is explicitly out of scope. Capture the event; defer the workflow.

**Multi-warehouse.** Assumes one building. If there's a second site or an
overflow freezer, location IDs need a facility prefix from day one. Cheap now,
painful later. **Ask.**

**Units of measure.** Cases here throughout. Real orders arrive in cases,
pounds, and eaches, sometimes for the same SKU. Conversion factors belong on
ITEM. Not modeled yet.

---

## 9. Questions this document is designed to force

Ranked by how much schema hangs on the answer.

1. **Catch weight — yes or no?** Reshapes the case table and the pricing model.
2. **Case-level or pallet-level tracking?** Collapses or keeps an entire table
   and a full-time labeling job. (§4.3)
3. **Are the racks labeled today?** Gates every "where is it" feature. (§3.2)
4. **Do you break cases?** Closes or opens the partial-case gap.
5. **How do you pick which case of a SKU?** Does anyone check dates today — is
   FEFO real, or aspirational?
6. **Is there wifi in the freezer?** Determines online vs. offline-first, which
   is an architecture decision, not a feature.
7. **What happened the last time something went really wrong?** The most
   valuable question in the list, and the least likely to be answered by a spec.

---

## Appendix — how the prototype maps to this document

`prototype/index.html` implements §1–§6 with seeded demo data. Deliberately
omitted: allocation, partial cases, catch weight, pricing, returns workflow. Its
job is to make "scan a pallet, see everything" concrete in about ten seconds,
not to be v1.
