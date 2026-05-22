# ServiceNow HAM — Common Issues, Pitfalls & Troubleshooting

Issues encountered during HAM implementations and ongoing support, based on ServiceNow community posts, Known Error Portal articles, implementation experience, and real-world project work.

---

## Issue 1: Discovery Creating Duplicate Hardware Asset Records

**Category:** Configuration / CMDB  
**Severity:** Critical — inflates asset counts; corrupts financial reporting; destroys trust in the system

**The Problem:**
Every Discovery run creates new Hardware Asset records instead of updating existing ones. A 3,000-device environment accumulates 9,000+ asset records within months. IT reports show triple the actual asset count. Finance can't reconcile depreciation because the same device has 3 asset records with different cost values.

**Why It Happens:**
The Identification & Reconciliation Engine (IRE) is not configured to use stable hardware identifiers. Discovery defaults to IP address or hostname as the matching key — both are volatile. When a device gets a new DHCP address or the hostname changes after a rebuild, Discovery can't find the existing record and creates a new one.

**Root Cause:**
Identification Rules for the Hardware (`cmdb_ci_computer`) CI class use volatile attributes instead of permanent ones.

**Fix / Prevention:**
- Navigate to **Identification Rules** for `cmdb_ci_computer` — set Serial Number as the primary identifier, MAC Address as secondary
- Run the CMDB **Deduplication** tool to merge existing duplicates (available in the CMDB workspace)
- Use the **CMDB Success Advisor** to score CI data quality before and after remediation
- After each Discovery run: monitor "New CIs created" count — a spike signals identifier problems
- IRE best practice: Serial Number → MAC Address → Asset Tag → Hostname (in that priority order)

**Real-world impact:** On one Dell multi-tenant project, hostname was used as the primary identifier. When machines were reimaged with standard naming conventions, every device got a "new" CI and asset record — 47,000 duplicate records generated in 2 weeks.

---

## Issue 2: Asset States Out of Sync — CMDB Shows "In Use," Asset Shows "Retired"

**Category:** Data Quality / Integration  
**Severity:** High — SecOps finds vulnerabilities on assets no one thinks are active; incident routing fails

**The Problem:**
A laptop is retired and the Asset record updated to "Retired" — but the linked CI in CMDB still shows "Operational." SecOps vulnerability scans find the IP (from a reassigned address) and create a vulnerability finding linked to the old CI. The CI shows an active server with no asset owner — nobody knows who's responsible.

Or the reverse: a new device is deployed and the CI is created by Discovery, but the Asset record stays "In Stock" because the deployment task was never completed in ServiceNow.

**Why It Happens:**
The Asset record (`alm_hardware`) and CI record (`cmdb_ci`) are linked but NOT automatically synchronized in all scenarios. They're managed by different teams (ITAM vs. CMDB/ITOM) with different workflows.

**Root Cause:**
No Business Rule syncing Asset state changes to CI state, and vice versa.

**Fix / Prevention:**
- Create a Business Rule on `alm_hardware`: when asset state changes to "Retired" → set linked CI `install_status` to "Retired"
- Create a Business Rule on `alm_hardware`: when asset state = "In Use" AND CI install_status != "Installed" → sync the CI
- Monthly reconciliation report: query assets and CIs where states are out of sync — assign to ITAM team to resolve
- Assign single team ownership for both asset + CI for each device class — split ownership is the root cause of drift

---

## Issue 3: Asset Lifecycle Flows Not Triggered — Manual Workarounds Everywhere

**Category:** Process / Adoption  
**Severity:** High — automation exists but isn't used; IT does everything manually

**The Problem:**
ServiceNow has OOTB lifecycle automation flows (Hardware Asset Request, Stock Order, Refresh, Reclamation) but IT staff are processing assets manually — emailing stock requests, manually updating states, using spreadsheets for tracking. The flows are configured but no one uses them.

**Why It Happens:**
- IT staff don't know the automated flows exist (training gap)
- The flows require data preconditions (stockroom assigned, product model linked, stock rules configured) that weren't set up during implementation
- Procurement flows weren't connected to the catalog — employees order hardware via email instead of the Service Catalog
- Flow configurations are incomplete: flows exist but approval policies or notification emails weren't configured

