Authoritative project context. Update this file as facts are confirmed; do not
let assumptions harden into requirements.

**Status:** Pre-discovery. No code written. No scope committed.
**Last updated:** 2026-08-18
**Nuances:** Currently includes (possibly intertwined) information about the home page and the inventory page. 

---

## 1. Engagement

Paid engagement / partnership with a **food wholesale distributor** (comparable
company: SJ Distributors). 

Prioritize building an inventory management system. Second priority is a system to track shipment trucks (parking) to increase efficiency. 

Long-term vision includes additional modules — invoices, sales, A/P, A/R,
shipments. **That list is a description of an ERP, not a roadmap.** It is a
decade of work in an audited domain. Treat it as direction, not scope.

### Current client state

- Operations run on **printed paper records**.
- Paper is **hand-typed into Excel** after the fact.
- No existing ERP, WMS, or inventory software.

---

## 2. The general plan.

1. (now) Draft a rough prototype to show the representative of what we think they want/need. That way, we can show a blueprint, and ask the client if it suits their needs, then ask for details to add. 
2. (later) We will visit the warehouse to get a walkthrough of the site and current workflow (we must take notes during this step).
3. (later) Based on the visit, we can come up with a rough plan and determine & discuss what we can actually build for them (the system itself).
4. (later) Once the visit is over, we can take time to write spec, propose build plan, and write the quote.

---

## 3. What the client (likely) wants.

1. Capture data once, when it physically happens, instead of re-keying it.
2. Know what's on hand and where, without walking the warehouse.
3. Know what happened (and who last touched it) to a shipment/case/product if anything goes wrong (including theft).
4. Have an audit trail of every single shipment and individual case.

**The meeting's key move:** Understand the client's current process, any inefficiencies, and any extra premium information/data that the client would like to have access to. Then, determine what we can provide to the client. 

---

## 4. Current phase and background information/example of workflow. **MOST IMPORTANT PART**

Currently have not met with client. Prepare to meet with them at the warehouse. 
Current prototype is not up to my standards because there is too much going on. I'd like to redo the prototype. 
Focus on preparing to build them a inventory management system (get well versed in this sector). 

For example, regarding the inventory management system (within the inventory module) there are many tiers of products (individual case/box/unit, pallet, general item, etc.). A shipment from Tyson Foods could arrive in a reefer trailer (where the driver forgot to turn on the cooling system, causing the products to go rancid), containing 5 pallets of different chicken cuts. The cases in the pallet are distributed and placed in various areas of the warehouse, by different workers. The cases get loaded onto various local, smaller distribution trucks to the end user. The end user complains that the product was not up to quality standards, so we want to be able to pinpoint what went wrong. 

Idealy, the prototype (and the end result) will be easy to use, without much clutter on the screen. The home page will contain all the modules: large squares with rounded edges. Three squares per row. The modules will include Accounts Receivable, Accounts Payable, Customers/Vendors, Inventory, All Invoices, Item Master, Sales, Shipments, Parking (incoming and outgoing trucks), Purchase Orders (POs). 

---