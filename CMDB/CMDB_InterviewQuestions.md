# ServiceNow CMDB — Interview Questions & Answers

---

## PART 1: Core Concepts

**Q1: What is the CMDB and why is it important?**
**A:** The CMDB (Configuration Management Database) stores information about Configuration Items (CIs) and their relationships. It's the backbone of the ServiceNow platform — almost every module (ITSM, ITOM, SecOps, IRM, CSM) depends on accurate CMDB data.

**Why important:**
- Impact analysis: understand what breaks when one CI fails
- Change planning: assess risk before making changes
- Incident resolution: identify root cause faster with dependency maps
- SecOps: prioritize vulnerabilities based on business service criticality

---

**Q2: What is a Configuration Item (CI)?**
**A:** A CI is any component that needs to be managed to deliver IT services successfully. Examples: servers, applications, databases, network devices, virtual machines, cloud resources.

CIs are stored in the `cmdb_ci` table and its child tables. Each CI has attributes (name, class, status, owner) and relationships to other CIs.

---

**Q3: What are the three key CMDB tables?**
**A:**
1. `cmdb` — Base/abstract table (foundation layer)
2. `cmdb_ci` — Configuration Item — stores all CI records; all specific CI types extend this
3. `cmdb_rel_ci` — CI Relationship — stores relationships between CIs (depends on, runs on, etc.)

---

**Q4: What is the difference between a CI and an Asset?**
**A:**
- **CI:** Logical/physical infrastructure component. Focus: relationships, dependencies, status, configuration. Supports incident/change/problem management.
- **Asset:** Financial/inventory record. Focus: cost, ownership, depreciation, lifecycle. Supports procurement and financial tracking.

They are linked — a server has both a CI record AND an Asset record — but serve different purposes.

---

**Q5: What are CI Relationships and how do they support impact analysis?**
**A:** CI Relationships (stored in `cmdb_rel_ci`) describe how CIs are connected:
- Depends on / Used by
- Runs on / Hosts
- Virtualized by / Virtualizes
- Connected to / Contains

**Impact analysis:** When CI X fails, the relationship map immediately shows every CI that depends on X — and which business services are affected. Without relationships, you know a server failed but not what broke for the business.

---

**Q6: What is ServiceNow Discovery?**
**A:** Discovery is ServiceNow's tool for automatically finding and documenting CIs in the network. It:
1. Scans IP ranges for devices
2. Sends probes to collect raw data
3. Sensors process data into CI records
4. Applies identification rules (is this CI new or existing?)
5. Applies reconciliation rules (which data source is authoritative?)
6. Updates CMDB with current, accurate CI information

---

**Q7: What is the MID Server?**
**A:** The MID (Management, Instrumentation, and Discovery) Server is a Java application installed on the internal network. It acts as a secure bridge between the ServiceNow cloud and internal infrastructure — necessary because ServiceNow cloud cannot directly reach internal systems behind firewalls.

**Architecture:**
```
ServiceNow Cloud ←→ MID Server ←→ Firewall ←→ Internal Network
```

---

**Q8: What is the difference between Probes and Sensors?**
**A:**
- **Probe:** Gathers raw data from target systems (like a scout). Example: connects via SSH to a Linux server and collects CPU, RAM, running processes.
- **Sensor:** Processes raw probe data and writes structured CI records to the CMDB.

Together: Probe collects → Sensor writes.

---

**Q9: What are Discovery Patterns?**
**A:** Patterns are reusable discovery logic for complex applications. They define step-by-step how to identify, classify, and populate CI data for specific technologies (e.g., "How to discover a SAP instance"). More powerful than probes/sensors for complex application discovery.

---

**Q10: What is the difference between Horizontal and Vertical Discovery?**
**A:**
| Type | Finds | Example |
|------|-------|---------|
| **Horizontal** | Infrastructure components | Servers, network devices, storage |
| **Vertical** | Applications and their dependencies | Tomcat on Linux with Oracle DB — the full application stack |

Horizontal = infrastructure; Vertical = application stack mapping.

---

## PART 2: Identification & Reconciliation

**Q11: What are Identification Rules?**
**A:** Identification Rules determine whether a discovered record matches an existing CI in the CMDB — preventing duplicate creation.

**Logic:**
- Does a CI with this hostname/IP already exist? → UPDATE existing CI
- No match found? → CREATE new CI

Without good identification rules, every Discovery run creates duplicates!

---

**Q12: What are Reconciliation Rules?**
**A:** When multiple sources provide different values for the same field, Reconciliation Rules determine which source "wins."

**Example:**
- Discovery says server has 32GB RAM
- Manual entry says 16GB RAM
- Reconciliation rule says: Discovery > Manual Entry
- → Discovery's value (32GB) is kept

Rules define priority per data source per attribute.

---

**Q13: What happens if Identification Rules are poorly configured?**
**A:** Duplicate CI records are created every time Discovery runs. Consequences:
- CMDB cluttered with thousands of duplicate records
- Relationship maps broken (wrong CI referenced)
- Impact analysis unreliable
- Reports show inflated CI counts
- Performance degrades

---

## PART 3: CMDB Health & Quality

