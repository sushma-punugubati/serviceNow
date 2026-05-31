# ServiceNow CSDM — Interview Notes
## "What did you do in CSDM?" + Challenges Faced

---

## 1. OVERVIEW — What is CSDM?

CSDM (Common Service Data Model) is ServiceNow's prescriptive framework for structuring CMDB data. It defines the standard entities, relationships, and data model layers that all ServiceNow modules use. Without CSDM, Discovery fills the CMDB with individual CI records that have no service context — useful for inventory but not for impact analysis, change risk, or event correlation.

My CSDM work at Dell was a significant multi-month project to align our existing CMDB to the CSDM framework. Before the alignment, we had a CMDB full of server and application CIs with no consistent relationships — Service Mapping worked but Event Management could not use the topology to determine business service impact. After the alignment, all ITSM, ITOM, and SecOps features started working together correctly.

Key work I did:
- CSDM gap analysis: mapping existing CI classes to CSDM framework
- Application Service record creation for critical business services
- Service Graph Connector deployment for SCCM (replacing raw import set)
- Relationship type standardization across 20,000+ CI relationship records
- CMDB Health Score tracking and improvement program
- CSDM governance: preventing non-compliant CI creation

---

## 2. WHAT I DID — Project Work (Scenario-Based)

### 2.1 CSDM Gap Analysis

- Ran the CMDB Health Dashboard and found our initial compliance score was **38%** — primarily due to non-standard CI classes, missing Application Service layer, and thousands of orphaned CIs
- Did a full **CSDM gap analysis**: mapped every CI class in use to the CSDM standard classes. Found 9 custom CI classes that needed consolidation and 3 that had no CSDM equivalent (required a governance decision)
- Created a migration plan: prioritized by CI count × impact — the classes with the most records and highest operational importance were migrated first

### 2.2 Application Service Layer Creation

- The most impactful CSDM work: created **Application Service records** for Dell's 60 most critical IT services. Before this, Discovery had populated thousands of server CIs but no service layer existed — ITSM agents had no way to link incidents to business services
- Each Application Service record was linked to: its parent Business Application, the infrastructure CIs that support it (via Service Mapping), the assignment group responsible for it, and the SLA definition that governs it
- This one change transformed Event Management: alerts bound to infrastructure CIs could now traverse the CSDM relationship chain up to the Application Service, automatically showing business impact on every alert

### 2.3 Service Graph Connector — SCCM

- Replaced the raw SCCM import set (which had been running for 3+ years without relationship mapping) with the **SCCM Service Graph Connector**
- The connector change had immediate CSDM impact: SCCM-sourced server CIs were now created with CSDM-aligned classes and standard relationship types connecting them to their installed software, domain, and organizational unit
- Handled the transition carefully: ran the connector in parallel with the import set for 2 weeks to confirm record matching rates before decommissioning the import set

### 2.4 Relationship Type Standardization

- The hardest technical work: existing CI relationships used a mix of custom type names accumulated over years. Service Mapping had created "Deployed On" relationships; manual entries had "Part Of" and "Supports"; old legacy data had blank relationship types
- Wrote a script to audit all `cmdb_rel_ci` records and identify non-CSDM relationship types — found **14 distinct non-standard types** across approximately 18,000 relationship records
- Migrated each non-standard type to the closest CSDM equivalent: "Deployed On" → "Runs on", "Part Of" → "Member of", etc. Updated the corresponding Service Mapping pattern configurations so future runs created the correct types
- Validated each migration by checking that the BSM Maps for affected services still rendered correctly after the relationship type change

### 2.5 CMDB Health Score Governance

- Set up a **CMDB governance program** with health score targets per CI class owner: each team responsible for their class's completeness and compliance score, reviewed monthly
- Built PA indicators tracking CSDM compliance by CI class — managers could see their team's score trending up or down over time
- Added Business Rules to prevent the creation of non-compliant CI records: a server CI created without an assignment group is caught at creation time, not discovered months later in a health audit

---

## 3. CHALLENGES FACED

### Challenge 1: Resistance to Reclassification — Reports Breaking During Migration

**Problem:** When we started reclassifying application CIs from `cmdb_ci_appl` to the correct CSDM classes (`cmdb_ci_business_app` and `cmdb_ci_service`), several Performance Analytics dashboards broke immediately. The operations managers who used those dashboards daily raised urgent escalations — they saw the migration as IT breaking working tools.

