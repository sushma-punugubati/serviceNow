# ServiceNow Hardware Asset Management (HAM) — Complete Interview Preparation Notes

> Compiled from ServiceNow official product documentation, ServiceNow Community articles, and real-world implementation patterns. Suitable for ITAM Analyst, HAM Consultant, and ServiceNow Developer interviews.

---

## Table of Contents

1. What is Hardware Asset Management (HAM)?
2. HAM vs. CMDB — Asset vs. Configuration Item
3. Core Data Model & Key Tables
4. Hardware Asset Lifecycle (the heart of HAM)
5. Asset States & Substates
6. Model Categories, Models & Resource Categories
7. Stockrooms & Inventory Management
8. Transfer Orders & Stock Rules
9. Procurement, Purchase Orders & Receiving Slips
10. Consumables & Bundle Assets
11. Hardware Contracts (Lease, Warranty, Maintenance)
12. Hardware Refresh & End-of-Life Planning
13. Asset Audits & Reconciliation
14. Hardware Model Normalization
15. HAM Pro — Premium Capabilities
16. HAM + ITIL Process Integration (Incident, Change, Request)
17. Reports, Dashboards & KPIs
18. Common Integrations (Discovery, SCCM, Intune, JAMF)
19. Real-World Implementation Scenarios
20. Top 40+ Interview Questions & Answers

---

## 1. What is Hardware Asset Management (HAM)?

Hardware Asset Management is the ServiceNow application used to track every physical IT asset — laptops, desktops, monitors, servers, network gear, mobile devices, printers, peripherals — from the moment it is requested until it is finally retired and disposed of. HAM standardizes the **financial, contractual, and operational** view of hardware across the enterprise.

**Why organizations invest in HAM**
- Reduce hardware spend through better visibility and reuse.
- Ensure compliance with lease, warranty and disposal obligations.
- Eliminate "ghost assets" (paid for but missing) and "zombie assets" (still in CMDB but physically gone).
- Provide a clean asset feed to Incident, Problem, Change, and Request management.

**Real-world example.** A bank running 18,000 laptops discovers via HAM dashboards that 1,200 devices are past warranty, 400 are still active under leases that expired six months ago (incurring penalty rent), and 250 are sitting in a closet but never returned to a stockroom. Quantifying this in dollars is usually how the HAM business case is sold to leadership.

---

## 2. HAM vs. CMDB — Asset vs. Configuration Item

This is the single most common interview question. The clearest mental model: an **Asset** answers "Did we pay for it and do we still own it?" while a **Configuration Item (CI)** answers "Is it deployed and providing a service?"

| Aspect | Asset (`alm_hardware`) | Configuration Item (`cmdb_ci_computer`, etc.) |
|---|---|---|
| Primary purpose | Financial / contractual / lifecycle | Technical / operational / service |
| Lifecycle starts | When requested or ordered | When installed / discovered |
| Lifecycle ends | When disposed | When decommissioned (CI is retired but not deleted) |
| Key fields | Asset tag, cost, PO, warranty, lease, owner | IP address, OS, CPU, installed software, relationships |
| Relationship to other CIs | None (financial record) | Upstream / downstream dependencies |
| Source of truth for | Cost, ownership, contract | Service impact, change planning |

**One-to-one rule.** When you create a hardware asset, ServiceNow automatically creates a corresponding CI in the `cmdb_ci` hierarchy. They are linked through the **Configuration Item** reference field on the asset and you should not break this link.

**Real-world example.** A laptop CI shows `Status = Operational`. The corresponding asset shows `State = In Use, Substate = null, Assigned to = Sushma, Warranty end = 2027-08-12, Lease contract = LC0001234`. When Sushma leaves the company, the asset moves to **In Stock / Pending Repair** and the CI moves to **Operational → Non-Operational**. Both records survive the employee exit; neither is deleted.

---

## 3. Core Data Model & Key Tables

