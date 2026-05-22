# ServiceNow Hardware Asset Management (HAM) — Complete Interview Notes

> Source: ServiceNow HAM Fundamentals course (2022), Dell LCH project work, platform experience  
> Last updated: May 2026

---

## Table of Contents

1. [What is HAM?](#1-what-is-ham)
2. [Asset Lifecycle — The 8 Stages](#2-asset-lifecycle--the-8-stages)
3. [Asset Classes and Tables](#3-asset-classes-and-tables)
4. [Product Models and Normalization](#4-product-models-and-normalization)
5. [Asset Lifecycle Automation — Flows](#5-asset-lifecycle-automation--flows)
6. [Standard OOTB Flows](#6-standard-ootb-flows)
7. [Asset Tasks](#7-asset-tasks)
8. [Asset Actions](#8-asset-actions)
9. [Stockrooms, Stock Rules, and Transfer Orders](#9-stockrooms-stock-rules-and-transfer-orders)
10. [Contract Management](#10-contract-management)
11. [Reporting and Dashboards](#11-reporting-and-dashboards)
12. [Assets vs. CIs — Key Distinction](#12-assets-vs-cis--key-distinction)
13. [Discovery Integration (SCCM / Intune)](#13-discovery-integration-sccm--intune)
14. [Real-World Project: Dell LCH (Life Cycle Hub)](#14-real-world-project-dell-lch-life-cycle-hub)
15. [HAM Roles](#15-ham-roles)
16. [Common Interview Questions & Answers](#16-common-interview-questions--answers)

---

## 1. What is HAM?

**Hardware Asset Management (HAM)** is the ServiceNow module that manages the full lifecycle of physical hardware — from the moment it's planned for purchase through eventual disposal.

- Activated via the **Hardware Asset Management plugin**
- Sits on top of the base ITAM platform
- Provides automation, flows, tasks, dashboards, and contract tracking
- Single system of record: asset data, CI data, and financial data all linked together

**Why it matters:**
Without HAM, teams manually update asset records, CMDB CIs, software allocations, and maintenance contracts every time a hardware event happens. Every manual step is a risk for data inaccuracy. HAM automates all of those updates so agents can focus on resolution, not data entry.

---

## 2. Asset Lifecycle — The 8 Stages

```
Plan → Acquire → Receive → Inventory → Deploy → Maintain → Retire → End of Life
```

| Stage | What happens |
|-------|-------------|
| **Plan** | Define what hardware is needed; create purchase requests |
| **Acquire** | Purchase orders raised; vendor sourcing begins |
| **Receive** | Hardware physically arrives; ASN (Advance Ship Notice) or manual receiving |
| **Inventory** | Asset stored in stockroom; ready for deployment |
| **Deploy** | Asset assigned and deployed to a user or location |
| **Maintain** | Active use; incidents, repairs, warranty tracking |
| **Retire** | Asset removed from service; reclamation or swap initiated |
| **End of Life** | Disposal or return; disposal orders, RMA, vendor credit |

**Real-world example (Dell LCH):**
At Dell's Life Cycle Hub, when a hardware asset was picked for a customer order, the system tracked it through Receive → Inventory → Deploy automatically. The partner received status updates via the EPN Portal at each stage without any manual intervention from the Dell team.

---

## 3. Asset Classes and Tables

The **base asset class** is `alm_asset`. All asset types extend from it.

```
alm_asset (base)
├── alm_hardware      → Physical hardware assets (laptops, servers, etc.)
├── alm_consumable    → Consumables (printer toner, cables, etc.)
├── alm_license       → Software license entitlements
└── alm_facility      → Facility-related assets
```

**Key rule:** When creating a custom asset class, the table name must start with `u_alm_` to identify it as an asset table.

**Asset record key fields:**
- `ci` — linked CI record
- `model` — product model (make/model of device)
- `state` / `substate` — lifecycle position (In use, In stock, On order, Retired, etc.)
- `assigned_to` / `location` — who has it and where
- `cost` / `cost_center` — financial tracking
- `stockroom` — where it's physically stored when not deployed

---

## 4. Product Models and Normalization

**Product models** describe the make and model of an asset — not the individual unit.

Base model class: `cmdb_model`

```
cmdb_model (base)
├── cmdb_hardware_product_model    → Laptops, servers, desktops
├── cmdb_consumable_product_model  → Consumables
├── cmdb_software_product_model    → Software
├── cmdb_contract_product_model    → Contract types
├── cmdb_facility_product_model    → Facility assets
└── cmdb_application_product_model → Applications
```

**Asset Normalization:**
- HAM includes a **Hardware Model Content Service** — a database of known hardware models from manufacturers
- Normalization matches your asset records to canonical model data (manufacturer lifecycle dates, end-of-life info)
- Keeps your asset catalog clean and accurate without manual upkeep
- Unnormalized models show up on the Hardware Asset Dashboard as issues to remediate

**Real-world example:**
At Dell, when SCCM discovery data came in, the computer model field often had slightly different naming conventions from the manufacturer. The normalization engine mapped these to the correct product model records so lifecycle reporting was accurate across all devices.

---

## 5. Asset Lifecycle Automation — Flows

**What it is:** Pre-built Flow Designer flows that automate asset record updates throughout the lifecycle. When HAM plugin is installed, these flows are available out-of-the-box.

**Key concept:** These flows are **read-only** in their base form. You cannot modify the original — if you need to customize, **copy** the flow and configure the copy. This preserves the base automation while letting you tailor it.

**How to view flows:**
Navigate to `Process Automation > Flow Designer` → filter by **Hardware Asset Management** application

**Required role to run flows:** `flow_operator`  
**Required role to customize flows:** `flow_designer`

**What flows do:**
- Update asset state, substate, location, assigned_to automatically
- Update linked CIs in CMDB
- Manage software allocations (bring back to inventory when hardware is swapped/retired)
- Apply/end maintenance contracts
- Remove the need for agents to manually touch multiple records

---

## 6. Standard OOTB Flows

| Flow Name | What it does |
|-----------|-------------|
| **Standard Hardware Asset Request** | Request, source, and deploy hardware from Service Catalog. Takes asset from Plan → Deploy. |
| **Hardware Stock Order Flow** | Bulk order hardware for stockrooms. Takes asset through Plan → Inventory. |
| **Hardware Asset Refresh Flow** | Refresh aged assets nearing end-of-life. Replaces old assets with new ones. |
| **Hardware Asset Reclamation Flow** | Reclaim assigned assets when employee leaves or changes roles. Integrates with HR. |
| **Hardware Asset Disposal Flow** | Create disposal orders for end-of-life or non-functional assets. |
| **Loaner Asset Request Flow** | Request a loaner asset for a short period of time. |
| **Asset RMA Flow** | Return Merchandise Authorization — buyer return or replace faulty product. |
| **Lease Contract Expiration** | What to do before lease expires: buy out, extend, or return asset. |
| **Contract Renewal Flow** | Renew contracts nearing expiration. |

**How to trigger flows:**
- **Service Catalog → Asset Lifecycle category** (must be added to catalog home page)
- **Application Navigator** (e.g., Inventory > Stock > Submit Stock Order)
- **Automatically** via contract events, scheduled jobs, or other lifecycle triggers

**Real-world example (Hardware Asset Refresh):**
A client had 438 laptops nearing end-of-warranty (visible on the Hardware Asset Dashboard). We used the Hardware Asset Refresh Flow — submitted a multi-model refresh order, the flow scheduled deployment dates with users, handed off new assets, reclaimed old ones, and transferred software entitlements — all guided by prescriptive tasks. Finance saw the swap happen in the asset records in real time.

---

## 7. Asset Tasks

**What they are:** Background automation tasks that fire automatically when an Incident, Change, or Work Order is resolved.

**No configuration required** — when HAM plugin is active, asset tasks automatically attach to existing Incident, Change, and Work Order flows.

**What asset tasks do automatically:**
- Update the affected CI (e.g., mark as removed from service)
- Update the related asset record (state, substate, assigned_to, location)
- Handle software allocations (return to inventory when device is swapped/retired)
- Apply/end maintenance contract dates

**Pre-HAM (manual) vs. Post-HAM (automated):**

| Action | Pre-HAM | Post-HAM |
|--------|---------|----------|
| Update CI | Agent does it manually | Asset task does it automatically |
| Update asset record | Agent does it manually | Asset task does it automatically |
| Return software allocations | Agent does it manually | Asset task does it automatically |
| End maintenance contract | Agent does it manually | Asset task does it automatically |

**Real-world example:**
Before HAM, when an agent resolved a Change ticket to retire a server, they had to manually: update the CI, update the asset record, return software allocations, and add end dates to contracts. Miss any one of these and you're paying for contracts on decommissioned hardware or software licenses on machines that don't exist. HAM asset tasks handle all four automatically the moment the ticket is resolved.

---

## 8. Asset Actions

When an agent resolves an **Incident, Change, or Work Order**, they must select an **Asset Action** on the Affected CI tab. This tells HAM what happened so the right asset task fires.

**Required role:** `itil` or `admin`

| Asset Action | What it does |
|-------------|-------------|
| **No Action** | No updates to asset or CI (e.g., user error, no hardware change) |
| **Update/Repair** | No asset or CI record updates (e.g., software fix, restarted computer) |
| **Swap** | Updates both asset AND CI records. Swapped CI field must be filled with the new CI. |
| **Retire** | Updates both asset AND CI records. Asset removed from service, software reallocated, contracts ended. |

**Swap specifics:**
- Agent fills in the **Swapped CI** field with the new device
- Software allocations move from old CI to new CI
- Maintenance contracts: end date applied to old asset, start date created for new asset
- Location updates automatically based on assigned user

**Interview tip:** If asset actions aren't working, the first thing to check is whether the CI is opted into the correct **HAM Resource Categories**. If the CI class isn't mapped to a resource category, asset tasks won't fire.

---

## 9. Stockrooms, Stock Rules, and Transfer Orders

### Stockrooms
Physical or virtual locations where assets are stored before deployment (or after return).

- Each stockroom has a **manager**, **location**, and **type**
- Assets in stockroom have state: **In stock**
- Required for HAM lifecycle flows to function properly

### Transfer Orders
Move assets between stockrooms. Key for:
- Rebalancing inventory across locations
- Fulfilling requests from a stockroom that has stock when another doesn't
- Tracking asset movement with an audit trail

### Stock Rules
Automated rules that monitor stockroom inventory levels and trigger restocking when stock falls below a threshold.

**Two restocking options:**
1. **Create a task** for the stockroom manager to create a purchase order (email notification sent)
2. **Create a transfer order** to restock from another stockroom

**How it runs:** A daily scheduled job called **Stock Rule Runner** evaluates all defined stock rules. If a threshold is breached, it kicks off the appropriate restocking action.

**Bulk Stock Order — 3 ways to trigger:**
1. Automatic via stock rule when threshold is breached
2. Ad-hoc from **Service Catalog → Asset Lifecycle → Hardware Inventory Stock Order**
3. Manual via **Inventory → Stock → Submit Stock Order**

**Real-world example:**
At Dell LCH, stockrooms were created per customer/partner. When a partner's stockroom dropped below the minimum threshold for a given device model, the Stock Rule Runner would automatically create a transfer order to pull stock from the central warehouse — no manual intervention needed.

---

## 10. Contract Management

### Why track contracts in HAM?
- Know when warranties expire before they lapse (avoid paying for out-of-warranty repairs)
- Track lease end dates and decide: buy out, extend, or return
- Match maintenance agreements to the assets they cover
- Identify and eliminate contracts on decommissioned hardware

### Contract Types in ServiceNow
- Insurance
- Lease
- License Bundle
- Maintenance
- Non-Disclosure
- Purchasing Agreements
- Purchase Order
- Service
- Software License
- Subscription
- **Warranty** (most common for HAM)

### Contract Lifecycle States

```
Draft → Active → Expired/Cancelled
         ↑______________|
         (Adjust or Renew)
```

| State | Description |
|-------|-------------|
| **Draft** | Being set up; most fields editable |
| **Active** | Approved and within start/end dates; many fields read-only |
| **Expired** | End date reached |
| **Cancelled** | Discontinued; condition checkers become inactive |

### Manual Contract Actions

| Action | When to use | Key behavior |
|--------|-------------|--------------|
| **Renew** | Start a new contract when old one expires | New contract created; history retained; requires State = Active or Expired, Substate = None or Rejected |
| **Adjust (Extend)** | Extend existing contract's end date | Same contract record updated; all dates recalculate; asset end dates also update |
| **Cancel** | Discontinue contract | State → Cancelled; condition checkers inactive; rate cards inactive |

### Contract Notifications
ServiceNow automatically sends expiration reminders to contract administrators:
- **90 days** before expiration
- **60 days** before expiration
- **30 days** before expiration
- **On the expiration date**

Runs via an automated scheduled event that fires nightly.

### Contract Renewal Flow
Automated flow for renewing contracts. Consists of 10 steps:
1. Contract renewal request
2. Contract selection
3. Build renewal
4. Hardware assets
5. Software assets (SAM Pro required)
6. Terms and conditions
7. Rate cards
8. Renewal confirmation
9. Renewal approval
10. Renewal purchase order

**Supports:** Software license, Subscription, Maintenance, and Warranty contract models.

### Tracking Assets Covered by Contracts
- Add assets to the **Assets Covered** related list on the contract record
- For large volumes, use Import Sets to bulk-load the relationships
- Useful for lease contracts where you need to know exactly which devices are covered

### Terms and Conditions
- Create T&C records and associate them with contracts
- ServiceNow tracks changes to T&C over the life of the contract
- Changes tracked in **Contract History** (start/end dates, T&C changes)

**Real-world example (Dell LCH):**
Service life dates (EOL/EOSL) on assets were calculated using the latest of the warranty end date and lease end date, with customer-defined offsets applied. A scheduled job ran periodically to refresh these calculations across all assets — this fed the end-of-life reporting on the Hardware Asset Dashboard.

---

## 11. Reporting and Dashboards

### Hardware Asset Dashboard
Navigate to: `Asset > Hardware Asset Dashboard`  
Required role: `asset`

The command center for the hardware asset estate. Powered by **Performance Analytics**. Updates in real-time (except Lifecycle Overview sections, which run on a daily scheduled job).

**5 tabs on the Hardware Asset Dashboard:**

| Tab | What it shows |
|-----|--------------|
| **Asset Health** | Defective, missing, aging assets; assets not discovered in 30+ days |
| **Model Management** | Models reaching end-of-life (this month / quarter / year) based on manufacturer lifecycle data |
| **Procurement** | Open POs, orders by vendor, expenditures, orders needing sourcing |
| **Inventory** | Open stock orders, transfer orders, loaner orders, RMA lines |
| **End of Life** | Assets nearing warranty expiration; disposal status and methods |

### Asset Health Details
- **Incomplete Assets** — missing purchase info (PO number, order line, receiving line)
- **Eligible for Refresh** — assets at/near EOL, eligible for replacement
- **Asset Incident Frequency** — assets with high incident rates (prioritize replacement)
- **Active Assets Not Discovered** — deployed assets not seen by discovery in 30+ days

### Asset Management Executive Dashboard
Navigate to: `Asset Executive Workspace > Asset Management Executive Dashboard`  
Required role: `sn_itam_common.asset_exec`

Provides KPIs across HAM, SAM, and Cloud Insights in one view:
- Total spend
- Assessment fulfillment time
- Missing hardware assets
- Expiring contracts in 90 days
- Assets pulled from pool vs. net new purchase
- End of life models in next 90 days

Available from **Tokyo release** onwards.

### My Assets Homepage
`Self-Service > My Assets` — user-facing view showing:
- Requests and assigned assets
- Entitled software
- Upcoming scheduled retirements

### Report Types Available
Pie chart, Bar chart, List, Pivot table, Box chart, Calendar, Control chart, Histogram, Horizontal bar, Line chart, Pareto chart, Trend chart, Trend box chart, Vertical bar

**Important:** ACLs control access to underlying table data. Users without sufficient permissions will see filtered list reports.

---

## 12. Assets vs. CIs — Key Distinction

This is one of the most common interview topics.

**Assets** = financial/lifecycle tracking objects (`alm_asset` table family)  
**CIs** = operational/configuration tracking objects (`cmdb_ci` table family)

The same physical device is tracked as **both** — linked together via the `ci` field on the asset record.

### What are CIs AND Assets (both)?
- Servers
- Workstations / Desktops
- Enterprise network hardware / appliances
- SAN Enterprise and local storage devices

### Keep non-asset CIs OUT of asset management:
- Ports
- IP addresses
- Patches
- Virtual machines

### Best practices — Asset vs. CMDB

| Rule | Why |
|------|-----|
| Keep non-asset CIs out of asset management | IP addresses and ports don't have financial value or lifecycle |
| Keep asset and CMDB in sync | Operation status and location changes should match |
| Any supportable asset should be a CI | Incidents reference CIs, not assets |
| Asset data should support operations | Repair under warranty, leverage maintenance contracts |
| Make sure any supportable asset is a CI | Incidents reference CIs, not assets! |

**Real-world example:**
When we retired a server in a Change ticket at Dell, HAM's asset task automatically updated the CI state to "Removed from service" AND the asset substate to "Disposed" — keeping both records in sync without the agent touching either. Before HAM, this required separate manual updates and often got missed.

---

## 13. Discovery Integration (SCCM / Intune)

### How SCCM feeds HAM (Service Graph Connector)

**Flow:**
```
SCCM SQL DB → MID Server (JDBC) → Staging Tables → Transform Maps → CMDB (CI + Asset)
```

**CMDB tables populated from SCCM:**
- `cmdb_ci_computer` (Computer) — **required**
- `cmdb_ci_disk` (Disk)
- `cmdb_ci_ip_address` (IP Address)
- `cmdb_ci_network_adapter` (Network Adapter)
- `cmdb_serial_number` (Serial Number)
- `cmdb_software_instance` (Software Instance — if SAM not installed)
- `samp_sw_usage` (Software Usage)
- `cmdb_rel_ci` (CI Relationships)

**SCCM onboarding steps (per customer in a multi-tenant setup):**
1. **Grant DB access** — Windows mid server user profile with `db_datareader` role
2. **Configure SCCM Connector** — Flow Designer > Integrations > Outbound > SCCM JDBC Conn Cred
3. **Validate data source** — System Import Sets, set incremental fetch fields
4. **Schedule periodic import** — active, daily repeat interval
5. **Set Company field** via Business Rule on `cmdb_ci_computer` table (maps connection alias to company name)

**Key BR script for SCCM Company mapping:**
```javascript
(function executeRule(current, previous /*null when async*/) {
    var imp = new GlideRecord('sn_sccm_integrate_sccm_2019_computer_id');
    imp.addQuery('u_name', current.name);
    imp.orderByDesc('sys_created_on');
    imp.setLimit(1);
    imp.query();
    if (imp.next()) {
        var conn = new GlideRecord('sn_cmdb_int_util_service_graph_connection');
        conn.addQuery('connection_alias', imp.u_connectionid);
        conn.query();
        if (conn.next()) {
            current.company = conn.connection_alias.u_company;
        }
    }
})(current, previous);
```

**Incremental fetch configuration:**
- Use last run datetime: **True**
- Last run database field: **LastHWScan**
- This ensures only changed records are pulled on each scheduled run

---

## 14. Real-World Project: Dell LCH (Life Cycle Hub)

Dell's **Life Cycle Hub (LCH)** is a custom scoped ServiceNow application built on HAM to manage hardware asset lifecycle for Dell's managed service customers (partners). This is a B2B platform where Dell manages hardware on behalf of enterprise clients.

**Application scope:** `x_dusa2_lch_global`

### Key Concepts in LCH

#### Hard Booking
When a partner sends an FR (Fulfillment Request) update with a specific serial number for a picked asset:
- ServiceNow compares the serial to the asset on soft reserve
- **3 outcomes:**
  1. Serial doesn't exist → failure message back to partner
  2. Serial matches soft reserve asset → update both
  3. Serial doesn't match soft reserve → complex resolution logic
- Business Rule on `alm_asset` table handles the substatus change from "Reserved" → appropriate next state
- The RITM's "Reserved Asset" field is cleared as part of the same record update (to prevent recursion)

#### Non-Conforming (MAN-1441)
When a received asset cannot be matched in inventory:
- Asset moved to **Non-Conforming stockroom** with state: In stock non-conforming
- LCH Task created (type: Non-Conforming) against the RITM
- Task assigned to default assignment group from LCH System Property
- Partner can submit an update for re-evaluation
- Non-conforming substate is a **global choice** on `alm_asset` with scoped dependent values

#### Data Discrepancy (MAN-2834)
When the received asset exists in inventory but doesn't match what was expected:
- Create LCH Task (type: Data Discrepancy) against the RITM
- Update RITM expected asset to "In use"
- If returned asset isn't entitled → update to entitled
- Update RITM received asset state
- RITM status updated

#### Asset Quarantine
When an asset is returned to a partner, it may enter a **quarantine period**:
- New date field on asset record, calculated using customer default
- Scheduled job runs daily to check expired quarantines and close tasks
- When task closes, partner is notified via Update Asset
- SDM can submit catalog item to select assets for quarantine

#### Asset Recovery/Recycle
EOL assets go through disposal approval:
- Uses `sn_hamp_hardware_disposal` (disposal order) for customer approval before disposal
- Disposal order tracks line items; each line item = one asset being disposed
- SDM manages the approval/rejection via line items
- Line item approved → status: complete
- Line item rejected → status: cancelled
- Triggered by scheduled job (Periodic Contract Term Refresh)

#### Asset Registration Flow (LCH EPN Asset Registration)
Triggered via FR Update when an asset is picked:
1. A line item is updated with a picked asset via FR Update
2. Subflow `LCH EPN Asset Registration` fires from Script Include `DDLCHEPNIntegrations`
3. Subflow handles: data collection → de-registration → registration (comparing asset profile vs. requested profile)
4. At each stage, an FR Update is sent to EPN and an event fires for the partner to receive notification

**Registration flow states:** Data collection → Deregister → Register → Stock → Complete

**Post-registration status logic:**
- Pre-register request: → Stock
- Solution + persona consolidate flag = true: → Consolidate
- Solution + persona consolidate flag = false: → Configure
- All other: → Ship

#### Service Life / EOL Calculation
```
EOL = earliest of:
  1. Latest lease end date + Lease Offset (customer property)
  2. Latest warranty end date + Warranty Offset (customer property)
  → capture the earlier (retirement date)
  3. If EOSL is set (manual override) and device is owned:
     compare EOSL to calculated date → take the later
  4. Apply "End of service life date" offset (months)
  5. Final result = actual EOL
```

Key terms:
- **EOSL** (End of Service Life) — manually set optional override on `alm_hardware` table
- **EOL** (End of Life) — calculated final date
- **Model EOL** — when asset model is "Retired", EOL returns true regardless of dates

#### EPN Portal
Partner-facing portal that exposes ServiceNow data via APIs:
- **Receive Asset** — partner confirms receipt of hardware
- **Get Inventory** — partner views their stockroom
- **EPN Get Tasks (Aggregate)** — partner views open tasks
- **FR Update** — partner sends fulfillment updates to ServiceNow
- All API calls filtered by partner code; unrecognized codes return no results

#### LCH Entitlement Table
`x_dusa2_lch_global_lch_entitlement`
- Originally for SDR entitlements, modified to use `ast_contract` for OOB alignment
- Captures start/end dates with reference to asset record
- Enables reporting on how many assets are currently entitled

#### Staging Tables (Multi-tenant data import)
**Customer User Staging:** `x_dusa2_lch_global_customer_user_staging`
- Fields: Company (auto-populated by BR), First Name, Last Name, Email
- One staging table reused across all customers — no new table per customer
- Customers POST only their own data; read access restricted to own company's records
- Transform: first creates Contact record, then creates LCH User record referencing contact

**Inventory Tracking Staging:** `x_dusa2_lch_global_lch_staging_table`
- Minimum required fields: Company, Serial Number
- Optional: Model Manufacturer, Model Name, Asset Tag, State, Substate, Warranty dates, Lease dates, EOSL, Pick ID
- Transform maps source table data to HAM target tables

#### Kafka Integration
Dell needed to publish asset events to a Kafka streaming platform. Three connectivity options evaluated:

| Option | Decision | Reason |
|--------|----------|--------|
| **HIP (Hybrid Integration Platform)** | ✅ Selected | Fully self-service; supports both publish and consume; best for requirements |
| **API Gateway (L7)** | ❌ Rejected | Publish only; manual reconfiguration required by API engineering team |
| **Kafka Connect** | ❌ Rejected | Not enabled at cluster level in SDS |

**HIP flow:**
```
SNOW Upstream (JSON) → HIP [route, format, transform] → Kafka topic
Kafka topic → HIP [route, format, transform] → SNOW Downstream (JSON)
```

---

## 15. HAM Roles

| Role | Description |
|------|-------------|
| `asset` | Read/write access to asset records; view Hardware Asset Dashboard |
| `inventory_user` | Access Service Catalog Asset Lifecycle flows |
| `flow_operator` | Run HAM flows |
| `flow_designer` | Customize/copy HAM flows |
| `itil` or `admin` | Set Asset Action on Incident/Change/Work Order |
| `asset_manager` | Full asset management permissions |
| `contract_manager` | Manage contracts (renew, adjust, cancel) |
| `sn_itam_common.asset_exec` | View Asset Management Executive Dashboard |

---

## 16. Common Interview Questions & Answers

**Q: What is the difference between an asset and a CI?**  
A: An asset is a financial/lifecycle tracking record (who owns it, what it cost, when it's due for replacement). A CI is an operational/configuration record (what it's connected to, what services it supports). The same physical device is usually both — the asset's `ci` field links the two. Incidents reference CIs; purchase orders reference assets.

**Q: What triggers an asset task in ServiceNow?**  
A: Asset tasks fire automatically when an agent resolves an Incident, Change, or Work Order and selects an Asset Action on the Affected CIs related list. The asset task fires in the background and updates the CI, asset record, software allocations, and maintenance contracts based on the action selected (Swap or Retire trigger updates; No Action and Update/Repair do not).

**Q: What's the difference between Copy flow vs. modifying an OOB HAM flow?**  
A: OOB HAM flows are read-only — you cannot modify them directly. If your process needs additional steps or local procedures, you copy the standard flow and configure the copy using Flow Designer. This way you keep all the built-in HAM automation and just layer your customizations on top. The copy starts as read-write.

**Q: How does HAM integrate with HRSD for asset reclamation?**  
A: The Hardware Asset Reclamation Flow integrates with HR Service Delivery. When an employee separation event is triggered in HRSD, it kicks off the reclamation flow — guiding HR, managers, and the asset team through retrieving the assigned hardware. The asset can then be repaired, stored back in inventory, reassigned, or sent for repair. This eliminates the manual coordination that typically happens between HR and IT when someone leaves.

**Q: How do stock rules work?**  
A: Stock rules monitor a stockroom's inventory level for a specific model. When the quantity drops below the defined threshold, the Stock Rule Runner (a daily scheduled job) triggers either a purchase order task for the stockroom manager or a transfer order from another stockroom. This keeps stockrooms stocked automatically without someone manually watching inventory levels.

**Q: What happens to software allocations when an asset is retired?**  
A: When an agent resolves a ticket with Asset Action = Retire, HAM's asset task automatically returns all software allocations from that asset back to the corporate software inventory. This is critical for SAM compliance — without it, you'd keep paying for software licenses assigned to hardware that no longer exists.

**Q: What is CSDM and how does it relate to HAM?**  
A: The Common Service Data Model (CSDM) is ServiceNow's framework for how data should be structured and related across the platform. For HAM, this means ensuring that hardware assets are properly linked to CIs, which are properly linked to services and business applications. HAM implements CSDM by maintaining the asset-CI relationship and keeping both records in sync through the lifecycle.

**Q: What is asset normalization and why does it matter?**  
A: Asset normalization maps your asset model records to a canonical database of known hardware models (the Hardware Model Content Service). This gives you accurate manufacturer lifecycle data (end-of-life dates, end-of-service dates) automatically, rather than manually researching each model. It also cleans up inconsistent model names that come from discovery tools.

**Q: How does the Hardware Asset Refresh flow work?**  
A: You submit a Hardware Asset Refresh Order specifying: type of refresh (single or multi-model), replacement model(s), and criteria for selecting assets to replace (e.g., "Eligible for refresh" flag). The flow then handles scheduling deployment dates, coordinating with users, deploying new assets, reclaiming old ones, transferring software entitlements, and updating all asset/CI records automatically throughout the process.

**Q: In a multi-tenant HAM environment, how do you separate customer data?**  
A: Using a staging table approach with company-based access control. Each customer's data flows into a shared staging table; a Business Rule auto-populates the Company field based on the service account making the request. ACLs restrict read access so customers can only see their own records. Transform maps then move data from staging into the live HAM tables with proper relationships. This avoids creating separate staging tables per customer while maintaining complete data isolation.

---

*Notes compiled from: ServiceNow HAM Fundamentals Course (2022 edition) + Dell Life Cycle Hub project documentation*