**Root Cause:**
Lifecycle flows require a complete foundational setup: stockrooms, product models, stock rules, catalog items, and approval groups. If any piece is missing, flows fail or produce no output — and users revert to manual processes.

**Fix / Prevention:**
- Crawl → Walk → Run: prove the manual process works in ServiceNow first, then enable automation
- Stockroom setup checklist: before enabling flows, verify every site has a stockroom record, every stockroom has assignment groups, and stock rules are configured for the top 10 device models
- Catalog integration: wire the "Request Hardware" catalog item to the Hardware Asset Request flow — don't let employees email for hardware
- Governance: establish that any hardware request not submitted through ServiceNow catalog is rejected — enforcement drives adoption
- Train procurement and IT receiving teams specifically on the flows that affect their work

---

## Issue 4: Stockroom Inventory Inaccurate — Physical Doesn't Match System

**Category:** Process / HAM  
**Severity:** Medium-High — new hardware purchased unnecessarily; available assets sit unused

**The Problem:**
ServiceNow shows 12 laptops "In Stock" in the Austin stockroom, but physically there are only 7 — 5 were deployed last week without anyone updating ServiceNow. Or, the system shows 0 laptops available so IT approves a $25,000 hardware purchase, but a warehouse count reveals 8 perfectly good laptops sitting in boxes.

**Why It Happens:**
IT receiving staff physically process hardware (unbox, prep, deploy) but skip the ServiceNow asset task step. The physical operation and the system operation are decoupled.

**Root Cause:**
Asset receiving and deployment tasks in ServiceNow are optional from the IT tech's perspective — they can physically handle the hardware without ever opening ServiceNow. Without enforcement, they skip it.

**Fix / Prevention:**
- Make ServiceNow the **trigger** for physical work, not the recording step after
- Barcode scanning at receiving: scan the asset tag → ServiceNow mobile app → automatic stockroom assignment (HAM Pro feature)
- Receiving task workflow: the IT receiving tech cannot close the shipment until all asset records are updated in ServiceNow
- Real-time stockroom dashboard visible to IT managers and procurement — stock levels visible before any new PO is approved
- Quarterly physical audit: count physical stock, compare to ServiceNow records, investigate every discrepancy
- Stock rule compliance metric: track the % of stock rules where physical count matches system — target 98%+

---

## Issue 5: End-of-Life Assets Not Tracked or Actioned

**Category:** Reporting / SecOps Integration  
**Severity:** Medium-High — security risk; unplanned hardware refresh costs

**The Problem:**
The HAM system has warranty expiry and EOL dates on asset records, but no process exists to act on them. IT discovers that 200 production laptops are running on expired warranties with no vendor support — discovered during a security audit, not from the ITAM dashboard. Emergency refresh project costs 40% more than planned because there was no lead time for procurement.

**Why It Happens:**
EOL tracking data is captured in the asset records, but nobody has subscribed to the "End of Life" tab on the Hardware Asset Dashboard or configured automated notifications. The data is there; the process to act on it isn't.

**Root Cause:**
ITAM data without a consumption process has no value. EOL dates are passive data until someone actively monitors and acts on them.

**Fix / Prevention:**
- Configure scheduled reports: "Assets reaching EOL in 90 / 60 / 30 days" — auto-emailed to IT managers and procurement weekly
- Hardware Asset Dashboard → EOL tab: make this a required weekly review for the HAM team
- Trigger the OOTB **Refresh** lifecycle flow automatically when an asset hits EOL − 180 days — creates a refresh project record in SPM
- SecOps integration: flag EOL assets with elevated vulnerability priority in Security Operations — EOL = no security patches = higher risk score
- Budget planning: EOL report quarterly → feeds annual hardware refresh budget forecast
- Content Service (HAM Pro): EOL dates auto-updated from manufacturer data — no manual maintenance

---

## Issue 6: Service Graph Connector (SCCM/Intune) Not Populating Asset Records

**Category:** Integration / Discovery  
**Severity:** High — HAM has no device data; everything is manually entered

**The Problem:**
SCCM connector is installed and running. CMDB is getting CI records. But the `alm_hardware` (Asset) records are NOT being created or updated from SCCM data — the CI exists without an Asset record, or the Asset record has no hardware details (serial, model, manufacturer).