| Table | Purpose | Notes |
|---|---|---|
| `alm_asset` | Parent table for all assets (hardware, software, consumable) | Abstract parent — do not write business rules directly here |
| `alm_hardware` | All hardware assets | Child of `alm_asset`. Where 95% of HAM work happens |
| `alm_consumable` | Cables, toners, batteries — tracked by quantity | Quantity-based, not serial-based |
| `alm_bundle` | A group of assets shipped together (e.g., laptop + dock + monitor) | A child asset can belong to one bundle |
| `alm_stockroom` | Storage locations | Physical (warehouse) or virtual (in-transit, vendor RMA) |
| `alm_transfer_order` | Header for moving stock between stockrooms | One per request |
| `alm_transfer_order_line` | Each individual asset on a transfer | One row per asset |
| `alm_stock_rule` | Min/Max thresholds that auto-create transfer orders | Driven by hardware model |
| `cmdb_model` | Manufacturer + Name + Model Number (product catalog) | The "what" — not the "where" |
| `cmdb_model_category` | Bridges model class with CMDB CI class | E.g., "Computer" → `cmdb_ci_computer` |
| `cmdb_hardware_product_model` | Hardware-specific product model (manufacturer attributes) | Extends `cmdb_model` |
| `cmdb_ci` and children | Configuration items | Linked to assets 1:1 |
| `ast_contract` | Contracts (lease, warranty, maintenance, support) | Connected to assets via `m2m_contract_asset` |
| `proc_po` / `proc_po_item` | Purchase orders and line items | Drives "create asset on receipt" |
| `proc_rec_slip` | Receiving slip used at the loading dock | Trigger for asset creation in real-time |

**Real-world example.** During implementation a developer wants to add a custom field `u_last_audit_scan`. Adding it to `alm_hardware` is correct because the field only applies to hardware. Adding it to `alm_asset` would unnecessarily pollute software and consumable records.

---

## 4. Hardware Asset Lifecycle

ServiceNow defines four major lifecycle stages. Mastering this flow is essential for the interview.

### 4.1 Plan
Decide whether to **buy, lease, or re-use** an existing asset. Often driven by a demand or a project plan. HAM Pro adds an **Asset Reservation** concept so a stocked laptop can be earmarked for a future hire before it physically ships.

**Example:** A finance director requests 30 laptops for new analysts starting next quarter. The HAM team reviews refresh-eligible laptops (5+ years old) that can be rotated to a less demanding team, plans 10 reuses, and procures only 20 new units.

### 4.2 Acquire (Procurement)
Triggered by a fulfilled `sc_request_item` (RITM) or a demand. The procurement application creates a **Purchase Order**. When the asset physically arrives, a **Receiving Slip** generates `alm_hardware` records with an initial state of **In Stock / Available**.

**Example:** PO `PO0001234` for 20 Dell Latitude 7440s. When the box arrives, the receiving clerk scans the asset tags into a Receiving Slip. ServiceNow auto-creates 20 assets, links them to the PO line, sets `cost = $1,150`, and stamps `acquisition method = Purchased`.

### 4.3 Deploy / Operate
Asset is allocated to a user (or a CI), shipped, and turned on. State moves from **In Stock → In Use**. Discovery picks up the device and populates the linked CI.

**Example:** Asset is assigned to John in the Bangalore office; a Transfer Order moves it from the central warehouse to the Bangalore stockroom; then a Provisioning task moves it to John. The CI gets its `IP address`, `OS = Win 11`, `Last seen = today`.

### 4.4 Retire / Dispose
End of useful life. State moves to **In Stock → Pending Repair** (if salvageable) or directly to **Retired → Pending Disposal → Disposed**.

**Example:** A 6-year-old laptop fails. The repair shop says it's not economical to fix. State goes to **Retired / Pending Disposal**. A Transfer Order moves it to the disposal vendor stockroom (**In Transit / Pending Disposal**). The vendor sends a certificate of destruction; state becomes **Retired / Disposed** and the linked CI is set to **Retired**.

---

## 5. Asset States & Substates

States and substates together describe both **where** an asset is and **why**. Out-of-the-box values:

| State | Common Substates | Meaning |
|---|---|---|
| In Stock | Available, Reserved, Defective, Pending Repair, Pending Disposal, Pending Transfer | Sitting in a stockroom |
| In Use | (none) — sometimes "Use" substates added by customers | Deployed to a user or CI |
| On Order | (none) | Ordered but not yet received |
| In Transit | Pending Disposal, Pending Install | Being moved between locations |
| Missing | (none) | Cannot be found during audit |
| In Maintenance | (none) | Sent for repair |
| Retired | Pending Disposal, Sold, Donated, Disposed, Vendor Credit | End of life |

**Why substates matter.** A "Retired" asset that is still inside the building is a security risk (data on the disk). A "Retired / Disposed" asset means the disk has been wiped/destroyed. Reports must distinguish the two.

**Real-world example.** A pharmaceutical company auditor demands evidence that all retired laptops with patient data were physically destroyed. The HAM team filters `state = Retired AND substate = Disposed` and exports the list with the certificate of destruction attachments. Without substates, this audit would fail.

---

## 6. Model Categories, Models & Resource Categories

