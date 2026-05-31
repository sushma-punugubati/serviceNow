# ServiceNow ITAM — Interview Notes
## "What did you do in ITAM (HAM + SAM)?" + Challenges Faced

---

## 1. OVERVIEW — What is ITAM?

ITAM (IT Asset Management) in ServiceNow covers the full lifecycle of both hardware and software assets. It consists of two distinct but related modules:

- **HAM Pro (Hardware Asset Management)**: Tracks physical assets from procurement through deployment, transfer, and disposal. The CMDB stores the CI; the Asset module tracks the financial and lifecycle record. HAM bridges the two — when a laptop is purchased, received, deployed to an employee, and eventually decommissioned, every step is tracked and auditable.

- **SAM Pro (Software Asset Management)**: Tracks software licenses, installations, and compliance. The goal: always know how many licenses you own, how many are installed, and whether you are compliant — without waiting for an audit to find out.

Together, HAM and SAM answer the question: "What do we have, where is it, who has it, and are we compliant?"

Key areas worked on:
- Full hardware asset lifecycle (procurement to disposal)
- ASN integration and automated asset record creation
- Stockroom and transfer order management
- Disposal workflows with financial reconciliation
- SCCM/Intune integration via MID Server
- Software models and normalization
- License entitlement records and reconciliation schedules
- Publisher pack configuration
- Reclamation workflows
- SaaS License Management

---

## 2. WHAT I DID — Project Work (Scenario-Based)

### 2.1 Hardware Asset Lifecycle (HAM Pro)

- Implemented the **full hardware asset lifecycle** from procurement through disposal at Dell — starting at PO receipt, assets were tracked through receiving, stock, deployment, transfer, and end-of-life
- Built the **ASN (Advance Shipment Notice) integration** — when a Purchase Order was approved and the vendor submitted an ASN, ServiceNow automatically created Asset records in Pre-allocated status and updated them to In Stock when physical receipt was confirmed. This eliminated the manual step of creating asset records after hardware arrived
- Configured **Stockrooms** for each corporate location with proper asset flow: received assets went to the regional stockroom, from which they could be deployed to employees via deployment requests or transferred to other stockrooms
- Built **Transfer Order workflows** for inter-site asset moves — assets transferred between locations followed a formal checkout/transit/receipt process with full chain of custody tracked in ServiceNow

### 2.2 Asset-CMDB Synchronization

- Configured the **CMDB-Asset sync**: when an asset moved to Deployed status, a corresponding CI record was created or updated in the CMDB with the assigned user, location, and department. When an asset was retired, the CI was moved to Retired status
- Integrated with **SCCM and Intune via MID Server** — hardware discovery data from both tools populated the CMDB CI records, which were then reconciled with the HAM asset records using asset tag and serial number as matching keys
- Set up **reconciliation alerts**: when SCCM discovered a device that had no matching asset record (indicating an untracked device on the network), an automated task was created for the asset team to investigate

### 2.3 Disposal Workflow and Financial Reconciliation

- Built **disposal workflows** that handled the end-of-life process: request for disposal → IT data wipe confirmation → physical pickup scheduling → vendor receipt confirmation → asset retirement in ServiceNow
- Configured **financial reconciliation** on asset retirement: when an asset was disposed, the asset record was updated with the disposal date and residual book value, and a notification was sent to the Finance team to update the asset register in the ERP system
- Ensured **data privacy compliance**: the disposal workflow required a completed data wipe certificate before the asset could move to Disposed status — the certificate was attached to the asset record as the audit trail

### 2.4 Software Asset Management (SAM Pro)

- Set up **Software Models** for all enterprise software in the catalog — each model defined the software publisher, product name, version, and license metric (per user, per device, per processor, per seat)
- Configured **normalization rules** to map the many variations of software names from SCCM discovery (e.g., "Microsoft Office 365 ProPlus," "Microsoft 365 Apps for enterprise," "M365 Apps") to a single canonical Software Model — critical for accurate reconciliation
- Built **License Entitlement records** for each software agreement: mapped to the correct Software Model, defined the quantity purchased, license metric, and agreement end date
- Configured **Reconciliation Schedules** to run automatically weekly — compared installed counts from discovery against entitlement quantities and flagged over-licensed and under-licensed positions

### 2.5 Publisher Packs and Reclamation

- Got into **Publisher Pack configuration** for Microsoft, Adobe, IBM, and Oracle — publisher packs provided pre-built software models and license metrics from ServiceNow's Content Library, significantly reducing the normalization effort for major vendors
- Set up **Reclamation Workflows** to flag software installed on machines with no usage in the last 90 days — the workflow notified the assigned user and their manager and, if no response after 7 days, submitted an automated uninstall request to the software deployment team
- Configured **SaaS License Management** for cloud applications (Salesforce, Zoom, Slack, Tableau) — pulled usage data via API integrations to identify seats that had not logged in for 60+ days, which fed the reclamation workflow for SaaS licenses

---

## 3. CHALLENGES FACED

### Challenge 1: ASN Integration — Asset Records Created But Not Matched to POs