**Solution:** Did not have sufficient pre-migration dependency analysis. From that point, built a migration checklist:
1. Run a report of all PA indicators, reports, and Business Rules referencing the source CI class
2. Update each consumer to the new class first
3. Run in parallel (old + new class) for one sprint
4. Only complete reclassification after all consumers are confirmed working on the new class

Added a CSDM migration test environment: all CI reclassifications tested in DEV first with the full PA and reporting suite validated before promoting to production.

---

### Challenge 2: No Business Stakeholder Engagement for Application Portfolio

**Problem:** Creating Business Application records required business stakeholders to identify their application portfolio — who owned each app, what business capability it enabled, what its lifecycle state was. Most stakeholders were unresponsive. Six weeks into the project, we had Business Application records for only 12 of the 60 target applications because we could not get stakeholders to respond.

**Solution:** Escalated to the IT Director to get executive sponsorship. Got the CTO to send a mandate: all application owners must complete the CSDM intake form by a specific date, or their application would be flagged as "unmanaged" in the Application Portfolio (which had contractual and audit implications). Compliance went from 20% to 85% within 2 weeks of the mandate. Lesson: CSDM beyond the infrastructure layer requires executive sponsorship — it is not an IT project, it is a business governance project.

---

### Challenge 3: Service Graph Connector Creating Duplicate CIs

**Problem:** When we deployed the SCCM Service Graph Connector alongside the existing Discovery setup, the connector created duplicate CI records for 800+ Windows servers. Discovery had created CIs using hostname as the identifier; the SCCM connector used a different matching key (hardware GUID from SCCM) and could not find the existing Discovery CIs.

**Solution:** Added a cross-source identification rule to the IRE configuration: for `cmdb_ci_server` class, try hostname + domain first, then try hardware GUID as fallback. This allowed the connector to find Discovery-created CIs using hostname and merge its SCCM-sourced attributes into the existing record rather than creating a new one. Ran CMDB Deduplication to merge the 800 existing duplicates. Going forward, both sources now update the same CI record with the data each is authoritative for.

---

### Challenge 4: Non-Standard Relationship Types Breaking Event Management After CSDM Alignment

**Problem:** After we completed the Application Service layer and linked infrastructure CIs to Application Services via CSDM-standard "Runs on" relationships, Event Management still could not show business service impact. Alerts were binding to CIs correctly but the business service field on the alert was blank.

**Solution:** Discovered that our Event Management alert correlation rules were configured to traverse only the old custom relationship types ("Deployed On") that we had just migrated away from. The rules had not been updated to use CSDM relationship types. Updated the Event Management correlation configuration to traverse "Runs on" and "Depends on" relationships (CSDM standard). After the update, business service impact appeared correctly on bound alerts within minutes. This reinforced the lesson: when changing relationship types in CMDB, every feature that traverses relationships must be reviewed and updated simultaneously.

---

## 4. KEY CONCEPTS TO KNOW (Quick Reference)

| Concept | What to Say |
|---|---|
| **CSDM** | ServiceNow's prescriptive framework for structuring CMDB data — defines CI classes, layers, and relationship types |
| **Application Service** | Running deployment of a Business Application — the bridge between business and technical CMDB layers |
| **Business Application** | The application as a product/asset — owned by business stakeholders |
| **Technical Service** | Shared platform layer supporting multiple Application Services |
| **Service Offering** | Terms and conditions under which a service is delivered — SLA, price, tier |
| **Service Graph Connector** | Pre-built integration that populates CMDB with CSDM-aligned CIs and relationships |
| **CSDM Maturity** | Foundation → Crawl → Walk → Run → Fly — five levels of CSDM adoption |
| **CMDB Health Score** | Completeness, Correctness, Compliance, Orphan rate — measures CSDM data quality |
| **Relationship Type** | Standard CSDM names: Runs on, Depends on, Hosted on, Managed by — replaces custom types |

---

## 5. METRICS / OUTCOMES TO QUOTE

- CMDB Health Score improved from **38% to 74%** over 6 months after CSDM alignment project
- Application Service layer created for **60 critical services** — Event Management business impact went from blank to accurate on all 60 services
- SCCM Service Graph Connector replaced raw import set — **800 duplicate CIs eliminated** and ongoing CSDM compliance for all SCCM-sourced data
- Relationship type standardization: **18,000+ relationship records** migrated to CSDM-standard types
- Event Management business service impact accuracy: improved from **0% to 94%** for the 60 CSDM-aligned services
- Change Management impact analysis: went from returning blank results to correctly identifying affected Application Services for **~85% of production changes**