### 6.1 Hardware Model
A `cmdb_model` (or `cmdb_hardware_product_model`) record represents a **product** — for example "Dell Latitude 7440, 16 GB RAM". One model has many assets (1:N).

### 6.2 Model Category
The bridge between an asset model and the CMDB class it should create CIs in. Each model category points to:
- An asset class (e.g., `alm_hardware`)
- A CMDB CI class (e.g., `cmdb_ci_computer`)

**Example:** Model Category `Computer` maps to `cmdb_ci_computer`. So when you create an asset using a model with this category, the corresponding CI lands in `cmdb_ci_computer`, not in the generic `cmdb_ci_hardware`.

### 6.3 Resource Category (HAM Pro Licensing)
HAM Pro is licensed by **resource category**. Categories are buckets like "Computers", "Servers", "Network Devices", "Mobile Devices". You opt-in to the categories you want managed under HAM Pro, and cost scales with the number of tracked assets in those buckets.

**Real-world example.** A 10,000-employee company subscribes only to "Computers" and "Mobile Devices" under HAM Pro, while servers and network gear remain in HAM Standard. This keeps cost down because servers are managed by a separate infrastructure team that uses Discovery + CMDB rather than HAM workflows.

---

## 7. Stockrooms & Inventory Management

A **stockroom** (`alm_stockroom`) is any physical or virtual location where assets can sit. Common types:

- **Main warehouse** — central storage
- **Site stockroom** — per-office storage
- **Vendor stockroom** — virtual, represents assets at a third party (e.g., during repair or imaging)
- **Disposal stockroom** — assets pending or undergoing disposal

Each user can have a **Default stockroom** based on their location, allowing automatic sourcing when they request hardware.

**Real-world example.** Sushma in Hyderabad requests a laptop via the catalog. Sourcing rule finds the "Hyderabad-Site-01" stockroom as her default; one Latitude 7440 with state `In Stock / Available` is reserved. If none are available, the system can create a **transfer order** from the Mumbai warehouse, or trigger a new PO.

---

## 8. Transfer Orders & Stock Rules

### 8.1 Transfer Order (TO)
A `alm_transfer_order` moves one or more assets between stockrooms. Each line (`alm_transfer_order_line`) represents one asset.

**Lifecycle of a TO:** Draft → Requested → Ready for Shipping → Shipped → Received → Closed.

While in transit the asset's state changes to **In Transit**; once received, it becomes **In Stock / Available** in the destination stockroom.

### 8.2 Stock Rules
A `alm_stock_rule` defines `min_quantity` and `max_quantity` for a model in a stockroom. When stock drops below `min_quantity`, ServiceNow automatically generates a **transfer order** from a source stockroom (or a procurement request if no source has stock).

**Real-world example.** "Dell Latitude 7440" at the San Francisco stockroom has min = 10, max = 30. When the count drops to 9, the system auto-creates a TO from the central US warehouse (which has 200 in stock) to ship 20 units, restoring inventory to max. This eliminates manual "I'm running low" emails.

---

## 9. Procurement, Purchase Orders & Receiving Slips

The Procurement module provides two main ways to bring assets into the platform:

1. **From a Purchase Order** — receive directly against a PO line. Quantities/serials are entered as items arrive.
2. **From a Receiving Slip** — typically used at a loading dock; receive without first locating the PO, then reconcile to the PO afterwards.

When you receive, ServiceNow can:
- Create **pre-asset stubs** at PO creation time (so asset tags can be pre-printed), or
- Create **new asset records** at the moment of receipt.

**Best practice for the interview.** Pre-creating assets on the PO enables asset-tag pre-printing and faster receiving, but requires the procurement team to know exact serial counts up front. Most enterprises use receive-on-arrival with bulk scan to keep the dock fast.

**Real-world example.** A retailer orders 500 monitors. Their CFO wants to capitalize each monitor individually. They pre-create 500 assets when the PO is approved, print 500 asset-tag labels, and ship them to the dock. When the truck arrives, the receiving clerk applies the pre-printed labels and clicks "Receive" — all 500 transition from **On Order** to **In Stock / Available** in one action.

---

## 10. Consumables & Bundle Assets

### 10.1 Consumables
Cables, toners, memory sticks, batteries — items that are tracked by **quantity**, not by serial number. Stored in `alm_consumable`. They have stockroom + model + quantity-on-hand, and are issued in lots.

**Real-world example.** A help-desk technician issues two HDMI cables to a meeting room. The consumable record decrements by 2; when it hits the stock rule's minimum, a refill PO is auto-suggested.

