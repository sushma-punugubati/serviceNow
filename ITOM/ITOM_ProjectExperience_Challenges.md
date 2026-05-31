# ServiceNow ITOM — Interview Notes
## "What did you do in ITOM?" + Challenges Faced

---

## 1. OVERVIEW — What is ITOM?

ITOM (IT Operations Management) gives IT organizations automated, continuous visibility into their infrastructure. In my experience at Dell, ITOM is the foundation that makes everything else work correctly — ITSM incident context, Change impact analysis, SecOps CI prioritization, and SAM discovery data all depend on the CMDB being accurate, and ITOM Discovery is what keeps it accurate.

My ITOM work spans Discovery scheduling and MID Server management, CSDM-aligned CMDB population, Service Mapping for business service topology, Event Management integration with Splunk, and both on-premises and cloud-based discovery for AWS and Azure infrastructure.

---

## 2. WHAT I DID — Project Work (Scenario-Based)

### 2.1 Discovery and MID Server Operations

- Manage **Discovery Schedules** at Dell covering multiple data centers and network segments — each segment has its own Discovery Schedule aligned to the appropriate MID Server for that network zone
- Set up and maintain multiple **MID Servers** — configured worker thread counts, credential stores, and network access for each server. Monitor Work Queue depth and probe success rates after each Discovery run
- Configured **pattern-based discovery** for complex application stacks — moved away from legacy probe/sensor configurations for key applications to patterns that are easier to maintain and support credential-less authentication where possible
- Maintain **IRE identification rules** using stable identifiers: serial numbers for physical hardware, VMware UUIDs for virtual machines — eliminated duplicate CI creation from IP changes that was a chronic issue before my involvement
- Manage **reconciliation rules** defining authoritative data sources per CI attribute — Discovery owns IP/OS/RAM, SCCM owns installed software, Asset Management owns serial numbers and financial data

### 2.2 Event Management — Splunk Integration

- Integrated **Splunk SIEM alerts** into ServiceNow Event Management via REST — configured Splunk saved search alerts to POST to the ServiceNow Event Management endpoint on trigger
- Built **Alert Rules** for deduplication: same alert type from same source within a 1-hour window updates the existing Alert record rather than creating a new one — reduced alert volume from 15,000/day to ~200 actionable events
- Set up **CI Binding rules** with hostname normalization: try short hostname first, fall back to FQDN, fall back to IP address — binding success rate above 94%
- Configured **Alert Correlation** rules to group related infrastructure alerts under a single Alert Group — a network switch failure generating 50 downstream alerts creates one Alert Group, one incident
- Built **AIOps event-to-incident automation**: Alert Groups that exceed severity threshold and remain unacknowledged for 5 minutes automatically create ITSM Incidents with pre-populated CI context and business service impact

### 2.3 CSDM Framework Alignment

- Led the CMDB alignment to the **CSDM (Common Service Data Model)** framework at Dell — restructured CI classes and relationship types to match the CSDM layers: hardware, software, application services, technical services, business services
- Used **Service Graph Connectors** to replace raw import sets for SCCM data — proper CSDM-aligned relationship mapping replaced the stale, unmapped CI data we had accumulated
- Built **Business Service records** in CMDB mapping to the correct technical services and applications — this is what makes BSM Maps and Service Mapping topology maps useful for operations

### 2.4 Platform Upgrade and ITOM Regression Testing

- Handle **MID Server upgrades** as part of every platform upgrade — validate MID Server version compatibility, confirm worker thread configuration survived the upgrade, run a manual Discovery trigger post-upgrade to confirm probes are executing correctly
- Run **ITOM regression tests** post-upgrade: trigger Discovery against a known test range, verify CI counts are within expected bounds, check Event Management alert ingestion with a test event, confirm Service Mapping runs for a sample service

---

## 3. CHALLENGES FACED

### Challenge 1: MID Server Work Queue Backlog After Infrastructure Growth

**Problem:** Our nightly Discovery run was completing in 4 hours when I started. Over 18 months as the infrastructure grew, the same run started taking 14+ hours — it was still running when the next scheduled run tried to start, causing queued runs to pile up. CMDB data was 48+ hours behind reality.

