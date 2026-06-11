# ServiceNow CSDM — Interview Questions & Answers

---

## PART 1: Core Concepts

**Q1: What is CSDM and why does ServiceNow require it?**
**A:** CSDM (Common Service Data Model) is ServiceNow's prescriptive framework for organizing CMDB data consistently. It defines standard CI classes, relationship types, and data model layers — so all platform modules (ITSM, ITOM, SecOps, IRM) read from the same structured model.

ServiceNow does not technically *require* CSDM, but many platform features only work correctly when CMDB data follows CSDM standards:
- Service Mapping needs standard relationship types to traverse the topology
- Event Management needs the Application Service layer to show business impact
- Change Management needs CSDM relationships to calculate accurate impact
- AI-driven features like Now Assist summarization and probable cause use CSDM context

**The business case:** Organizations that implement CSDM get reliable impact analysis, accurate change risk, and correct event correlation. Those that skip it get an expensive CMDB that does not actually support the outcomes they paid for.

---

**Q2: What are the four main CSDM layers?**
**A:**
1. **Business Layer:** Business Capabilities — what the business does (not an IT concept)
2. **Application Portfolio:** Business Applications — the application as a software asset (what you purchased or built)
3. **Service Portfolio / Application Service:** Running deployments of Business Applications in specific environments — the bridge between business and technical layers
4. **Technical / Infrastructure:** Technical Services, servers, VMs, databases, network — the physical and virtual infrastructure that runs everything

The Application Service is the most critical layer — it is where the business layer meets the technical layer and where most ITOM, ITSM, and monitoring features anchor their data.

---

**Q3: What is the difference between a Business Application and an Application Service?**
**A:**
| Business Application | Application Service |
|---------------------|---------------------|
| The application as a product/asset | A specific deployment/instance of that application |
| One record per application | Multiple records — one per environment (PROD, UAT, DEV) |
| "Oracle ERP" | "Oracle ERP - Production" and "Oracle ERP - UAT" |
| Owned by business stakeholder | Managed by IT operations |
| Table: `cmdb_ci_business_app` | Table: `cmdb_ci_service` |
| Lifecycle: concept to retirement | Lifecycle: deployed to decommissioned |

**Interview tip:** Always clarify this distinction when talking about CSDM — "Business Application" and "Application Service" sound similar but serve very different purposes. The Business Application is the product. The Application Service is the running instance.

---

**Q4: What is the CSDM maturity model?**
**A:** CSDM adoption is measured in five maturity levels:

| Level | Focus | Key Milestone |
|-------|-------|---------------|
| **Foundation** | Infrastructure CIs | Servers, network devices in CMDB — no service layer |
| **Crawl** | Service relationships | Application Services linked to infrastructure CIs |
| **Walk** | Application Portfolio | Business Applications mapped to Application Services |
| **Run** | Service Portfolio | Service Offerings defined, full CSDM model operational |
| **Fly** | Business Capabilities | CSDM drives cost allocation, business decisions, AI |

Most organizations operate at Foundation/Crawl and target Walk/Run. Fly requires significant engagement from business stakeholders beyond IT and is typically a multi-year journey.

**Practical approach:** Do not try to implement all CSDM layers at once. Start with the services that matter most (top 20 critical business services), get them to Walk/Run, prove the value, then expand.

---

**Q5: What are Service Graph Connectors and how do they differ from raw import sets?**
**A:** Service Graph Connectors are pre-built integrations that populate the CMDB following CSDM standards.

**Raw Import Set:**
- Loads data into a staging table, transforms to CMDB
- Creates CI records — but usually without relationships
- CI classes may not align to CSDM
- No pre-configured IRE identification rules — duplicates are common
- Relationship types: none or custom

**Service Graph Connector:**
- Pre-built for specific data sources (SCCM, vSphere, AWS, Azure)
- Creates CIs AND standard CSDM relationships in one operation
- CSDM-aligned CI classes out of the box
- Pre-configured IRE identification and reconciliation rules
- Standard CSDM relationship types (Runs on, Hosted on, etc.)

**When to use which:** Service Graph Connectors are the modern standard for any supported data source. Raw import sets are used for custom data sources where no connector exists.

---

**Q6: What is a Service Offering and where does it fit in CSDM?**
**A:** A Service Offering defines how a service is delivered to its consumers — the terms, commitments, pricing, and conditions under which a service is available.

**Structure:**
```
Service Portfolio → Service Offering → Application Service
```