### 10.2 Bundles
A **bundle** is a parent-child relationship between assets that travel together. The parent is in `alm_bundle`; children point to the parent via `Parent` field.

**Real-world example.** "New hire kit" = laptop + dock + 2 monitors + keyboard + mouse + headset. The kit is a bundle; the kit is assigned to one user; if any single piece is returned, the child asset is unbundled and put back in stock, while the rest of the bundle continues with the user.

---

## 11. Hardware Contracts

Contracts (`ast_contract`) are linked to assets via the M2M table `m2m_contract_asset`. Common contract types in HAM:

- **Lease** — recurring rent, ownership stays with lessor; tracks return date.
- **Warranty** — covers repairs/replacements during a defined term.
- **Maintenance / Support** — pay-for-fix or pay-for-availability beyond warranty.
- **Purchase** — one-time agreement; often references the PO.

Key contract fields: Start date, End date, Renewal date, Contract value, Vendor, Asset coverage list.

**Lease expiry workflow.** Notifications fire 90/60/30 days before lease end. HAM Pro lets you define **lease refresh tasks** so end-users receive replacement devices on a fixed schedule before the lease return deadline, preventing penalty fees.

**Real-world example.** A manufacturer leased 2,000 laptops on a 36-month deal that expires next month. Without HAM, returning 2,000 devices on time is chaotic. With HAM contracts + transfer orders, the team auto-builds collection tasks 60 days before expiry, ships replacements, collects old units to a "Lease Return" stockroom, and ships pallets to the lessor with serial-level manifests.

---

## 12. Hardware Refresh & End-of-Life Planning

The **Asset Refresh** capability identifies assets approaching end-of-life based on:
- Age vs. **Useful Life** on the model (e.g., laptops = 48 months; servers = 60 months).
- Warranty end date.
- Manufacturer End-of-Life (EOL) or End-of-Support (EOS) date.
- Performance metrics (HAM Pro pulls from Discovery / agent data).

Assets that meet refresh criteria appear on an **Eligible for Refresh** report.

**HAM Pro — Hardware Refresh Zero Touch.** Automates the entire workflow:
1. Identifies eligible devices.
2. Generates a procurement request for replacements.
3. Schedules user-facing tasks (mail-in-old / receive-new).
4. Tracks return → wipe → disposal.

**Real-world example.** 1,500 laptops cross their 48-month mark in Q3. Zero-Touch creates an RITM for 1,500 replacements, sends each affected employee a self-service portal task with a date picker, and prepares pre-paid shipping labels — the IT team only intervenes on exceptions.

---

## 13. Asset Audits & Reconciliation

The Audit module schedules **periodic physical inventory checks** to verify the asset records match physical reality. ServiceNow supports:

- **Stockroom audits** — verify every asset in a stockroom is physically present.
- **User asset audits** — verify devices assigned to employees.
- **Wall-to-wall audits** — comprehensive site audit, usually with barcode scanners.

Output: discrepancy report flags **Missing**, **Found** (in physical but not in system), **Wrong location**, **Wrong owner** items.

**Real-world example.** Annual audit at a London office scans 612 assets; 8 are missing and 3 are unexpected (BYO devices that crept into the room). HAM updates 8 to **Missing**, creates 3 new asset records for the found items, and opens an investigation security incident for the missing ones.

---

## 14. Hardware Model Normalization

Without normalization, your model catalog grows messy: "Dell Latitude 7440", "Dell LATITUDE-7440", "DELL Lat 7440 i7" all referring to the same device. The **Hardware Model Normalization** service:

- Maps inbound model strings to a canonical name using ServiceNow's **Content Library**.
- Uses the manufacturer's **UNSPSC** (United Nations Standard Products and Services Code) for uniqueness.
- Updates the display name and removes duplicate model records.

**Real-world example.** Discovery feeds 14 variant spellings of "HP EliteBook 840 G9" into `cmdb_model`. Normalization consolidates them into one canonical model, the reports show 7,200 EliteBooks instead of confusing per-spelling slices, and refresh planning becomes accurate.

---

## 15. HAM Pro — Premium Capabilities

HAM Standard (Core Asset Management) gives you basic asset records, stockrooms, transfer orders, and contracts. **HAM Pro** adds:

| Feature | What it does | Business benefit |
|---|---|---|
| Asset Workspace (Workspace UX) | Single pane for asset analysts | Faster handling time |
| Mobile receiving & barcode scan | Loading-dock-friendly mobile UI | Receiving < 60 seconds |
| Hardware Refresh Zero Touch | Automates aging-device replacement | Predictable refresh cycles |
| Asset reservation | Earmark stock for future demand | Smoother onboarding |
| Predictive AI insights | Forecast stock-outs, anomaly detection | Avoids surprises |
| Enhanced model normalization | Content-library-driven cleansing | Clean reports |
| End-of-Life forecasting | Pulls vendor EOL/EOS data automatically | Reduces unsupported gear risk |
| Disposal management | Vendor pickup, certificate tracking | Audit compliance |
| Stockroom optimization | AI-suggested min/max thresholds | Reduced carrying cost |

**Interview tip:** When asked "Standard vs. Pro?", a strong answer is: *"Standard handles records, locations, and basic flow. Pro adds automation, prediction, and a purpose-built analyst workspace — most useful for organizations above ~5,000 devices or with heavy lease/refresh cycles."*

---

## 16. HAM + ITIL Process Integration

### Incident
When a user reports a broken laptop, the Incident form's **Configuration Item** field maps back to the asset. The technician can see warranty status before deciding repair vs. replace.

### Problem
Recurring failures of a specific model — for example all 50 Lenovo Yoga 7s losing Wi-Fi — generate a problem record. HAM data drives the impact analysis ("how many of this model do we own?").

### Change
Hardware refresh, BIOS upgrades, firmware patching, and decommissioning all flow through Change. The change record references the affected CIs; HAM provides the asset list to plan windows by location.

### Request Fulfillment
The catalog item "Order a new laptop" generates an RITM → procurement task or a stockroom allocation. The fulfillment ends by **assigning the asset** to the requester and updating state to **In Use**.

**Real-world example.** A change to replace 200 desktops with laptops in Singapore needs a CAB approval. The CAB views a HAM-driven dashboard: 200 desktops, average age 5 years, all out of warranty, replacement laptops already in stock at the local stockroom — approved.

---

## 17. Reports, Dashboards & KPIs

Common HAM KPIs:

- **Asset utilization** — % assets in `In Use` vs. `In Stock` (avoid idle inventory).
- **Stock turnover** — units issued per quarter vs. average stock on hand.
- **Out-of-warranty %** — share of operational assets without active warranty.
- **Refresh compliance** — % assets older than useful life still in production.
- **Lease return rate** — % devices returned on time vs. penalty.
- **Audit discrepancy rate** — missing / found / misplaced per audit.
- **MTTR for asset-related incidents** — speed of HAM data driving repair decisions.

**Real-world example.** A monthly HAM dashboard for the CIO shows: 12% out-of-warranty (red ▲ from 8%), 22% In Stock (yellow), $1.2M of leases expiring within 90 days. The CIO approves a one-time PO worth $600k to refresh out-of-warranty units, and asks ITSM to investigate the In-Stock spike (often a hint of over-ordering).

---

## 18. Common Integrations

| Source | Brings into HAM | How |
|---|---|---|
| **ServiceNow Discovery** | Up-to-date CI attributes; auto-creates CIs | MID Server probes |
| **SCCM / MECM** | Windows device inventory, software | ServiceGraph connector |
| **Microsoft Intune / JAMF** | Mobile + Mac inventory | ITSM/ITOM Connectors |
| **Lansweeper / ManageEngine** | Mid-market device inventory | IntegrationHub |
| **HR system (Workday/SAP)** | Joiners/Movers/Leavers feed | Triggers asset reassignment |
| **ERP (SAP/Oracle)** | Financials, depreciation | REST/iDoc integration |

**Real-world example.** Workday triggers a Leaver event when an employee resigns. A flow finds all assets assigned to that user and creates a "Return assets" task on the manager; once returned, assets go to **In Stock / Pending Repair**. The HR-to-HAM link prevents devices from disappearing with departing employees.

---

## 19. Real-World Implementation Scenarios

### Scenario A — Onboarding a new hire
1. HR creates the new-hire record; an integration creates a sc_request for a "New Hire Kit" catalog item.
2. HAM bundles a laptop + monitor + dock from the local stockroom (or triggers a transfer order if local stock is low).
3. Asset records get **Assigned to = New Hire** and state moves to **In Stock / Reserved** → **In Transit** → **In Use** as the shipment progresses.
4. Discovery confirms the CI is operational; the request is closed.

### Scenario B — Broken laptop replacement
1. End user opens Incident `INC0101`; CI is auto-populated from the device telemetry.
2. Technician sees warranty `Active`; opens a vendor RMA.
3. The faulty asset moves to **In Maintenance**; a loaner from the local stockroom is allocated to the user.
4. Vendor replaces the device; new serial is updated, contract is re-linked; loaner returns to stock.

