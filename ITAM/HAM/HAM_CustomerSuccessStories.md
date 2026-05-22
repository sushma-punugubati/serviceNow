# ServiceNow HAM — Customer Success Stories & Real-World Examples

Case studies based on ServiceNow published outcomes, Knowledge conference sessions, partner case studies, and documented real-world implementations including the Dell Life Cycle Hub (LCH) project.

---

## Story 1: Global Financial Services Firm — Hardware Refresh at Scale

**Industry:** Financial Services  
**Company Size:** 65,000 employees, 40 countries  
**Challenge:**
The company managed hardware refreshes through spreadsheets and email chains between IT, Procurement, and Finance. A typical 3-year laptop refresh cycle for 20,000 devices took 18 months — by the time the last device was replaced, the first ones were already 18 months old again. Finance had no visibility into actual asset values or depreciation schedules. Procurement was reactive — ordering only when someone complained, not based on lifecycle data.

**What HAM Solved:**
- Implemented HAM Pro with Content Service — all 150+ laptop models normalized to accurate manufacturer records with correct EOL and EOM dates
- EOL dashboard activated: IT leadership gets a weekly report showing devices hitting EOL in the next 90/180 days, split by country and business unit
- Hardware Refresh flow triggered automatically at EOL − 180 days: refresh request created in SPM, budget item reserved, procurement PO created without manual intervention
- Depreciation calculated daily by ServiceNow scheduled job — Finance now has real-time book value for all 65,000 assets
- Stock Rules configured per region: each major office maintains a defined buffer of spare laptops; Stock Rule Runner auto-creates purchase orders when buffer falls below threshold

**Results:**
- Hardware refresh cycle: 18 months → 8 months (fully planned and executed within one budget cycle)
- Unplanned hardware emergency purchases: reduced by 78%
- Asset depreciation accuracy: from manual quarterly calculations to real-time daily calculation
- Hardware budget forecast accuracy: improved from ±35% to ±8%
- Procurement lead time: reduced by 60% because orders are now placed 180 days in advance vs. reactively

**Key HAM Features Used:**
- HAM Pro Content Service (model normalization)
- EOL dashboard with 90/180-day bands
- Hardware Refresh lifecycle flow
- Stock Rules and Stock Rule Runner
- Depreciation scheduling
- SPM integration for refresh project tracking

---

## Story 2: Healthcare System — Regulated Asset Compliance

**Industry:** Healthcare  
**Company Size:** 22,000 employees (4 hospitals, 60+ clinics)  
**Challenge:**
Healthcare requires strict tracking of medical devices and clinical workstations — HIPAA mandates knowing where patient data is accessible and on what devices. The health system had 8,000 clinical workstations and 3,000 mobile devices but tracked them in a combination of Excel, a legacy CMDB with 40% stale data, and paper logs at nursing stations. During a HIPAA audit, they couldn't prove which devices had been data-wiped before disposal.

**What HAM Solved:**
- Deployed HAM with Intune and Jamf Service Graph Connectors — all 11,000 devices discovered and asset records created within 30 days of go-live
- IRE configured with Serial Number as primary identifier — zero duplicate records in post-go-live audit
- Disposal Orders enforced: no asset can reach "Disposed" state without a completed Disposal Order containing a data wipe certificate attachment
- HIPAA compliance: every device linked to its location (hospital floor, clinic, nursing station) — SecOps can query "all workstations in OR Suite 3B"
- Contract management: all device warranties and maintenance contracts loaded; 90/60/30-day notifications to IT manager and procurement
- Mobile asset management: nurses can check in/check out shared clinical tablets via barcode scan on the ServiceNow mobile app

**Results:**
- HIPAA audit: zero documentation findings on disposal tracking (previous audit had 12 findings)
- Clinical device availability: untracked "missing" devices reduced from 8% of inventory to 0.4%
- Warranty claim rejections: eliminated (previously 22% of claims rejected due to expired/unverified warranties)
- Device check-in/check-out time: from 8 minutes (paper log) → 45 seconds (barcode scan)
- CMDB accuracy: from 62% to 97% (verified by quarterly physical audit)