**Example:**
- Service Portfolio: "IT Infrastructure Services"
- Service Offering: "Virtual Server - Standard" (2 vCPU, 8GB RAM, 99.9% SLA, $200/month)
- Application Service: The specific VM instances delivered under this offering

**Why it matters:** Service Offerings bridge the gap between the business's view of IT (what they can order and what they pay) and IT's view (what is running and how). They are the foundation for Service Catalog items, SLA definitions, and chargeback/showback reporting.

---

**Q7: What is the CMDB Health Score and what does it measure?**
**A:** The CMDB Health Dashboard provides a quantitative score across four dimensions:

| Dimension | What it checks |
|-----------|----------------|
| **Completeness** | Are required attributes populated on CI records? (e.g., does every server have an IP address, OS, and assignment group?) |
| **Correctness** | Are attribute values valid? (no null values in mandatory fields, correct formats, valid reference values) |
| **Compliance** | Do CIs use CSDM-standard CI classes and relationship types? |
| **Orphaned CIs** | Are there CI records with no relationships — isolated records that provide no service context? |

**Target scores:** 85%+ is the ServiceNow benchmark for "good" data quality. Below 60% means too many CMDB-dependent features are unreliable.

**Key insight:** A high health score in itself is not the goal — the goal is that the features that depend on CMDB (Service Mapping, impact analysis, event correlation) work correctly. CSDM health score is a leading indicator for that.

---

**Q8: How does CSDM relate to Service Mapping?**
**A:** CSDM and Service Mapping are complementary — Service Mapping *populates* the CSDM layers, and CSDM *validates* that Service Mapping's output is structured correctly.

**Service Mapping's role in CSDM:**
- Discovers Application Services and their dependencies automatically
- Creates CI records for application components at the Application Service and Technical Service layers
- Creates CI relationships using CSDM-standard relationship types (if configured correctly)

**CSDM's role in Service Mapping:**
- Defines what the output should look like (correct CI classes, correct relationship types)
- Provides the Business Application context that Application Services link to
- Ensures the map is navigable by Event Management and Change Impact Analysis

**Common issue:** Service Mapping configured before CSDM alignment creates relationships using non-standard types — the maps look right visually but Event Management cannot traverse them. Fix: update pattern relationship step configurations to use CSDM relationship types.

---

**Q9: What is the impact of CSDM alignment on ITSM?**
**A:**
1. **Incident Management:** CI field on incidents links to CSDM-structured Application Service → agent immediately sees which business service is affected and who the business owner is

2. **Change Management:** Impact calculation traverses CSDM relationships — a change to a server shows all Application Services that depend on it. Without CSDM relationships, impact analysis returns empty or wrong results

3. **Problem Management:** Root cause analysis traverses CSDM dependency chain — helps identify whether a pattern of incidents traces to a shared infrastructure component

4. **Service Catalog:** Catalog items linked to Service Offerings → published to the correct audience based on Service Portfolio context

5. **SLA:** SLA definitions can be linked to Service Offerings rather than individual groups — consistent SLA commitments across all ways of consuming a service

---

**Q10: What are orphaned CIs and why are they a problem?**
**A:** An **orphaned CI** is a CI record with no relationships to any other CI — it exists in isolation in the CMDB, connected to nothing.

**Why they are a problem:**
- Impact analysis cannot include orphaned CIs — no relationships means no path to traverse
- Service Mapping cannot discover them as part of a topology (nothing points to them)
- Event Management alerts bound to orphaned CIs cannot be correlated to a business service
- They inflate CI counts without providing any operational value

**Common sources of orphaned CIs:**
- Import sets that created CIs without relationships
- Manual CI creation without relationship setup
- Decommissioned CI relationships deleted but CI record left behind
- Discovery creating CIs for isolated devices with no application context

**Fix:** Run the CMDB Orphaned CI report regularly. For each orphaned CI: investigate whether it should be related to an Application Service or Technical Service, or whether it should be retired if it no longer exists.

---

## PART 2: Implementation

**Q11: You are starting a CSDM implementation for an organization at Foundation maturity. What is your approach?**
**A:**
1. **Assess current state:** Run the CMDB Health Dashboard to understand current completeness, compliance, and orphan rates. Identify the top 20 most critical business services from the ITSM team.

2. **Define the Application Service layer first:** Create Application Service records for the top 20 critical services. This is the highest-value layer — it connects all the ITOM features (Service Mapping, Event Management) to business context.