### Scenario C — Office closure
1. 380 assets in a closed office need redistribution.
2. HAM identifies all assets with `Location = Closed Office`.
3. Transfer Orders ship usable devices to other stockrooms; out-of-warranty units go to the disposal vendor stockroom.
4. Once disposal certificates arrive, all 80 disposed assets are marked **Retired / Disposed**; remaining 300 are now **In Stock / Available** in their destinations.

### Scenario D — Quarterly lease return
1. 600 laptops on Lease `LC-2026-04` expire next month.
2. HAM creates 600 individual return tasks using a Flow Designer flow.
3. Employees ship laptops to the disposal stockroom using prepaid labels.
4. As each device is scanned in, state moves to **In Stock / Pending Disposal** and a transfer order to the lessor is added to the next pallet.
5. Once acknowledged by the lessor, assets are **Retired / Vendor Credit**.

---

## 20. Top 40+ Interview Questions & Answers

**1. What is the difference between an Asset and a CI?**
An asset is the financial / lifecycle record (`alm_hardware`). A CI is the operational record (`cmdb_ci_computer`). They are linked one-to-one; one tracks ownership and cost, the other tracks technical configuration.

**2. Walk me through the HAM lifecycle.**
Plan → Acquire → Deploy/Operate → Retire/Dispose. Each stage maps to specific states (On Order, In Stock, In Use, In Transit, Retired) and substates.

**3. Where are hardware assets stored?**
In `alm_hardware`, which is a child of `alm_asset`.

**4. What is a Model Category and why does it matter?**
A bridge between a hardware model and the CMDB CI class. It tells ServiceNow which CI class to create when an asset of that model is generated.

**5. What is the difference between HAM Standard and HAM Pro?**
Standard provides core records, stockrooms, transfer orders. Pro adds Workspace UX, mobile receiving, Zero-Touch refresh, AI insights, advanced normalization, and resource-category licensing.

**6. What is a Stockroom?**
A physical or virtual location where hardware sits before deployment or after retirement. Stored in `alm_stockroom`.

**7. Explain Transfer Orders.**
A request to move one or more assets between stockrooms. Header is `alm_transfer_order`; each asset is a line in `alm_transfer_order_line`. Lifecycle: Draft → Requested → Ready for shipping → Shipped → Received → Closed.

**8. What is a Stock Rule?**
A min/max threshold per model per stockroom. When stock drops below `min`, a transfer order or PO is automatically created.

**9. How do receiving slips differ from receiving on a PO?**
Receiving slips are dock-friendly: scan items first, reconcile to a PO later. Receiving on a PO requires the PO to be open and is line-driven.

**10. What are consumables in HAM?**
Quantity-tracked items (cables, toners) stored in `alm_consumable`. They lack serial numbers.

**11. What is a bundle asset?**
A parent asset (in `alm_bundle`) that groups multiple child assets shipped together — e.g., a new-hire kit.

**12. How are contracts linked to assets?**
Via the many-to-many table `m2m_contract_asset`, with the contract header in `ast_contract`.

**13. List common asset substates and explain `Pending Disposal`.**
Substates include Available, Reserved, Pending Repair, Pending Disposal, Pending Transfer. `Pending Disposal` indicates an asset has reached EOL but is awaiting physical disposal; pairs with states In Stock, In Transit, or Retired.

**14. How does HAM integrate with Discovery?**
Discovery populates CMDB CIs; HAM ties those CIs to assets via the Configuration Item field on `alm_hardware`. Discovery never directly writes to `alm_hardware`.

**15. What is Hardware Model Normalization?**
A capability that standardizes model names, removes duplicates, and uses the manufacturer's UNSPSC code. Without it, reports are split across spelling variants.

**16. How is asset refresh automated?**
Models carry a `Useful life`; HAM flags assets older than that as eligible for refresh. HAM Pro's Zero-Touch Refresh fully automates ordering, shipping, returns, and retirement.

**17. What is the difference between Retired and Disposed?**
Retired means EOL but possibly still on-site; Disposed means physically destroyed/sold with a certificate. The substate distinguishes them.

**18. How does HAM relate to the CMDB?**
HAM is the financial/lifecycle layer; the CMDB is the operational/service layer. They share a 1:1 link via the Configuration Item field on the hardware asset.

**19. How are assets assigned to users?**
Through the `Assigned to` field on `alm_hardware`. Assignments are typically driven by request fulfillment, not manual edits.

**20. How do you handle a missing asset?**
Update state to `Missing`. Trigger an audit follow-up and (optionally) open a security incident. After a defined period, the asset is moved to Retired with a substate like `Lost`.

