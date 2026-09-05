# One Sales Order, Start to Finish

Follows a single sales order through every module, every person who touches it,
and what changes at each step.

Companion to [module_plan.md](module_plan.md) and [po_walkthrough.md](po_walkthrough.md).

---

## The short version

```
SO STATUS       MODULE                      WHO
Draft        →  Sales Orders                Salesperson
Confirmed    →  Sales Orders                Salesperson
Released     →  Shipments (outgoing)        Office
Picked       →  Shipments (outgoing)        Picker
Loaded       →  Shipments (outgoing)        Loader
Delivered    →  Shipments (outgoing)        Driver
Invoiced     →  Invoices                    Clerk
Paid         →  Accounts Receivable         Clerk
```

Running example: Joe's Diner orders 10 cases of chicken wings and 5 of shrimp for
Tuesday. Sales order #5512.

---

## Step 1 — Customer places an order

**Who:** Salesperson
**Opens:** Sales Orders (new), pulling from **Item Master** (what they're buying)
and **Customers & Vendors** (who they are, their price, their terms, where to deliver)
**Does:** Enters what Joe wants and when he wants it.
**Creates:** SO #5512. Status = **Draft**.
**Feeds:** Step 2

> Note that Joe doesn't get the same price as everyone else. The price comes off
> his record in Customers & Vendors, not off a single master price list.

## Step 2 — Check the customer and the stock

**Who:** Salesperson (the system does the checking)
**Opens:** Sales Orders, reading **Customers & Vendors** and **Inventory**
**Does:** Two checks before this becomes real — is Joe within his credit limit,
and is there actually enough on hand to fill it.
**Changes:** Status = **Confirmed**. Or it stops here on a credit hold, or part of
it becomes a backorder.
**Feeds — the mirror of a purchase order, it feeds three places at once:**

* **A/R** — as money we *expect* to collect. Not a bill yet. Nothing is owed until Joe actually receives product.
* **Inventory** — the cases get marked *promised to someone*, so a second salesperson can't sell the same shrimp an hour later.
* **Parking / Dock Scheduling** — as a truck going out on a date.

## Step 3 — Route it

**Who:** Dispatcher / office
**Opens:** Sales Orders and **Parking / Dock Scheduling**
**Does:** Assigns the order to a truck, a driver, and a position in the day's stop
sequence. Joe's Diner is stop 4 on Dave's Tuesday route.
**Changes:** The order now carries a route, a driver, and a stop number.
**Feeds:** Shipments (outgoing), Parking / Dock Scheduling

## Step 4 — Release it to the warehouse

**Who:** Office
**Opens:** Shipments (outgoing)
**Does:** Prints the pick ticket and turns the order loose on the floor.
**Changes:** Status = **Released**.
**Feeds:** Step 5 — the warehouse now has work to do.

## Step 5 — Picking

**Who:** Picker
**Opens:** Shipments (outgoing) on a handheld, reading **Inventory** for locations
**Does:** Walks the racks pulling cases, scanning each one.
**Creates:** A record of what was **actually** picked.
**Changes:** Status = **Picked**. Each case's status in Inventory changes from
promised to picked.
**Feeds:** Inventory, and exceptions if the shelf came up short.

> **This is where "ordered" and "picked" split apart** — the outbound twin of
> ordered vs. received on the buying side. Joe ordered 10 cases of wings; there
> were only 7. Everything downstream has to use the picked number, because that
> is what he can actually be billed for.

> Which case gets picked matters too. If the picker takes the newest case instead
> of the oldest, the aging report will faithfully report a problem that the
> picking process is creating.

## Step 6 — Check and QC

**Who:** Checker
**Opens:** Shipments (outgoing)
**Does:** A second person verifies the pile before it goes near a truck — right
items, right count, right temperature, nothing damaged.
**Feeds:** Step 7

> This is the step that can disappear. If the scans in step 5 are reliable, the
> scanner has already proved the right cases were pulled and the manual check
> stops earning its keep. That is a whole job's worth of labor, and it is one of
> the clearest places the system pays for itself.

## Step 7 — Stage by route

**Who:** Picker / warehouse
**Opens:** Shipments (outgoing)
**Does:** Moves the finished order to the staging lane belonging to Dave's truck.
**Changes:** The cases' location changes to a staging lane — still ours, still in
the building, but no longer in the racks.
**Feeds:** Inventory

## Step 8 — Load the truck

**Who:** Loader
**Opens:** Shipments (outgoing) and **Parking / Dock Scheduling**
**Does:** Scans cases onto the truck, loaded in reverse stop order so the first
stop is nearest the door.
**Changes:**

* **Inventory on-hand goes down** — this is the only moment stock decreases
* Status = **Loaded**
* Parking marks the truck as leaving the yard

**Feeds:** Inventory, Parking / Dock Scheduling

> The exact mirror of *Receive* on the buying side. Stock goes up in exactly one
> place and comes down in exactly one place. Everywhere else it only moves.

## Step 9 — Deliver the route

**Who:** Driver
**Opens:** Shipments (outgoing) on a phone or tablet
**Does:** Drops at each stop and collects a signature.
**Creates:** Proof of delivery — who signed, when, and what they accepted.
**Changes:** Status = **Delivered**.
**Feeds:** Invoices

> The signature is the whole ballgame. Without it, Joe can claim three weeks
> later that the shrimp never arrived, and there is no way to argue.

## Step 10 — Something went wrong *(only if needed)*

**Who:** Driver flags it; Salesperson or Manager owns it
**Opens:** Shipments (outgoing) / Sales Orders / Accounts Receivable
**Does:**

* Short pick → bill for 7 cases, not 10
* Joe refuses it at the door — too warm, damaged, wrong item → it comes back on the truck
* Substitution he didn't agree to → credit
* Returned product → decide whether it goes back on the shelf or gets destroyed

**Changes:** The invoice amount drops; returned product either re-enters inventory
or gets written off.
**Feeds:** Inventory, A/R, Reports

## Step 11 — Driver close-out

**Who:** Driver, with the office
**Opens:** Shipments (outgoing)
**Does:** End of day. What got delivered, what came back on the truck, any cash
collected, anything that went wrong.
**Changes:** The route is closed and reconciled — truck against paperwork.
**Feeds:** Invoices, Inventory (returns), A/R

## Step 12 — Invoice the customer

**Who:** Clerk
**Opens:** Invoices, pulling up the sales order and the delivery record
**Does:** Bills Joe for what was **delivered and signed for** — not what he
originally ordered.
**Changes:** Status = **Invoiced**.
**Feeds:** Accounts Receivable

> The same rule as the three-way match on the buying side, pointed the other
> direction. Over there the receipt wins; over here the signed delivery wins.
> Billing off the order instead of the delivery is how you end up in an argument
> you cannot win.

## Step 13 — Collect the money

**Who:** A/R clerk, Manager chases the late ones
**Opens:** Accounts Receivable
**Does:** Records the payment against terms — Net 30 from the delivery date, with
a target of collecting inside 30 days.
**Changes:** Status = **Paid**. The receivable clears.
**Feeds:** Reports

## Step 14 — It shows up in the numbers

**Who:** Manager / owner
**Opens:** Reports
**Sees:**

* Gross margin on this order — what Joe paid, minus what the product cost
* A/R aging: how long Joe took to pay
* Fill rate: how often orders go out short
* Whether Joe is ordering more, less, or has quietly stopped

**Feeds:** Back to **Step 1**. The salesperson sees which customers went quiet and
calls them — and the loop closes.

> Margin only works because the cost came from the **buying** cycle. The price
> comes from this walkthrough; the cost comes from Receiving in
> [po_walkthrough.md](po_walkthrough.md). Neither cycle can report profit alone.

> This also feeds Step 1 of the purchase order cycle — what sold this week is
> what the buyer looks at to decide what to buy next week. The two loops turn
> together.

---

## Questions this raises for GB

* Who actually does each of these jobs today? How many are the same person?
* Who is allowed to release an order for a customer who's over their credit limit?
* When you're short on an item, who decides — ship partial, hold the whole order, or substitute? Does anyone call the customer first?
* Does a driver ever sell off the truck, or take a new order at the door?
* Does a customer ever pay the driver in cash? What happens to that money?
* How are routes built today — by hand, by habit, or by software?
* When product comes back on the truck, who decides whether it goes back on the shelf or gets thrown out?
* Do you invoice per delivery, or send a statement at the end of the month?
* Does the customer sign paper today, and where does that paper end up?