**Problem:** After deploying the ASN integration, asset records were being created on ASN receipt but 20% of them were not being linked to their Purchase Order. These "orphaned" assets had no financial record, causing discrepancies between the asset register and the finance ERP.

**Solution:** Investigated the matching logic and found that the PO-to-ASN matching was using the PO Number field, but vendor ASN files often padded PO numbers with leading zeros that did not match ServiceNow's format. Fixed the transform map to strip leading zeros from the PO Number before matching. Also added a fallback match on the vendor part number + quantity combination for cases where PO numbers were missing from the ASN file entirely.

---

### Challenge 2: Software Normalization — 400+ Variants for Common Products

**Problem:** Running SAM Pro for the first time against SCCM data revealed over 400 distinct software names for what was actually about 80 unique products. Microsoft Office alone had 47 distinct strings depending on version, edition, and install source. Without normalization, reconciliation was impossible.

**Solution:** Used the ServiceNow **Content Library** (publisher packs) to download pre-normalized models for the top 20 publishers by installation count — this handled about 60% of installations automatically. For remaining software, ran a normalization workshop with the software asset team to manually map the top 50 remaining products by volume. Created custom normalization rules for internally developed software that had no publisher pack equivalent.

---

### Challenge 3: Reclamation Triggering on Actively Used Software

**Problem:** The reclamation workflow was triggering for software that appeared unused based on SCCM data but was actually being used via Citrix virtual desktop sessions — SCCM only saw the local installation, not the virtual session usage.

**Solution:** Added a secondary usage check for any software flagged for reclamation: before sending the reclamation notification, a script checked the Citrix usage log integration to confirm no virtual session usage in the same 90-day window. Only software with no local AND no virtual usage proceeded through reclamation. Also added a self-service override: employees could mark software as "actively needed" through the Employee Center, which suppressed reclamation for 6 months.

---

### Challenge 4: License Count Discrepancies Between SAM and Finance

**Problem:** SAM Pro showed 1,200 Microsoft Office licenses purchased, but Finance had 1,450 in their system. The discrepancy raised compliance concerns and eroded trust in SAM data.

**Solution:** Conducted a reconciliation exercise comparing every entitlement record in SAM Pro against every line item in the Microsoft agreement held by Finance. Found two sources of discrepancy: 200 licenses on expired agreements that Finance still counted but SAM Pro had marked as expired, and 50 licenses allocated to a department that had been divested and removed from SAM Pro. Updated the reconciliation to include expired agreements with a separate "expired" flag rather than excluding them entirely, so Finance and SAM showed the same gross number with an explanation for the difference.

---

### Challenge 5: Disposal Records Not Reflecting Physical Reality

**Problem:** Assets marked as Disposed in ServiceNow were sometimes still physically present in offices — either the disposal vendor never collected them or the disposal request was closed prematurely. CMDB was showing these assets as Retired while they were actually still in use or at risk.

**Solution:** Added a two-step disposal verification: the asset moved to "Awaiting Collection" state on disposal request approval, and only moved to "Disposed" when the disposal vendor submitted a signed pickup receipt (attached as a document to the asset record). Added a monthly audit report comparing "Awaiting Collection" assets older than 30 days to identify stalled disposals.

---

## 4. KEY CONCEPTS TO KNOW (Quick Reference)

| Concept | What to Say |
|---|---|
| **HAM Pro** | Hardware Asset Management — tracks physical assets from procurement to disposal with full lifecycle and financial records |
| **SAM Pro** | Software Asset Management — tracks software licenses, installations, and compliance position |
| **Asset vs. CI** | Asset = financial/lifecycle record (cost, ownership, depreciation). CI = technical/operational record (relationships, configuration, status) |
| **ASN** | Advance Shipment Notice — vendor notification of incoming shipment, used to pre-create asset records before physical receipt |
| **Software Model** | The canonical definition of a software product in SAM — publisher, name, version, license metric |
| **License Entitlement** | The record of licenses purchased — quantity, metric, agreement terms, expiration date |
| **Software Reconciliation** | The process of comparing installed license count vs. purchased entitlement — identifies over/under-licensed positions |
| **Publisher Pack** | Pre-built set of software models from ServiceNow's Content Library for major publishers |
| **Reclamation** | The process of identifying unused software licenses and recovering them for reassignment |
| **SaaS License Management** | Tracking cloud application subscription usage to identify unused seats |

---

## 5. METRICS / OUTCOMES TO QUOTE

- ASN integration reduced asset record creation time from **2-3 days manual entry to same-day automated** on PO receipt
- Software normalization: reduced 400+ software name variants to **~80 canonical models** — made reconciliation possible
- Reclamation program: **~8% of software licenses recovered** in first year — meaningful cost savings at enterprise scale
- SaaS unused seat identification: flagged **15-20% of SaaS subscriptions** as candidates for reclamation or downgrade
- Disposal compliance: **100% of disposals** required signed vendor pickup receipt before closure — eliminated "ghost assets"
- License compliance position: shifted from reactive (finding out at audit time) to **proactive weekly reconciliation** with automated alerts