**21. What is the audit module used for?**
Periodic physical inventory verification at stockrooms, users, or sites. Generates discrepancy reports.

**22. Give an example of a use-case for a custom field on `alm_hardware`.**
A `u_disk_wipe_certificate` attachment reference to support compliance evidence on disposal.

**23. How do you stop assets from being deleted?**
Use ACLs to deny delete operations except for users with a HAM Admin role; rely on retire/dispose workflow instead of delete.

**24. What is the role hierarchy in HAM?**
Common roles: `asset`, `asset_admin`, `procurement_user`, `inventory_user`, `inventory_admin`, `ham_admin` (HAM Pro). Each scopes specific tables and forms.

**25. How does HAM Pro support remote workers?**
Through reservations, drop-ship workflows from vendor or central warehouse direct to home address, and mobile-app receiving.

**26. What is a `pre-asset`?**
A stub asset created at PO time so that asset tags can be pre-printed; gets serial / receiving info added later.

**27. Difference between `cmdb_model` and `cmdb_hardware_product_model`?**
`cmdb_model` is the generic product table; `cmdb_hardware_product_model` is a hardware-specific extension that holds attributes such as form factor, BIOS family, EOL date.

**28. What happens to the CI when an asset is retired?**
The CI is updated to a non-operational state (`Install status` becomes Retired / Decommissioned). It is **not** deleted, preserving historic relationships.

**29. How are warranties tracked?**
As contracts in `ast_contract` linked to one or more assets via `m2m_contract_asset`. The asset form shows the active warranty and its end date.

**30. What is the catalog-item / asset connection?**
A `sc_req_item` (RITM) drives procurement or stockroom allocation. The fulfillment task ultimately writes the asset's `Assigned to` and updates its state to In Use.

**31. How do you bulk-update asset locations?**
Through Import Sets and Transform Maps that match on Serial / Asset Tag, plus business rules to validate state transitions.

**32. What governance is needed before launching HAM?**
A naming convention for asset tags, an approved model catalog, defined state/substate workflow, a list of stockrooms, and clear ownership (who can change `Assigned to`, who can dispose).

**33. What metrics prove HAM is working?**
Reduction in audit discrepancies, lower out-of-warranty %, fewer stock-outs, faster onboarding time, and quantified savings from reuse.

**34. How is depreciation handled?**
HAM stores acquisition cost and depreciation method/start date; an out-of-the-box scheduled job updates `Depreciated amount` and `Residual value`. Larger orgs often integrate to an ERP for the authoritative book of record.

**35. What is the role of Resource Categories in HAM Pro?**
They define which buckets of hardware are licensed under HAM Pro. Cost scales by category and asset count.

**36. How would you migrate from a spreadsheet-based asset list to ServiceNow HAM?**
1) Cleanse data (unique serials, valid models, valid locations).
2) Define mapping (sheet columns to `alm_hardware` fields).
3) Use Import Sets + Transform Maps with coalesce on Serial Number.
4) Validate state/substate transitions; run a pilot stockroom first.
5) Reconcile against Discovery; close the gaps before go-live.

**37. How do you prevent users from creating CIs without an asset?**
Use Business Rules / ACLs that block direct insert on `cmdb_ci_computer` (and similar) by non-Discovery sources; force CI creation through `alm_hardware`. (Discovery is whitelisted.)

**38. What's a typical HAM project timeline?**
6–10 weeks for HAM Standard with 1–3 stockrooms and ~10k assets; HAM Pro adds 4–6 weeks for Workspace, Zero-Touch Refresh, and mobile dock receiving.

**39. Real-world challenge you've faced?**
*Sample answer:* "On a 25,000-device rollout, the existing CMDB had duplicate CIs from SCCM and JAMF. We ran Hardware Model Normalization, then enforced a single Identification & Reconciliation Engine (IRE) rule on serial number. The CI count dropped by 18%, and HAM assets aligned 1:1 within two reconcile cycles."

**40. What's the relationship between a Transfer Order and a Stockroom?**
A TO always has a **source stockroom** and a **destination stockroom**. The lines move individual assets between them.

**41. How is HAM data exposed to dashboards?**
Through standard ServiceNow Performance Analytics and Reports. HAM Pro adds out-of-the-box indicators (stock health, refresh forecast, audit accuracy).

**42. What's the role of Flow Designer in HAM?**
Automates lifecycle steps: receiving notifications, lease-expiry tasks, refresh enrollments, disposal certificate intake. Replaces older `wf_workflow` workflows.

