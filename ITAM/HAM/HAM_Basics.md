# ServiceNow HAM — Fundamentals & Study Guide

> Sources: ServiceNow Docs | ServiceNow Community | Now Learning | HAM Data Sheet | ServiceNow Knowledge Conference

---

## 1. What is Hardware Asset Management (HAM)?

**Hardware Asset Management (HAM)** is the ServiceNow module for tracking, managing, and optimizing physical IT assets — laptops, desktops, servers, mobile devices, printers, and peripherals — across their entire lifecycle from procurement through disposal.

> **Core value proposition:** HAM replaces spreadsheets and siloed tracking systems with a single, automated source of truth for every physical asset the organization owns — who has it, where it is, what it costs, whether it's still under warranty, and when it needs to be replaced.

### What HAM Manages
- Physical hardware: laptops, desktops, servers, monitors, printers, tablets, phones
- Consumables: toner cartridges, cables, batteries (tracked by quantity, not individually)
- Facilities assets: racks, UPS units, data center equipment
- Peripheral devices linked to primary assets

### HAM vs. HAM Pro
| Feature | HAM (Standard) | HAM Pro |
|---------|---------------|---------|
| Asset tracking & CMDB integration | ✓ | ✓ |
| Lifecycle management | ✓ | ✓ |
| Stockroom management | ✓ | ✓ |
| Standard reporting | ✓ | ✓ |
| Hardware Asset Workspace | Basic | Full workspace |
| Content Service (normalization) | — | ✓ |
| Hardware refresh automation | — | ✓ |
| Asset reclamation flows | — | ✓ |
| Sustainability / ESG dashboards | — | ✓ |
| Now Assist (Generative AI) | — | ✓ |
| Mobile-first auditing (barcode/RFID) | — | ✓ |
| Zero-touch provisioning | — | ✓ |
| Predictive cost analytics | — | ✓ |

**When to choose HAM Pro:** Organizations with 1,000+ assets, strict compliance requirements (HIPAA, GDPR, SOC2), or distributed global workforce managing frequent device moves.

---

## 2. Asset Lifecycle — 8 Stages

```
Plan → Acquire → Receive → In Stock → Deploy → Maintain → Retire → End of Life
```

| Stage | ServiceNow Asset State | What Happens |
|-------|----------------------|-------------|
| **Plan** | — | Identify need, budget, vendor selection, purchase request |
| **Acquire** | On Order | Purchase order created, vendor engaged |
| **Receive** | In Stock (Receiving) | Physical receipt, serial number logged, asset record created or updated |
| **In Stock** | In Stock | Asset in stockroom awaiting deployment |
| **Deploy** | In Use | Assigned to user/location, CI linked in CMDB, asset tasks fire |
| **Maintain** | In Use | Repairs, upgrades, warranty claims, swap if broken |
| **Retire** | Retired | Data wipe initiated, removed from service, financial write-off |
| **End of Life** | Disposed | Physical disposal, donation, or scrapping; disposal record created |

### Asset State Transitions (OOTB)
- **In Stock → In Use**: Triggered when deployment task is completed
- **In Use → In Maintenance**: Triggered when a Repair asset task fires
- **In Use → Retired**: Triggered by Retire asset action or retirement lifecycle flow
- **Retired → Disposed**: Triggered when disposal order is completed

---

## 3. Asset Classes & Tables

| Table | Label | Purpose |
|-------|-------|---------|
| `alm_asset` | Asset | **Base table** — all asset types extend this |
| `alm_hardware` | Hardware Asset | Physical IT hardware (laptops, servers, desktops) |
| `alm_consumable` | Consumable | Tracked by quantity — toner, batteries, cables |
| `alm_license` | Software License | Software entitlements (SAM) |
| `alm_facility` | Facility Asset | Non-IT physical assets — furniture, racks, building equipment |
| `ast_contract` | Contract | Vendor contracts, lease agreements, maintenance contracts |
| `alm_stockroom` | Stockroom | Physical storage locations for undeployed assets |
| `alm_asset_task` | Asset Task | Workflow tasks auto-generated at lifecycle events |
| `alm_stock_rule` | Stock Rule | Rules defining minimum stock levels per stockroom |
| `alm_transfer_order` | Transfer Order | Tracks asset movement between stockrooms/locations |
| `alm_disposal_order` | Disposal Order | End-of-life disposal tracking |

---

## 4. Product Models & Normalization