**Solution:** Analyzed MID Server thread configuration and Work Queue metrics. The default 25 worker threads were completely saturated for our current scope. Increased worker threads to 75 per MID Server (after testing in non-production to confirm no network congestion issues). Split the single Discovery Schedule into four parallel schedules aligned to network segments, each on a dedicated MID Server. Discovery now completes in under 3 hours with room to grow.

---

### Challenge 2: Splunk Alert Storm on Initial Event Management Integration

**Problem:** When we first connected Splunk to Event Management, 14,000 events arrived in the first 24 hours. The SOC team was immediately overwhelmed, the Incident queue filled with auto-created incidents, and the Event Management console was completely unusable.

**Solution:** Worked with the Splunk team to implement severity-based filtering at source — only Splunk alerts at "High" or "Critical" forwarded to ServiceNow. Added deduplication rules in Event Management. Disabled automatic Incident creation until alert volume was at an acceptable baseline. Tuned over 3 weeks until daily actionable event volume was under 200, then re-enabled Incident creation. The lesson: never enable Event Management auto-incident-creation before tuning alert volume — it compounds the noise problem into an ITSM problem.

---

### Challenge 3: CI Binding Failures After Server Naming Convention Change

**Problem:** The infrastructure team changed server naming conventions — all servers were renamed from short names (PROD-APP-01) to FQDNs (prod-app-01.dell.internal). After the rename, CI binding success rate dropped from 94% to 61% overnight. Splunk was sending alerts with the new FQDNs but CMDB CI records still had short names.

**Solution:** Updated CI Binding rules to try FQDN matching first (against the CMDB FQDN field), then strip the domain and try short name matching. Simultaneously ran a Discovery cycle to update all CI name fields to the new naming convention — this took 2 days to complete across all CIs. After both changes were in place, CI binding success rate returned to 96%. Added a monitoring check: CI binding success rate drops below 85% → automated alert to the ITOM team.

---

### Challenge 4: CSDM Migration Causing Broken Relationships in Existing Reports

**Problem:** When we started migrating CI relationship types to CSDM-aligned types (replacing custom relationship types like "Deployed On" with the CSDM-standard "Runs on"), several existing reports and dashboards that filtered on the old relationship type names broke. Operations managers who used these dashboards daily raised urgent escalations.

**Solution:** Before migrating any relationship type in production, ran a report of all reports, PA indicators, and dashboards that referenced the relationship type being changed. Updated each one to use the new CSDM relationship type name. Only migrated the relationship type in CMDB after all dependent reports were updated. Created a migration runbook documenting each relationship type change, the reports affected, and the update made — used this as a template for the remaining 8 relationship types in the migration plan.

---

## 4. KEY CONCEPTS TO KNOW (Quick Reference)

| Concept | What to Say |
|---|---|
| **MID Server** | On-premises Java agent bridging ServiceNow cloud to internal infrastructure |
| **ECC Queue** | Communication channel between ServiceNow and MID Server — probe instructions out, results in |
| **IRE** | Identification and Reconciliation Engine — prevents duplicate CIs, manages multi-source data |
| **Probe / Sensor** | Probe = data collection. Sensor = parsing and CMDB write. Patterns combine both |
| **CI Binding** | Matching an alert to its CMDB CI — provides business context for event-driven incidents |
| **Alert Correlation** | Grouping related alerts into one Alert Group to prevent alert-per-incident storms |
| **CSDM** | Common Service Data Model — ServiceNow's framework for consistent CMDB structure |
| **BSM Map** | Visual dependency map of a Business Service showing all supporting CIs |
| **Discovery Pattern** | Reusable multi-step discovery logic for complex applications — modern replacement for probe/sensor |

---

## 5. METRICS / OUTCOMES TO QUOTE

- Discovery Work Queue backlog eliminated — nightly run time reduced from **14+ hours to under 3 hours** after MID Server tuning
- Splunk alert volume reduced from **14,000/day to ~200 actionable events** after Event Management tuning
- CI Binding success rate maintained above **94%** after dual hostname/IP matching rules
- CMDB CI staleness eliminated for infrastructure scope — CIs updated **within 24 hours** of any infrastructure change
- CSDM alignment improved Service Mapping accuracy: service topology maps went from **~45% to 82% accuracy**
- Platform upgrade ITOM regression: zero post-upgrade Discovery or Event Management failures over last 3 upgrades