**43. How is data protected on disposed devices?**
Disposal vendor sends a Certificate of Destruction; HAM attaches it to the asset and only then sets substate to `Disposed`. Without the certificate, status remains `Pending Disposal`.

**44. What's your favorite HAM enhancement?**
*Sample answer:* "Adding a barcode-scan-driven receiving flow on the mobile app cut our dock time from 8 minutes per box to under 60 seconds and eliminated manual tag-entry errors."

---

## Quick-Recall Cheat Sheet

- **Tables to remember:** `alm_hardware`, `alm_asset`, `alm_consumable`, `alm_bundle`, `alm_stockroom`, `alm_transfer_order`, `alm_transfer_order_line`, `alm_stock_rule`, `ast_contract`, `m2m_contract_asset`, `cmdb_model`, `cmdb_model_category`, `proc_po`, `proc_rec_slip`.
- **Lifecycle:** Plan → Acquire → Deploy → Retire.
- **States:** On Order, In Stock, In Use, In Transit, In Maintenance, Missing, Retired.
- **Top substates:** Available, Reserved, Pending Repair, Pending Disposal, Disposed.
- **Standard vs Pro:** Pro adds Workspace, Zero-Touch Refresh, mobile receiving, model normalization, AI insights, and resource-category licensing.
- **Asset vs CI:** Asset = financial; CI = operational; one-to-one relationship.

---

## Sources & Further Reading

- ServiceNow Docs — [Hardware Asset Management Landing Page](https://www.servicenow.com/docs/r/it-asset-management/hardware-asset-management/ham-landing-page.html)
- ServiceNow Docs — [Maturity stages of your HAM program](https://www.servicenow.com/docs/bundle/zurich-it-asset-management/page/product/hardware-asset-management/reference/maturity-stages-ham.html)
- ServiceNow Docs — [Transfer Orders for Asset Management](https://www.servicenow.com/docs/bundle/xanadu-it-service-management/page/product/asset-management/concept/transfer-orders-asset.html)
- ServiceNow Docs — [Manage lifecycle of hardware models with calculated dates](https://www.servicenow.com/docs/bundle/zurich-it-asset-management/page/product/hardware-asset-management/concept/manage-ham-lifecycle-temp.html)
- ServiceNow Docs — [CSDM Hardware Lifecycle](https://www.servicenow.com/docs/bundle/washingtondc-servicenow-platform/page/product/csdm-implementation/concept/csdm-lifecycle-hardware.html)
- ServiceNow Docs — [Set asset states and substates](https://docs.servicenow.com/en-US/bundle/vancouver-it-service-management/page/product/asset-management/task/t_SettingAssetStatesAndSubstates.html)
- ServiceNow Docs — [Manage expiring contracts for leased hardware assets](https://www.servicenow.com/docs/bundle/vancouver-it-asset-management/page/product/hardware-asset-management/task/manage-your-leased-hw-asts-expiring-contract.html)
- ServiceNow Community — [Mastering HAM in ServiceNow — Chapter 1](https://www.servicenow.com/community/ham-blog/mastering-hardware-asset-management-in-servicenow-chapter-1/ba-p/3256151)
- ServiceNow Community — [Mastering HAM in ServiceNow — Chapter 2](https://www.servicenow.com/community/ham-articles/mastering-hardware-asset-management-in-servicenow-chapter-2/ta-p/3351555)
- ServiceNow Community — [Essential Plugins & Terminology for HAM](https://www.servicenow.com/community/ham-forum/essential-plugins-and-terminology-for-streamlined-hardware-asset/m-p/3239979)
- ServiceNow Community — [HAM FAQ](https://www.servicenow.com/community/ham-articles/hardware-asset-management-faq/ta-p/3038612)
- ServiceNow Community — [Path to Value for HAM](https://www.servicenow.com/community/ham-articles/the-path-to-value-for-hardware-asset-management/ta-p/3044907)
- ServiceNow Community — [Assets and CIs: Understanding the Difference](https://www.servicenow.com/community/sam-articles/assets-and-cis-understanding-the-difference/ta-p/2405242)
- ServiceNow Community — [Core Asset Management vs HAM Pro](https://www.servicenow.com/community/ham-blog/core-asset-management-vs-ham-pro-what-you-actually-get-and-when/ba-p/3518013)
- ServiceNow Data Sheet — [The value of Hardware Asset Management (PDF)](https://www.servicenow.com/content/dam/servicenow-assets/public/en-us/doc-type/resource-center/data-sheet/ds-hardware-asset-management.pdf)