### Product Model
Every hardware asset is linked to a **Product Model** — the "template" that describes the hardware type:
- Manufacturer (Dell, Apple, HP, Lenovo)
- Model (Latitude 5540, MacBook Pro 14", ThinkPad X1 Carbon)
- Asset class (Hardware, Consumable)
- Depreciation schedule
- Useful life (years)

**Product Model table:** `cmdb_model`

### Product Catalog vs. Product Model
- **Product Model**: Describes what the asset IS (specifications, class)
- **Product Catalog**: A catalog item employees can request to ORDER that model

### Content Service (HAM Pro) — Automated Normalization
The Content Service provides a library of pre-normalized product models from a ServiceNow-maintained database:
- Automatically matches discovered hardware to the correct manufacturer/model record
- Prevents "Dell Latitude 5540" vs. "DELL latitude5540" creating two different model records
- Keeps model data current — vendor updates (EOL dates, specifications) pushed automatically
- Eliminates manual model creation for thousands of device types

Without Content Service (standard HAM): teams manually create and maintain Product Model records. In a large enterprise with 500+ device models, this becomes a significant maintenance burden.

---

## 5. Stockroom Management

### What Is a Stockroom?
A Stockroom (`alm_stockroom`) is any physical location where assets are stored that aren't currently assigned to a user — receiving docks, IT storage rooms, warehouses, spare parts cabinets.

### Stockroom Types
| Type | Description |
|------|-------------|
| **Regular** | Standard storage — assets in stock awaiting deployment |
| **Defective** | Assets returned for repair or assessment |
| **Retired** | Assets pending disposal |
| **Virtual** | Conceptual grouping — e.g., "Assets in transit" |

### Stock Rules
Stock Rules define the minimum stock level for a specific model in a specific stockroom:
- "Stockroom Austin must always have at least 5 Dell Latitude 5540 laptops"
- The **Stock Rule Runner** scheduled job checks stock levels nightly
- When stock falls below minimum → automatically creates a **Bulk Stock Order**

### Transfer Orders
Moving an asset between stockrooms creates a Transfer Order (`alm_transfer_order`):
- Tracks source stockroom, destination stockroom, asset, and who authorized the move
- Provides full audit trail of every asset movement
- Used for: inter-office transfers, loaner returns, post-repair redistribution

---

## 6. Asset Tasks

Asset Tasks (`alm_asset_task`) are workflow-driven actions that fire automatically when Incidents, Changes, or Work Orders are resolved — they represent physical work that must be done to an asset.

### OOTB Asset Task Types
| Task Type | When It Fires | What IT Does |
|-----------|--------------|-------------|
| **Receiving Task** | Asset received (procurement flow) | Verify serial, log in stockroom, apply asset tag |
| **Deployment Task** | Asset assigned to user | Confirm delivery, update location, link CI |
| **Repair Task** | Incident/Change resolved with "Repair" action | Route to IT support, track repair, update asset |
| **Retirement Task** | Asset marked for retirement | Data wipe, remove from service |
| **Disposal Task** | Asset moved to Disposed state | Track physical disposal, update records |

### How Asset Tasks Are Generated
1. An Incident/Change/Work Order is resolved
2. The resolution form prompts: "What happened to the asset?"
3. The agent selects an **Asset Action**
4. Based on the action, an Asset Task is created automatically

---

## 7. Asset Actions

Asset Actions are the choices presented to an IT agent when closing an Incident or Change that involves a hardware asset:

| Action | What It Does | Asset Record Updated? | CI Record Updated? |
|--------|-------------|----------------------|-------------------|
| **No Action** | Records the disposition but no system changes | No | No |
| **Update/Repair** | Documents repair but no state changes | No | No |
| **Swap** | Replaces asset with a different one | Yes — both old and new | Yes — both old and new |
| **Retire** | Retires the asset from service | Yes | Yes — state → Retired |

> **Swap** is the most complex — it updates the old asset (returned to stockroom), the new asset (assigned to user), the CI (now linked to new asset), and returns any software allocations on the old asset.

---

## 8. Asset Lifecycle Automation Flows (OOTB)

These are pre-built, **read-only** flows in HAM Pro — they cannot be modified but can be copied and customized. They represent the core automated processes for hardware lifecycle management.

| Flow | Trigger | What It Does |
|------|---------|-------------|
| **Hardware Asset Request** | Employee submits catalog request | Reserves asset from stockroom → creates deployment task → assigns to user |
| **Stock Order** | Stock rule threshold breached | Creates purchase request → routes for approval → generates PO |
| **Refresh/Replace** | EOL approaching or asset flagged for replacement | Creates replacement request → orders new asset → schedules old asset for retirement |
| **Asset Reclamation** | Employee departure or asset recalled | Notifies employee/manager → creates recovery task → returns asset to stockroom |
| **Disposal** | Asset reaches Disposed state | Creates disposal order → generates chain-of-custody documentation → updates financial records |
| **Loaner** | Temporary asset requested | Reserves loaner → assigns with return date → auto-reclaims when due |
| **RMA (Return to Manufacturer)** | Warranty or defective return | Creates RMA record → generates shipping label → tracks return/replacement |
| **Contract Renewal** | Contract approaching expiry | Notifies asset manager → creates renewal task 90/60/30 days before expiry |

---

## 9. Contract Management

### Contract Lifecycle
```
Draft → Active → [Renewal Pending] → Renewed / Expired / Cancelled
```

### Contract Types
| Type | Description |
|------|-------------|
| **Lease** | Periodic payments for asset use; asset returned at end |
| **Maintenance** | Vendor support and repair agreements |
| **Warranty** | Manufacturer defect coverage |
| **Purchase** | Ownership agreement |
| **Subscription** | Recurring service contracts |

### Contract Notifications
OOTB scheduled notifications fire at:
- **90 days** before expiry → assigned to contract manager
- **60 days** before expiry → escalated
- **30 days** before expiry → urgent escalation

### Contract → Asset Relationship
Multiple assets can be linked to one contract (e.g., a 3-year lease covering 500 laptops). When the contract expires, all linked assets are flagged for renewal/return.

---

## 10. Hardware Asset Dashboard — 5 Tabs

The Hardware Asset Dashboard provides the primary operational view for HAM managers:

| Tab | What It Shows |
|-----|--------------|
| **Asset Health** | Overall asset state distribution, unassigned assets, assets past due for action |
| **Model Management** | Product model completeness, normalization status, model catalog coverage |
| **Procurement** | Open POs, spending by category, vendor performance |
| **Inventory** | Stockroom levels, stock rule compliance, assets in transit |
| **End of Life** | Assets approaching warranty expiry and EOL by 90/60/30/0 day bands |

---

## 11. Discovery & CMDB Integration

### How Discovery Feeds HAM
ServiceNow Discovery (or Service Graph Connectors) scans the network and creates/updates both:
- **CI records** in CMDB (`cmdb_ci_computer`, etc.)
- **Hardware Asset records** (`alm_hardware`) — linked to the CI

The CMDB and Asset record are synced: the asset's `ci` field points to the CI; the CI's `asset` field points back.

### Identification & Reconciliation Engine (IRE)
The IRE prevents duplicate records by:
- Matching incoming discovery data against existing CMDB records using serial number, MAC address, hostname
- Reconciling data from multiple discovery sources (SCCM + native Discovery + Intune) into one authoritative CI
- Preventing phantom records when IPs or hostnames change

### Service Graph Connectors
Pre-built integrations for common discovery tools:
| Connector | Source | What It Discovers |
|-----------|--------|------------------|
| **SCCM** (Config Manager) | Microsoft | Windows devices, installed software |
| **Intune** | Microsoft | Mobile/Windows managed devices, MDM compliance |
| **Jamf** | Jamf | macOS and iOS devices |
| **Tanium** | Tanium | Endpoint visibility, security posture |

Each connector uses a MID Server, pulls data via JDBC/REST, stages it in import tables, and transform maps push it to CMDB + Asset tables.

---

## 12. Key Roles

| Role | Access |
|------|--------|
| `asset` | View and manage asset records |
| `asset_manager` | Full HAM configuration — models, stockrooms, flows |
| `inventory_user` | Stockroom operations — receive, transfer, audit |
| `itil` | View assets linked to incidents/changes (read-only) |
| `procurement_user` | Create and manage purchase orders |

---

## 13. HAM vs. SAM vs. ITAM — The Hierarchy

```
ITAM (IT Asset Management) — umbrella discipline
  ├── HAM — Hardware Asset Management (physical devices)
  ├── SAM — Software Asset Management (licenses and compliance)
  └── Cloud ITAM — Cloud resource tracking
```

**ITAM** = the program and governance model  
**HAM** = the physical asset tracking and lifecycle execution  
**SAM** = the license compliance and optimization  

They share the same platform and data — a hardware asset has SAM software allocations attached to it; retiring the hardware triggers SAM to reclaim the software licenses.

---

## 14. Financial Tracking in HAM

| Capability | Description |
|-----------|-------------|
| **Cost basis** | Purchase price, including shipping and setup costs |
| **Depreciation** | Calculated daily by scheduled job; straight-line or reducing balance |
| **Residual value** | Remaining value at retirement — feeds financial write-off |
| **Total Cost of Ownership** | Acquisition + maintenance + support + disposal |
| **CapEx vs. OpEx** | Asset classification for financial reporting |
| **Budget forecasting** | EOL dates + replacement cost = hardware refresh budget |

---

## 15. Glossary

| Term | Definition |
|------|-----------|
| **HAM** | Hardware Asset Management — lifecycle tracking of physical IT assets |
| **HAM Pro** | Enhanced HAM with AI automation, Content Service, and advanced workflows |
| **Product Model** | Template describing hardware type (manufacturer, model, class) |
| **Content Service** | HAM Pro feature — automated product model normalization library |
| **Stockroom** | Physical storage location for undeployed assets |
| **Stock Rule** | Minimum inventory level threshold per model per stockroom |
| **Transfer Order** | Record tracking asset movement between locations |
| **Asset Task** | Workflow action item at a lifecycle stage |
| **Asset Action** | IT agent's choice when closing incident/change (Swap, Retire, etc.) |
| **IRE** | Identification and Reconciliation Engine — prevents CMDB duplicates |
| **Service Graph Connector** | Pre-built integration bringing discovery data into CMDB/HAM |
| **EOL** | End of Life — vendor stops selling/supporting the product |
| **EOM** | End of Maintenance — patches and security updates stop |
| **Disposal Order** | Record tracking physical asset disposal at end of life |
| **CMDB** | Configuration Management Database — stores CI records linked to assets |
| **TCO** | Total Cost of Ownership — all costs across asset lifecycle |
| **MID Server** | ServiceNow component that bridges network/firewall for discovery |