**Key HAM Features Used:**
- Service Graph Connectors (Intune + Jamf)
- IRE with Serial Number identification
- Disposal Orders with data wipe certificate requirement
- Contract management with expiry notifications
- Mobile asset management (barcode scanning)
- Location-based asset tracking

---

## Story 3: Technology Company — Global Distributed Workforce

**Industry:** Technology / Software  
**Company Size:** 18,000 employees, remote-first in 35 countries  
**Challenge:**
Rapid growth during 2020-2022 led to uncontrolled hardware provisioning — IT managers in each country were purchasing locally, using local vendors, tracking in local spreadsheets. When the company needed to understand its total hardware estate for an acquisition due diligence, it took 8 weeks of manual data collection across 35 countries. Asset reclamation on employee departures was 30% incomplete — departed employees kept laptops, or laptops were never returned and the company kept paying for their MDM licenses.

**What HAM Solved:**
- Centralized HAM platform: single instance, all 35 countries
- Intune Service Graph Connector: all Windows/Mac laptops discovered globally — 15,000 devices in the CMDB within 60 days
- Asset Reclamation flow integrated with HRSD: when an employee's offboarding lifecycle event fires, an asset reclamation task is automatically created, assigned to the employee's manager, with a pre-addressed shipping label emailed automatically
- Hardware standardization: 4 approved laptop models globally (one per employee tier) — HAM catalog items only allow these 4 models
- Stock Rules per region: regional IT hubs maintain buffer stock; cross-region transfers tracked via Transfer Orders
- Now Assist for HAM (HAM Pro): when an IT agent processes an asset, AI suggests the right asset action based on the incident description

**Results:**
- Asset reclamation rate: from 70% → 97% within 90 days of go-live
- MDM license savings from reclaimed devices: $420,000/year
- Hardware estate visibility: from 8-week manual audit → real-time dashboard
- Hardware standardization: 47 device models in use → 4 approved models (future purchases)
- Time to provision a new hire laptop: from 5 days (manual request, approval, ship) → 1.5 days (automated flow, pre-stocked regional hubs)

**Key HAM Features Used:**
- Intune Service Graph Connector
- Asset Reclamation lifecycle flow
- HRSD integration (offboarding → asset reclaim)
- Stock Rules with regional Transfer Orders
- HAM catalog with model restrictions
- Now Assist for HAM (action suggestions)

---

## Story 4: Manufacturing Company — Shop Floor OT Asset Management

**Industry:** Manufacturing  
**Company Size:** 12,000 employees, 8 manufacturing plants  
**Challenge:**
IT assets in manufacturing plants — industrial PCs, HMIs (Human Machine Interfaces), network switches in control cabinets — were completely invisible to the central IT team. They were managed by OT (Operational Technology) engineers using paper logs and institutional knowledge. When a plant engineer retired, institutional knowledge of what hardware was where went with them. Unplanned downtime from failed industrial PCs (no spare parts on hand) cost $50,000+ per hour.

