# Module Plan

**Goal:** everything GB needs to run the business without paper.
**Rule:** every fact gets typed in ONCE, at the moment it physically happens.

---

## How it all connects

```
              ITEM MASTER  +  CUSTOMERS & VENDORS
              (everything below points back to these)
                              │
        ┌─────────────────────┴─────────────────────┐
        │                                           │
  SALES ORDERS                               PURCHASE ORDERS
        ↓                                           ↓
    SHIPMENTS  ──────►  INVENTORY  ◄──────      RECEIVING
        ↓            (the warehouse itself)          ↓
       A/R                                          A/P
   (money in)                                  (money out)
        │                                           │
        └──────────────►  REPORTS  ◄────────────────┘
                    (what management watches)
```

Left side = product leaves, money comes in.
Right side = product arrives, money goes out.
Everything meets in the middle at Inventory.

---

## The modules

### 1. Item Master

* GB needs to be able to add every product they sell, one time, with its unit (case / pound / pallet)
* GB needs to be able to search any product

**Why first:** every other module points back to this list. Nothing works until items exist.

### 2. Customers & Vendors

* GB needs to be able to add a customer, with payment terms and a credit limit
* GB needs to be able to add a vendor
* GB needs to be able to search either one

### 3. Purchase Orders

* GB needs to be able to enter purchase orders (will go to A/P)
* GB needs to be able to view and search the purchase history

### 4. Receiving (incoming)

* GB needs to be able to check an arriving truck against the PO
* GB needs to be able to record the QC check: count, weight, temperature, condition, dates
* GB needs to be able to assign or confirm a barcode on every case
* GB needs to be able to assign a warehouse location
* GB needs to be able to receive the PO into inventory (this is when stock goes up)

### 5. Inventory

* GB needs to be able to view and search any item in the warehouse, with its location and quantity
* GB needs to be able to see the full history of a case: which vendor it came from, where it was put, who moved it, which truck took it, which customer got it
* GB needs to be able to adjust a count, with a reason attached

### 6. Sales Orders

* GB needs to be able to enter customer orders (will go to A/R)
* GB needs to be able to view and search the order history
* GB needs to be warned when a customer is over their credit limit

### 7. Shipments (outgoing)

* GB needs to be able to print a pick ticket and release it to the warehouse
* GB needs to be able to record what was *actually* picked (not just what was ordered)
* GB needs to be able to stage orders by route and driver
* GB needs to be able to record which driver and truck took which orders
* GB needs to be able to capture the customer's signature as proof of delivery
* GB needs to be able to close out each driver at the end of the day

### 8. Parking / Dock Scheduling

* GB needs to be able to see which trucks are coming in and going out today
* GB needs to be able to assign a truck to a door or a spot

### 9. Invoices

* GB needs to be able to view and search every invoice
* GB needs to be able to see whether it was delivered and signed

### 10. Accounts Receivable (money in)

* GB needs to be able to see who owes them money and how late it is
* GB needs to be able to record a payment

### 11. Accounts Payable (money out)

* GB needs to be able to see who they owe and when it's due
* GB needs to be able to pay only for what actually arrived in good condition

### 12. Reports

* GB needs to be able to see how old the stock is: 0–30 / 31–60 / 61–90 / 90+ days
* GB needs to be able to see inventory turns (target: ~30 days)
* GB needs to be able to see gross margin by customer and by item
* GB needs to be able to see A/R aging (target: under 30 days)

**Note:** this module is mostly free. It's just reports on top of data the other modules already collect — but it's the part management will care about most.

---

## Things that go wrong (needs to be recorded everywhere)

Not really its own module — every module needs a way to log it:

* short picks (ordered 10, only 7 available)
* wrong item picked
* customer rejects the delivery at the door
* returns and credits
* damaged or spoiled product
* temperature problems
* weight discrepancies (box says 40 lb, scale says 38.6 lb)
* claims against a vendor

Every one of these needs a person's name attached and a note. This is where the money leaks.

---

## Build order

Not all at once. Roughly:

1. **First — the physical loop:** Item Master, Customers & Vendors, Receiving, Inventory, Shipments (plus thin Sales Order and Purchase Order entry, just enough to generate a pick ticket). This is the inventory management system, and it's what stops the re-typing.
2. **Second — Parking / Dock Scheduling.**
3. **Third — the paperwork:** full Sales Orders, full Purchase Orders, Invoices.
4. **Fourth — the money:** A/R, A/P, Reports.

---

## Questions to ask GB at the warehouse

The answers here change what we build, so get them before writing any spec.

**About the product**

* When you sell a case of chicken, do you charge by the case or by the pound?
* Roughly how many different products do you carry?
* Do you track lot numbers or pack dates, or just the item?

**About the warehouse**

* When a worker picks an order, do they take the oldest case first, or whatever's closest?
* How do you know today what's been sitting here more than 90 days?
* How many locations / zones / freezers are there?

**About buying**

* When you decide how much to order, what do you actually look at?
* How long does a typical vendor take to deliver after you send the PO?

**About customers**

* Do customers have credit limits today? Who approves them?
* What happens when a customer refuses a delivery at the door?

**About the paper**

* Walk me through one order from phone call to payment — show me every piece of paper it touches.
* How many times does the same number get typed or copied by hand?
* Who enters the paper into Excel, and how long does it take each day?

**About the trucks**

* How many trucks and drivers go out each day, and how many stops each?
* How are the routes determined? By hand or via an algorithm? 
* How many trucks arrive in a typical day?
