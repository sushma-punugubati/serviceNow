# ServiceNow ITAM — Fundamentals & Study Guide

> Sources: [TechPratham ITAM Guide](https://www.techpratham.com/blog/top-65-servicenow-it-asset-management-itam-interview-questions-and-answers-for-2026) | [ServiceNow ITAM Community](https://www.servicenow.com/community/virtual-agent-forum/itam-interview-qustion/m-p/2840624)

---

## 1. What is ITAM?

**IT Asset Management (ITAM)** is ServiceNow's centralized solution for tracking, controlling, and optimizing IT assets across their entire lifecycle — from the moment you decide to buy something to the moment you retire it.

> **Think of it as:** The single source of truth for everything your organization owns — every laptop, server, software license, and cloud subscription — who has it, what it costs, whether it's compliant, and when it needs to be replaced.

**ITAM is an umbrella covering:**
- **HAM** — Hardware Asset Management (physical devices)
- **SAM** — Software Asset Management (licenses and usage)
- **Cloud Asset Management** — Cloud resource tracking and cost optimization
- **Mobile Asset Management** — Mobile device tracking

---

## 2. HAM vs SAM — The Critical Distinction

| Aspect | HAM (Hardware) | SAM (Software) |
|--------|---------------|----------------|
| **What it tracks** | Physical devices: laptops, servers, desktops, printers | Software licenses, subscriptions, SaaS usage |
| **Key concern** | Location, ownership, condition, EOL dates | License compliance, over/under-purchasing |
| **Main table** | `alm_hardware` | `alm_license` |
| **Key process** | Lifecycle from purchase to disposal | Entitlement vs. actual usage reconciliation |
| **Risk if ignored** | Lost/stolen assets, untracked costs | Audit fines, compliance violations |
| **Discovery role** | SCCM, Intune, barcode scanning | Software normalization, publisher packs |

---

## 3. Asset Lifecycle — 6 Stages

```
Planning → Procurement → Receiving → Deployment → Maintenance → Retirement
```

| Stage | What Happens | Key Actions |
|-------|-------------|-------------|
| **Planning** | Identify need, budget, vendor selection | Create purchase request, approval workflow |
| **Procurement** | Purchase order created, vendor engaged | PO tracking, contract signing |
| **Receiving** | Physical/digital asset received | Asset record created, serial number logged |
| **Deployment** | Asset assigned to user or location | Asset task created, CMDB CI linked |
| **Maintenance** | Ongoing support, repairs, upgrades | Incident/change links, warranty tracking |
| **Retirement** | End of life — data wipe and disposal | Disposal record, financial write-off, audit trail |

---

## 4. Key Tables

| Table Name | Label | Purpose |
|-----------|-------|---------|
| `alm_asset` | Asset | Base asset table — all assets |
| `alm_hardware` | Hardware Asset | Physical hardware |
| `alm_license` | Software License | Software license records |
| `ast_contract` | Contract | Vendor/support contracts |
| `alm_stockroom` | Stockroom | Asset storage locations |
| `alm_asset_task` | Asset Task | Workflow tasks for asset lifecycle actions |
| `cmdb_ci` | Configuration Item | Linked CI in CMDB (asset → CI relationship) |
| `sn_sam_sw_install` | Software Installation | Tracks what software is installed where |
| `sn_sam_publisher_pack` | Publisher Pack | Pre-built vendor license models |
| `procurement_request` | Procurement Request | Purchase requests |
| `po_order_line` | Purchase Order Line | Individual line items on purchase orders |

---

## 5. Asset vs CI — Key Difference

| Asset | Configuration Item (CI) |
|-------|------------------------|
| Financial/inventory record | Logical/physical infrastructure record |
| Tracks cost, depreciation, ownership | Tracks relationships, dependencies, status |
| Table: `alm_asset` | Table: `cmdb_ci` |
| Managed by IT Asset team | Managed by CMDB/ITOM team |
| Exists from procurement | Exists when in service |

> An asset and CI are **linked** — the asset record points to its CI counterpart. They serve different purposes but share data.

---

## 6. Software Asset Management (SAM) Deep Dive

### Software License Types
| Type | Description |
|------|-------------|
| **Named User** | One specific user per license |
| **Concurrent** | X users simultaneously |
| **Device-based** | Licensed per device, not user |
| **Enterprise Agreement** | Site-wide or org-wide license |
| **Subscription/SaaS** | Monthly/annual user subscriptions |
| **OEM** | Bundled with hardware |

### License Compliance Formula
```
Compliance Position = Entitlements Owned - Actual Usage
  Positive = Over-licensed (wasted money)
  Negative = Under-licensed (AUDIT RISK!)
```

### Publisher Packs
Pre-built license models for major vendors (Microsoft, Oracle, IBM, Adobe) that include:
- Predefined normalization rules
- License metric templates
- Compliance calculation logic

> **Why they matter:** Without Publisher Packs, teams must manually build compliance rules for each vendor. Publisher Packs automate 80% of SAM setup for major vendors.

### Software Normalization
Standardizes product names discovered across the environment:
- "MS Office 365" + "Microsoft Office365" + "Office 365 E3" → all normalized to one record
- Eliminates duplicates in compliance calculations
- Required before any meaningful SAM reporting

### License Workbench
The central SAM dashboard showing:
- Compliance status per product
- Usage vs. entitlements
- Optimization recommendations (unused licenses to reclaim)
- Audit readiness score

---

## 7. Hardware Asset Management (HAM) Deep Dive

### Hardware Asset Workspace
A unified interface for:
- Managing the complete hardware lifecycle
- Viewing dashboards for asset health and utilization
- Processing asset tasks (deployment, repairs, retirement)
- Stockroom management

### Asset Tasks
Workflow-driven actions triggered at each lifecycle stage:
- **Receiving task:** Verify asset, update serial number, store in stockroom
- **Deployment task:** Assign to user, update location, link to CI
- **Repair task:** Route to IT support, track repair status
- **Retirement task:** Wipe data, remove from service, update financial records

### Stockroom Management
Tracks where unassigned assets physically live:
- Multiple stockroom locations supported
- Assets moved between stockrooms with tracked history
- Stockroom audits to reconcile physical vs. system records

---

## 8. Key Integrations

| Integration | Purpose |
|------------|---------|
| **ServiceNow Discovery** | Auto-discovers hardware and installed software; populates CMDB and asset records |
| **SCCM / Intune** | Windows device and software inventory |
| **JAMF** | macOS/iOS device management |
| **SAP/Oracle ERP** | Financial data sync (purchase orders, depreciation) |
| **SecOps** | Asset data supports vulnerability prioritization |
| **HRSD** | Employee onboarding/offboarding triggers asset provisioning/recovery |
| **Change Management** | Change requests reference assets for impact assessment |

---

## 9. Cloud Asset Management

ServiceNow ITAM extends to cloud:
- Tracks cloud resource usage (AWS, Azure, GCP)
- Identifies unused/oversized instances
- Prevents cloud sprawl and shadow IT
- Cost optimization recommendations
- SaaS subscription management (track active vs inactive users)

---

## 10. Key Roles

| Role | Responsibilities |
|------|----------------|
| **Asset Manager** | Oversees entire asset lifecycle program |
| **Asset Coordinator** | Day-to-day asset tracking, intake, and distribution |
| **Software Asset Manager** | SAM compliance, license reconciliation, vendor audits |
| **Procurement Manager** | Purchase orders, vendor contracts |
| **IT Finance** | Depreciation, cost allocation, financial reporting |
| **Discovery Admin** | Maintains discovery schedules and normalization rules |

---

## 11. Financial Management in ITAM

- **Depreciation tracking:** Asset value reduces over useful life (straight-line or reducing balance)
- **Cost allocation:** Distribute asset costs to business units/cost centers
- **Budget forecasting:** Predict renewal and refresh costs based on lifecycle data
- **CapEx vs OpEx classification:** Helps financial reporting compliance
- **TCO (Total Cost of Ownership):** Purchase + maintenance + support + disposal

---

## 12. Discovery's Role in ITAM

ServiceNow Discovery:
1. Scans the network for devices and applications
2. Creates/updates CI records in CMDB
3. Links discovered hardware to Asset records
4. Reports installed software for SAM normalization
5. Feeds publisher packs for license compliance calculation

> **Without Discovery:** ITAM relies on manual entry and spreadsheets — error-prone and expensive.

---

## 13. Glossary

| Term | Simple Explanation |
|------|-------------------|
| **HAM** | Hardware Asset Management — physical devices |
| **SAM** | Software Asset Management — license compliance |
| **ITAM** | Umbrella: hardware + software + cloud + mobile |
| **Publisher Pack** | Pre-built vendor rules for license compliance |
| **Normalization** | Standardizing product names to eliminate duplicates |
| **License Workbench** | Central SAM compliance dashboard |
| **Entitlement** | How many licenses you legally own |
| **Compliance Position** | Entitlements minus actual usage (positive = good) |
| **Stockroom** | Physical storage location for undeployed assets |
| **Asset Task** | Workflow action at a lifecycle stage |
| **Depreciation** | Reduction in asset value over time |
| **EOL** | End of Life — vendor stops supporting the product |
| **EOM** | End of Maintenance — patches and updates stop |
| **TCO** | Total Cost of Ownership — all costs across lifecycle |
| **Shadow IT** | Unauthorized software/cloud usage outside IT control |