3. **Link to Business Applications:** Create or update Business Application records for each Application Service. Link Application Services to Business Applications.

4. **Deploy Service Graph Connectors:** Replace any raw import sets for major data sources (SCCM, vSphere) with Service Graph Connectors — this ensures new CI data arrives CSDM-aligned.

5. **Run Service Mapping:** For each Application Service, run Service Mapping to automatically discover and create Technical Service and Infrastructure relationships.

6. **Measure and expand:** Use the CMDB Health Score to track progress. Once the top 20 services are at Walk/Run maturity, expand to the next 50.

---

**Q12: How do you handle the business stakeholder engagement required for CSDM?**
**A:** CSDM beyond the infrastructure layer requires business input — Business Applications, Business Capabilities, and Service Offerings cannot be defined by IT alone.

**Engagement approach:**
1. **Identify Application Portfolio owners:** Each Business Application needs a business owner (who owns the product decision) and an IT owner (who manages the deployment). Establish this ownership before creating Business Application records.

2. **Service Offering workshops:** For each major service category, run a workshop with the business to define what they actually care about — SLAs, pricing, tier options. This becomes the Service Offering structure.

3. **Executive sponsorship:** CSDM often requires someone above the IT director level to mandate that business units provide application portfolio data. Without sponsorship, business units deprioritize CSDM contributions.

4. **Show the value early:** Demonstrate impact analysis working correctly for one well-mapped service before asking business stakeholders to contribute data for all services. Seeing the change risk assessment and incident impact visualization working drives engagement.

---

**Q13: How do Service Graph Connectors handle updates when the source data changes?**
**A:** Service Graph Connectors run on a schedule and handle updates through the IRE:

1. **Identification:** When new data arrives, IRE checks whether a CI matching the identification rule (serial number, hostname, VM UUID) already exists
2. **If CI exists:** IRE applies reconciliation rules to update attributes — only the data source with the highest precedence for a given attribute can overwrite it
3. **Relationships:** Connector compares existing relationships against the latest data — adds new relationships, marks stale ones for review
4. **Reconciliation precedence:** Service Graph Connector data typically has lower precedence than Discovery for attributes like IP address (Discovery is more current) but higher than manual entry for inventory data

**Key advantage:** Unlike raw import sets that often create duplicate records or overwrite manually-set fields, Service Graph Connectors respect the reconciliation rules and preserve data quality across multiple sources.

---

## PART 3: Scenarios and Advanced

**Q14: The CMDB health score shows 42% compliance. What is your remediation plan?**
**A:**
1. **Segment the problem:** Break the 42% down by CI class — which classes are dragging the score? Servers? Applications? Network devices? Target the highest-volume, lowest-compliance classes first.

2. **Identify root cause per class:** For each low-compliance class:
   - Missing mandatory attributes → fix the data source (Discovery, import set) to populate required fields
   - Wrong CI class → reclassify CIs to CSDM-aligned classes
   - Missing relationships → determine source (should Service Mapping, Service Graph Connector, or manual process create these?)
   - Orphaned CIs → investigate and either relate or retire

3. **Assign ownership:** Create a CMDB governance program where each CI class has an owning team responsible for their class's health score. Monthly review against targets.

4. **Automate remediation:** For missing attributes that can be auto-populated (e.g., IP from Discovery, OS version from SCCM), fix the data source rather than manually patching records.

5. **Track progress:** Set quarterly targets: Q1 → 55%, Q2 → 65%, Q3 → 75%. Report progress to IT leadership.

---

**Q15: How does CSDM differ between ServiceNow versions?**
**A:** CSDM has evolved significantly across ServiceNow releases:

| Version | CSDM Version | Key Changes |
|---------|-------------|-------------|
| Madrid - New York | CSDM 1.x | Basic service hierarchy introduced |
| Paris - Rome | CSDM 2.0 | Application Service and Business Application formalized |
| San Diego - Tokyo | CSDM 3.0 | Service Offering layer added, improved Service Graph Connector support |
| Utah - Xanadu | CSDM 4.0 | AI integration points, improved Fly maturity guidance, enhanced health scoring |

**Practical impact:** Organizations upgrading from older releases may have CMDB data that uses CSDM 1.x or 2.0 patterns — specific CI classes or relationship types that were renamed or deprecated. Platform upgrades require a CSDM compatibility review to ensure existing data still aligns with the current version's standards.