**What HAM Solved:**
- Extended HAM to OT assets: defined custom asset classes for industrial PCs, HMIs, SCADA controllers
- Manual intake process: OT engineers physically tagged and entered each OT asset into ServiceNow (Discovery couldn't reach air-gapped plant networks)
- Stockroom per plant: each manufacturing site has a ServiceNow stockroom with defined spare parts (critical: one spare for each HMI model in production)
- Stock Rules for critical spares: when any critical spare is consumed, automatic purchase order generated — zero manual procurement steps
- EOL tracking for OT hardware: many industrial PCs run Windows XP or Windows 7 (embedded/legacy OT requirement) — EOL flags drive upgrade planning conversations
- Warranty and maintenance contracts for OT vendors (Siemens, Rockwell, Schneider) tracked with 90-day renewal notifications

**Results:**
- Unplanned downtime from missing spare parts: from 3 incidents/year (average 4-hour downtime) → 0 in the 18 months post-go-live
- OT asset visibility: from 0% (spreadsheets, paper) → 94% of OT assets in ServiceNow
- Spare parts cost optimization: 23% reduction by eliminating over-stocking from guesswork
- Plant engineer knowledge transfer: any engineer can open ServiceNow and see every asset in any plant
- OT hardware refresh planning: first-ever 5-year OT refresh roadmap created using HAM EOL data

**Key HAM Features Used:**
- Custom asset classes for OT hardware
- Stockroom per plant with critical spare tracking
- Stock Rules with auto-PO generation
- EOL tracking for legacy hardware
- Contract management for OT vendor agreements
- Manual intake workflow (no Discovery for air-gapped networks)

---

## Story 5: University System — Shared Device Fleet Management

**Industry:** Higher Education  
**Company Size:** 45,000 students, 8,000 faculty and staff  
**Challenge:**
The university had 12,000 shared devices — computer lab PCs, library laptops, classroom AV equipment, research workstations — managed across 6 campuses. No central visibility. Devices were frequently "borrowed" from computer labs and not returned. Lab managers spent hours each week manually reconciling which devices were present. IT had no idea how many devices were approaching warranty expiry across all 6 campuses.

**What HAM Solved:**
- HAM with SCCM Service Graph Connector: all networked lab computers discovered and asset-record-linked within 45 days
- Barcode check-in/check-out (HAM Pro): students check out a laptop from the library by scanning their student ID and the laptop barcode → asset automatically assigned → due date recorded
- Overdue notifications: when a loaner laptop passes its return date, the borrower gets an automated email reminder; after 72 hours, an incident is created for the IT helpdesk
- Per-campus stockrooms: each campus library and IT department has its own stockroom — inter-campus transfers tracked
- EOL report by campus: IT leadership gets a monthly breakdown of aging devices per campus for capital planning
- AV equipment tracking: projectors, document cameras, and smartboards tracked with maintenance contracts — annual inspection tasks auto-created

**Results:**
- Lab device loss rate: reduced by 82% in the first academic year
- Loaner return compliance: from 71% returned within 1 week → 96% returned within 48 hours
- Warranty expiry surprises: eliminated — all devices tracked with 90-day advance warning
- Computer lab utilization reporting: IT now knows which labs are over-utilized (need more devices) vs. under-utilized (candidate for consolidation) — first-ever data-driven lab planning
- IT staff time on manual inventory: from 6 hours/week per campus → 30 minutes/week (exception management only)

**Key HAM Features Used:**
- SCCM Service Graph Connector
- Loaner lifecycle flow with overdue notifications
- Per-campus stockrooms with Transfer Orders
- Barcode check-in/check-out (HAM Pro mobile)
- EOL reporting by location
- Contract management for AV equipment

---

## Story 6: Dell Technologies — Multi-Tenant HAM (Life Cycle Hub)

**Industry:** Technology / Hardware Vendor  
**Company Size:** 150,000+ employees; Life Cycle Hub serves Dell's B2B customers  
**Context:** Real-world project — Dell's Life Cycle Hub (LCH) is a custom HAM scoped application built on ServiceNow to manage Dell hardware assets on behalf of Dell's enterprise customers.

**Challenge:**
Dell's enterprise customers needed a managed service for tracking their Dell hardware across its entire lifecycle — from purchase through warranty, deployment, maintenance, and end-of-life disposal. Each customer is a separate tenant with its own data, users, and processes. Dell needed to manage thousands of customers' assets in a single ServiceNow instance while maintaining strict data isolation between customers.

**Key Technical Implementations:**

**Multi-Tenancy Architecture:**
- Customer data isolated by company/account — Asset records tagged to customer company
- Separate stockrooms per customer account
- SCCM/Intune data from customer environments ingested via dedicated MID Servers per customer
- Business Rule: Company auto-populated from SCCM connection alias → company field on the asset record
  ```javascript
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
  ```

**Asset Classification Workflows:**
- **Hard Booking**: Reserved assets locked to a customer before physical receipt
- **Non-Conforming**: Assets that don't match the expected configuration (caught at receiving)
- **Data Discrepancy**: SCCM data doesn't match physically recorded specs — flagged for investigation
- **Asset Quarantine**: Assets with incomplete data held in quarantine state until resolved
- **Recovery/Recycle**: End-of-life assets routed to Dell's recycling program

**Service Life Calculation:**
- Custom logic: ServiceLife = (EOL Date − Purchase Date) in months
- Assets color-coded by service life remaining: >36 months (green) / 12-36 months (yellow) / <12 months (red)
- EOL date sourced from Dell's product database via API integration

**EPN Portal (External Partner Network):**
- Resellers and partners can view customer asset data through a scoped portal
- Data visibility controlled by partner relationship type
- Activity log: all partner portal actions logged against the asset record

**Integration Architecture:**
- Kafka (HIP — High-volume Integration Pipeline) selected for event streaming from Dell's manufacturing/warranty systems
- Staging tables for SCCM data → transform maps → IRE reconciliation → Asset + CI records
- Real-time warranty data push from Dell's systems to ServiceNow asset records

**Results for Dell's Customers:**
- Asset visibility from purchase to disposal: 100% (vs. customer-managed spreadsheets)
- Warranty claim success rate: significantly improved (warranty data current from Dell's systems)
- EOL planning: customers receive automated reports with 90/180-day refresh forecasts
- Non-conforming asset detection: caught at receiving vs. after deployment (cost savings)
- Multi-tenant scale: thousands of customers, millions of assets on single platform

---

## Story 7: Retail Chain — Mobile-First Asset Tracking

**Industry:** Retail  
**Company Size:** 800 stores, 35,000 devices (point-of-sale, handheld scanners, store laptops)  
**Challenge:**
800 retail stores meant 800 micro-IT-environments. Each store had point-of-sale systems, inventory scanners, store manager laptops, and shared tablets — totaling 35,000 devices managed by a central IT team of 40 people. Store managers were the de facto asset managers but had no system — they used physical binders. Device failures during peak shopping periods (Black Friday, holiday season) caused register downtime at $2,000/hour.

**What HAM Solved:**
- Intune Service Graph Connector: all 35,000 managed devices discovered and asset-record-linked
- Stockroom per region (not per store): 12 regional distribution centers maintain critical spare inventory
- Store manager self-service: store managers access the ServiceNow mobile app → scan a broken device → submit a replacement request → ServiceNow automatically identifies the nearest regional stockroom with a compatible spare → creates a transfer order and ships overnight
- Critical spare stock rules: POS terminals and handheld scanners have minimum stock rules at each regional hub — auto-PO generated when stock drops below threshold
- EOL tracking: all POS hardware EOL dates loaded — IT knows 3 years in advance which stores need hardware refresh
- Change freeze enforcement: hardware changes blocked during November 15 – January 5 (peak retail season) via ServiceNow change management integration

**Results:**
- Register downtime from hardware failures: from 14 incidents during holiday season → 2 (both caused by power issues, not hardware failure)
- Mean time to replace a failed device: from 3 days (manual process) → overnight (automated regional stockroom fulfillment)
- Store manager time on asset admin: from 2 hours/week → 10 minutes/week (exception only)
- Hardware over-ordering: $1.2M reduction in unnecessary spare hardware purchasing (stock rules replaced guesswork)
- Peak-season change freeze compliance: 100% (Change Management integration prevents unauthorized changes)

**Key HAM Features Used:**
- Intune Service Graph Connector
- Regional stockrooms with Transfer Orders
- Stock Rules with auto-PO generation
- ServiceNow Mobile (barcode scanning for device replacement requests)
- EOL tracking for multi-year refresh planning
- Change Management integration for freeze periods

---

## Common HAM Success Themes

| Theme | What Drives the Value |
|-------|----------------------|
| **Discovery automation** | Eliminates manual inventory — SCCM/Intune/Jamf connectors populate CMDB + Asset in one pass |
| **EOL tracking + alerts** | Converts reactive emergency purchasing into planned, budgeted refresh cycles |
| **Stock Rules + auto-PO** | Eliminates stockout situations and over-purchasing guesswork |
| **Lifecycle flows** | Multi-team coordination (Procurement, IT, Facilities, Finance) happens automatically |
| **Disposal documentation** | Creates audit-defensible chain of custody for HIPAA, GDPR, SOX compliance |
| **CMDB + Asset sync** | SecOps, ITSM, and ITAM all work from one accurate picture of every device |
| **Mobile scanning** | Makes stockroom operations real-time — physical and digital always in sync |
| **Contract management** | Warranty claims succeed; vendor SLAs are enforced; renewals are never missed |