**Q14: What are the three CMDB Health KPIs?**
**A:**
1. **Completeness:** Are all required fields populated? (e.g., does every server have a Support Group assigned?)
2. **Correctness:** Is the data accurate? (e.g., does the recorded hostname match the actual hostname?)
3. **Compliance:** Does the CMDB data meet defined organizational standards? (e.g., are all CIs properly classified?)

---

**Q15: How do you troubleshoot CMDB data discrepancies?**
**A:**
1. Check the CMDB Health Dashboard — identify which CI classes have issues
2. Review audit logs — who or what last updated the record?
3. Check reconciliation logs — which source won, and was that correct?
4. Review Discovery logs — did discovery complete successfully?
5. Compare against physical source (check the actual server)
6. Fix root cause: update reconciliation rules, fix discovery credentials, correct source data

---

**Q16: What is the CMDB Health Dashboard?**
**A:** A built-in ServiceNow dashboard that:
- Scores CMDB health by Completeness, Correctness, Compliance
- Shows trends over time (improving or degrading?)
- Highlights specific CI classes with poor scores
- Drives targeted remediation efforts

---

## PART 4: Service Mapping & CSDM

**Q17: What is Service Mapping?**
**A:** Service Mapping builds on Discovery to create visual maps showing exactly which CIs support a specific business service end-to-end. 

Example map for "Online Banking Portal":
```
Business Service → Web Servers → App Servers → Databases → Physical Servers
```

When any CI in the chain is at risk, ServiceNow immediately knows which business service is affected.

---

**Q18: What is CSDM?**
**A:** The Common Service Data Model (CSDM) is ServiceNow's prescribed blueprint for organizing CMDB data so all platform modules can share and consume it consistently.

CSDM layers:
```
Business Capability → Business Application → Application Service → Technical Service → Infrastructure CIs
```

Following CSDM ensures ITSM, ITOM, SecOps, SPM, and IRM all see the same data correctly.

---

**Q19: What is the difference between Application CI and Application Service CI?**
**A:**
- **Application CI** (`cmdb_ci_appl`): The software application as a product (e.g., "Salesforce CRM")
- **Application Service** (`cmdb_ci_service`): The running instance of the application supporting a business function (e.g., "Salesforce CRM - Production" with all its infrastructure dependencies mapped)

CSDM requires both to be modeled correctly for full service visibility.

---

## PART 5: Scenario-Based Questions

**Q20: Discovery runs but keeps creating duplicate server records. How do you fix this?**
**A:**
1. Review Identification Rules for the Server CI class
2. Check what attributes are being used to match (hostname? IP? serial number?)
3. If hostname changes dynamically (VMs, containers) → switch to more stable identifier (MAC address, serial)
4. Check if multiple Discovery schedules overlap covering the same IP range
5. Consolidate duplicate records using CMDB deduplication tools
6. Fix identification rules → verify next Discovery run creates no new duplicates

---

**Q21: A major incident occurred. How does CMDB help with impact analysis?**
**A:**
1. Identify the failed CI (e.g., database server DB-PROD-01)
2. Use "Dependency View" to see all CIs that depend on DB-PROD-01
3. Application Service map shows which business services are at risk
4. Business service list drives who gets notified (business owners)
5. Related incidents can be grouped (all incident reports about the same root cause)
6. CMDB data accelerates root cause identification and stakeholder communication

---

**Q22: A manager asks: "Why should I care about CMDB data quality? We only use it for Discovery."**
**A:** CMDB is the foundation for:
- **ITSM:** Incidents and changes reference CIs — inaccurate CIs = poor change impact assessment = more outages
- **SecOps:** Vulnerabilities prioritized by CI business criticality — bad CMDB = wrong priorities
- **SPM:** Projects mapped to services — bad CMDB = can't measure project impact
- **IRM:** Risks linked to business services — bad CMDB = gaps in risk coverage
- **Service Desk:** Agents can't see what a customer's systems are running

"CMDB data quality isn't a technical problem — it's a business risk problem."

---

## PART 6: Exam-Style Quick Facts

| Question | Answer |
|----------|--------|
| CI base table | `cmdb_ci` |
| Relationship table | `cmdb_rel_ci` |
| MID Server purpose | Bridge between ServiceNow cloud and internal network |
| Probe purpose | Collects raw data from targets |
| Sensor purpose | Processes probe data → writes CI records |
| Identification rules | Determine if discovered CI already exists |
| Reconciliation rules | Determine which data source wins on field conflict |
| Horizontal discovery | Infrastructure (servers, network) |
| Vertical discovery | Applications and their dependencies |
| CMDB health KPIs | Completeness, Correctness, Compliance |
| CSDM | Common Service Data Model — structural blueprint |
| Service Mapping | Builds end-to-end maps of business services |

---

## Study Checklist
- [ ] Know: 3 core CMDB tables and their purposes
- [ ] Know: CI vs Asset distinction clearly
- [ ] Know: MID Server role and architecture
- [ ] Know: Probe vs Sensor (collect vs write)
- [ ] Know: Identification rules prevent duplicates
- [ ] Know: Reconciliation rules resolve conflicts
- [ ] Know: Horizontal vs Vertical Discovery
- [ ] Know: 3 CMDB health KPIs
- [ ] Know: CSDM layers
- [ ] Know: Service Mapping purpose