**Why It Happens:**
The Service Graph Connector creates CI records in CMDB by default. Asset record creation is a **separate configuration** — the transform maps and connection between the CI and `alm_hardware` must be explicitly enabled. Many implementations configure the connector for CMDB population but forget the asset side.

**Root Cause:**
- Asset creation from discovery not enabled in connector configuration
- Transform maps don't include `alm_hardware` table as a target
- IRE reconciliation creates the CI but the "create asset if not exists" option is not checked
- Staging table data (e.g., `sn_sccm_integrate_sccm_2019_computer`) not being transformed to Asset records

**Fix / Prevention:**
- In the Service Graph Connector configuration: explicitly enable "Create Hardware Asset record" option
- Verify transform maps include `alm_hardware` as a target in addition to `cmdb_ci_computer`
- Check the staging table for the connector — confirm data is arriving (records exist in staging) before troubleshooting transform
- Business Rule option: after CI is created/updated by Discovery, check if linked Asset record exists — if not, auto-create it
- Test with a known device: run Discovery → verify CI created → verify Asset created → verify they are linked

---

## Issue 7: Product Models Missing or Incorrect — Asset Reports Are Meaningless

**Category:** Data Quality / Normalization  
**Severity:** Medium — reports aggregate incorrectly; EOL tracking fails; refresh planning impossible

**The Problem:**
The hardware asset inventory shows 4,500 assets, but 2,800 of them have "Unknown" as the Product Model. EOL date reports can't run because the EOL date is on the model record — if there's no model, there's no EOL date. Hardware refresh planning is impossible because you can't report "how many Dell Latitude 5540s do we have" when they're all categorized differently.

**Why It Happens:**
Product Models require normalization — either manual creation or automated matching. Without the HAM Pro Content Service, models must be manually created and mapped to discovery data. Discovery creates CIs with manufacturer/model data from the source system — but if those strings don't exactly match a Product Model record, no match occurs.

**Root Cause:**
- "Dell, Inc." in SCCM → "Dell" in ServiceNow Product Model → no match
- "LATITUDE 5540" in SCCM → "Latitude 5540" in Product Model → no match (case-sensitive)
- Product Models not created during implementation — assumed Discovery would auto-create them

**Fix / Prevention:**
- HAM Pro: enable Content Service — it auto-normalizes to the correct manufacturer/model regardless of input format
- Standard HAM: create normalization rules in the Software Library-equivalent for hardware — map incoming strings to canonical Product Model records
- Data import: bulk-import Product Models from vendor catalogs (Dell, HP, Lenovo all publish machine-readable hardware catalogs)
- Normalization audit: monthly report of assets with null or "Unknown" Product Model → assign to data quality team to resolve
- At go-live: require that the top 50 device models are created and linked before launching reporting

---

## Issue 8: Contracts Not Linked to Assets — Warranty and Support Gaps Unknown

**Category:** Contract Management  
**Severity:** Medium — warranty claims fail; support tickets rejected; unplanned support costs

**The Problem:**
IT opens a support ticket with Dell for a failed laptop. Dell rejects it: "This device is out of warranty." IT didn't know because ServiceNow has the contract record but the laptop was never linked to that contract. 200 other laptops might also be in the same situation — no way to know without checking each one individually.

**Why It Happens:**
Contracts and assets are managed in different workflows. The procurement team creates the contract record when the PO is finalized. The receiving team creates the asset records when hardware arrives. The link between them must be manually established — and it's commonly skipped.

**Root Cause:**
No automated process links asset records to contracts at the time of receipt. It requires manual effort that receiving staff don't prioritize.

**Fix / Prevention:**
- Automate contract-asset linking during the receiving flow: when assets arrive on a PO line, the flow should automatically link those assets to the contract associated with that PO
- Procurement flow enhancement: PO → Contract → line items → when assets are received against those line items, auto-link to contract
- Contract coverage report: assets with no linked contract flagged for review monthly
- Before go-live: bulk-link all existing asset records to their corresponding contracts (one-time data fix using scripts)
- Warranty lookup integration: use vendor APIs (Dell TechDirect, HP warranty API) to auto-populate warranty end dates by serial number

