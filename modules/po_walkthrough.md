# One Purchase Order, Start to Finish

Follows a single PO through every module, every person who touches it, and what
changes at each step.

Companion to [module_plan.md](module_plan.md).

---

## The short version

```
PO STATUS     MODULE                       WHO
Draft      →  Purchase Orders              Buyer
Sent       →  Purchase Orders              Buyer
Expected   →  Parking / Dock Scheduling    Gate
Received   →  Receiving (incoming)         Receiver
Matched    →  Accounts Payable             Clerk
Paid       →  Accounts Payable             Owner
```

---

## Step 1 — Decide what to buy

**Who:** Buyer
**Opens:** Sales Orders (order history)
**Does:** Looks at what's been selling to decide what's running low.
**Creates:** Nothing yet — just a decision.
**Feeds:** Step 2

> Later this gets better: Reports can show *on hand + already on order − what's
> selling* and suggest the buy. For now, the buyer eyeballs sales history.

## Step 2 — Write the purchase order

**Who:** Buyer
**Opens:** Purchase Orders (new PO), pulling from **Item Master** (what to order)
and **Customers & Vendors** (who from, their price, their terms)
**Does:** Enters items, quantities, agreed price, expected delivery date.
**Creates:** PO #1042. Status = **Sent**.
**Feeds — and this is the important part, it feeds three places at once:**

* **A/P** — as money we *expect* to owe. Not a bill yet. Nothing is owed until the product physically arrives.
* **Inventory** — as an "on order" quantity, so the buyer doesn't order the same chicken again next Tuesday.
* **Parking** — as a truck expected on a date.

## Step 3 — Vendor confirms and ships

**Who:** Buyer or office
**Opens:** Purchase Orders
**Does:** Marks the PO confirmed, updates the arrival date if the vendor changed it.
**Changes:** PO status = **Expected**.
**Feeds:** Parking / Dock Scheduling — now there's a real appointment on a real day.

## Step 4 — Truck arrives at the gate

**Who:** Gate / yard person
**Opens:** Parking / Dock Scheduling
**Does:** Checks the truck in, assigns it a door.
**Changes:** Truck marked arrived.
**Feeds:** Receiving (incoming) — the dock now knows PO #1042 is sitting at door 3.

## Step 5 — Receiving checks the truck

**Who:** Receiver
**Opens:** Receiving (incoming), which pulls up PO #1042 automatically
**Does:** The three-way look — what we ordered (PO), what the driver's paperwork
says (BOL), and what's actually in the trailer. Then QC: count, weight,
temperature, condition, dates.
**Creates:** A receipt record holding the **actual** quantities.
**Feeds:** Exceptions, if anything doesn't match.

> **This is where "ordered" and "received" split apart.** We ordered 5 pallets;
> 4 arrived and one is 8°F too warm. From here on, every module uses the
> *received* number, never the ordered number. On paper these two get blurred
> together — keeping them separate is most of what the system is for.

## Step 6 — Barcode and location

**Who:** Receiver
**Opens:** Receiving (incoming)
**Does:** Labels every case or pallet, assigns a warehouse location.
**Creates:** The individual case records — the first moment a physical box becomes
something the computer can follow.
**Feeds:** Inventory

## Step 7 — Receive it into inventory

**Who:** Receiver
**Opens:** Receiving (incoming) — presses **Receive**
**Does:** Commits the receipt.
**Changes:**

* Inventory on-hand goes **up** (this is the only moment stock increases)
* That "on order" quantity from step 2 goes away
* PO status = **Received** (or *Partially Received* if some is still coming)
* Two clocks start: the **aging clock** on the product, and the **payment terms clock** on the money

**Feeds:** Inventory, A/P

## Step 8 — Put it away

**Who:** Warehouse worker on a forklift
**Opens:** Inventory (handheld scanner)
**Does:** Scans the case, scans the location, drops it.
**Changes:** Each case's location is updated, and *who moved it* is logged.
**Feeds:** The audit trail — this is what answers "where is it / who touched it" later.

## Step 9 — The vendor's invoice arrives

**Who:** A/P clerk
**Opens:** Accounts Payable, which pulls up both the PO and the receipt
**Does:** The **three-way match**:

| | |
|---|---|
| PO says | what we agreed to buy |
| Receipt says | what actually showed up |
| Invoice says | what they're charging us |

All three should agree. When they don't, the receipt wins.
**Creates:** An approved payable — or a discrepancy that stops the payment.
**Feeds:** Step 10 if there's a problem, Step 11 if there isn't.

> This single check is what stops GB from paying for chicken that never arrived.
> It's the main reason A/P has to be able to see Receiving's data.

## Step 10 — Something was wrong *(only if needed)*

**Who:** Receiver flags it; Buyer or Manager owns it
**Opens:** Purchase Orders / Accounts Payable
**Does:**

* Short shipment → reduce the payable, don't pay for what didn't come
* Warm or damaged product → file a vendor claim, request a credit
* Rejected at the door → it never enters inventory at all

**Changes:** Payable amount drops; inventory never counted the bad product.
**Feeds:** A/P, Reports

## Step 11 — Pay the vendor

**Who:** A/P clerk enters it, Manager or owner approves
**Opens:** Accounts Payable
**Does:** Pays per terms (Net 30 from the receipt date, not the order date).
**Changes:** PO status = **Paid**. The payable clears.
**Feeds:** Reports

## Step 12 — It shows up in the numbers

**Who:** Manager / owner
**Opens:** Reports
**Sees:**

* The real landed cost of that item — which is what makes gross margin possible on the sales side
* The aging bucket that product sits in, counting from the receipt date
* Inventory turns

**Feeds:** Back to **Step 1**. The buyer looks at this next week to decide what to
buy — and the loop closes.

--- 
 
## Questions this raises for GB

* Who actually does each of these jobs today? How many are the same person?
* Does anyone approve a PO before it goes to the vendor, or does the buyer just send it?
* What happens today when the vendor's invoice doesn't match what arrived? Who catches it, and how?
* Does a truck ever show up with **no PO** — a walk-in, a substitution, a favor from a vendor? How is that handled?
* When only part of an order arrives, do you keep the PO open for the rest or write a new one?
* Who is allowed to change a price after the PO is sent?