---

## Issue 9: HAM Workspace Not Visible — Roles Not Configured

**Category:** Configuration / Access  
**Severity:** Medium — HAM team can't access the tools they need

**The Problem:**
A newly hired IT Asset Manager opens ServiceNow and can't find the Hardware Asset Workspace. Navigation shows ITSM tools but nothing HAM-specific. They're doing asset management through the regular list views.

**Why It Happens:**
The Hardware Asset Workspace requires specific roles. Standard `itil` role does not include HAM workspace access. Additionally, the HAM Pro workspace requires the `asset_manager` role — the `asset` role alone gives list-level access but not the full workspace experience.

**Root Cause:**
HAM roles and ITSM roles are separate. New IT staff onboarded with ITIL roles don't automatically get HAM access.

**Fix / Prevention:**
- HAM role matrix:
  - `asset`: basic asset list view, can update asset records
  - `asset_manager`: full HAM workspace, configuration, flows, dashboards
  - `inventory_user`: stockroom operations — receive, transfer, audit
  - `procurement_user`: PO creation and management
- Onboarding process for ITAM staff: separate from ITSM onboarding — include HAM-specific role assignment
- Known Error: if HAM Pro Workspace isn't visible after role assignment, check that the HAM Pro plugin (`com.snc.sn_hamp`) is activated on the instance

---

## Issue 10: Disposal Without Documentation — Audit and Compliance Risk

**Category:** Process / Compliance  
**Severity:** High — GDPR, HIPAA, and financial audit exposure

**The Problem:**
Assets are physically disposed of (sent to recycling vendor, destroyed, donated) without a Disposal Order in ServiceNow. The asset record is either deleted, stuck at "Retired" state, or updated informally. When an auditor asks "prove that the data on device #12345 was wiped before disposal" — there is no documentation trail.

**Why It Happens:**
IT teams treat disposal as the end of their work — once it's physically gone, they consider the job done. Creating a Disposal Order in ServiceNow feels like extra admin work with no immediate payoff.

**Root Cause:**
Disposal workflow is not enforced — IT can physically dispose of hardware without completing the ServiceNow Disposal Order. The process is advisory, not mandatory.

**Fix / Prevention:**
- Use the OOTB **Disposal** lifecycle flow: it creates a Disposal Order record, captures the disposal method, vendor, date, and data wipe certification
- Enforce: assets cannot transition to "Disposed" state without an associated Disposal Order (add state transition validation)
- Data wipe certification: require attachment of wipe certificate (from Blancco, DBAN, or similar tool) before Disposal Order can be closed
- Vendor management: disposal vendor must provide certificate of destruction → uploaded to the Disposal Order → linked to the asset record
- Compliance reporting: quarterly report of assets in "Retired" state for more than 60 days with no Disposal Order → escalate to IT manager

---

## Quick Troubleshooting Reference

| Symptom | Likely Cause | First Step |
|---------|-------------|-----------|
| Duplicate asset records after Discovery | IRE using volatile identifiers (IP/hostname) | Switch identification rules to Serial Number + MAC |
| Asset "In Stock" but CMDB CI shows "Operational" | No Business Rule syncing state changes | Add BR: Asset retired → set linked CI to Retired |
| SCCM connector running but no Asset records created | Asset creation not enabled in connector config | Enable "Create Hardware Asset" in connector settings |
| EOL dates missing on assets | Product Model not linked or Content Service not enabled | Link models; enable Content Service (HAM Pro) |
| HAM Workspace not visible | Missing asset_manager role | Assign correct HAM roles (asset, asset_manager, inventory_user) |
| Contracts not linked to assets | Manual linking never done | Automate contract-asset link in receiving flow |
| Stockroom count doesn't match physical | Asset tasks being skipped | Enforce receiving tasks; add barcode scanning |
| Disposal record missing for retired asset | No enforcement on disposal workflow | Add state validation: Disposed requires Disposal Order |
| Lifecycle flows not triggering | Missing prerequisites (stock rules, models, stockrooms) | Complete foundational setup before enabling flows |
| Vulnerability finding with no asset owner | CI exists without linked Asset record | Run CI-to-Asset reconciliation script; enforce IRE asset creation |
