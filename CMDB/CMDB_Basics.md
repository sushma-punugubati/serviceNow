# ServiceNow CMDB — Fundamentals & Study Guide

> Sources: [ServiceNow CMDB Community](https://www.servicenow.com/community/cmdb-forum/top-cmdb-amp-discovery-interview-questions/m-p/3306831) | [ServiceNow Docs](https://www.servicenow.com/docs/bundle/zurich-servicenow-platform/page/product/configuration-management/reference/cmdb-tables-details.html)

---

## 1. What is the CMDB?

The **Configuration Management Database (CMDB)** is the central repository in ServiceNow that stores information about all Configuration Items (CIs) — every component needed to deliver IT services — and the **relationships** between them.

> **Think of it as:** A living map of your entire IT infrastructure. Every server, application, network device, and service is a dot on the map, and the lines between them show how they're connected. When something breaks, you know what else it affects.

**The CMDB is NOT just an inventory list.** Its real power is in the **relationships** — understanding that Server A runs Application B which supports Business Service C means that when Server A goes down, you know exactly which business service is at risk.

---

## 2. Configuration Item (CI)

A **CI** is any component that needs to be managed to deliver a service successfully.

**Examples:**
- A physical server (Hardware CI)
- A database instance (Application CI)
- A business application like Salesforce (Application Service)
- A network switch (Network Device CI)
- A virtual machine (VM CI)

**CI attributes typically include:**
- Name, Class, Status, Manufacturer
- Location, Company, Department
- Maintenance Schedule, Support group
- Relationships to other CIs

---

## 3. CMDB Table Hierarchy

All CIs extend from a base table hierarchy:

```
cmdb (Base — abstract)
  └── cmdb_ci (Configuration Item — all CIs live here)
        ├── cmdb_ci_hardware (Hardware)
        │     ├── cmdb_ci_computer (Computers/Workstations)
        │     │     └── cmdb_ci_server (Servers)
        │     └── cmdb_ci_netgear (Network Gear)
        ├── cmdb_ci_appl (Application)
        ├── cmdb_ci_service (Service)
        │     └── cmdb_ci_business_app (Business Application)
        ├── cmdb_ci_db_instance (Database Instance)
        ├── cmdb_ci_network (Network Device)
        └── cmdb_ci_vm_instance (Virtual Machine Instance)
```

**Key Tables:**

| Table Name | Label | Purpose |
|-----------|-------|---------|
| `cmdb` | CMDB Base | Abstract base table |
| `cmdb_ci` | Configuration Item | Root CI table — all CIs |
| `cmdb_rel_ci` | CI Relationship | Stores relationships between CIs |
| `cmdb_ci_hardware` | Hardware | Physical hardware CIs |
| `cmdb_ci_computer` | Computer | Computers and workstations |
| `cmdb_ci_server` | Server | Server CIs |
| `cmdb_ci_appl` | Application | Application CIs |
| `cmdb_ci_service` | Service | Service CIs |
| `cmdb_ci_db_instance` | Database | Database instance CIs |
| `cmdb_ci_network` | Network Device | Network gear CIs |
| `cmdb_ci_vm_instance` | Virtual Machine | VM instance CIs |
| `cmdb_ci_cloud_service_account` | Cloud Service Account | Cloud resource CIs |
| `cmdb_ci_endpoint` | Endpoint | Endpoint/workstation CIs |

---

## 4. CI Relationships

The `cmdb_rel_ci` table stores relationships between CIs using relationship types:

| Relationship Type | Example |
|------------------|---------|
| **Depends on / Used by** | Application depends on Server |
| **Runs on / Hosts** | Database runs on Server |
| **Virtualized by / Virtualizes** | VM virtualized by Hypervisor |
| **Connected to** | Switch connected to Router |
| **Managed by / Manages** | Application managed by Team |
| **Contained in / Contains** | Server contained in Data Center |

**Why relationships matter:**
- **Impact Analysis:** If Server X goes down, which applications and services are affected?
- **Change Planning:** Before changing Server X, see all dependencies
- **Root Cause Analysis:** Trace an outage back through the dependency chain

---

## 5. CI vs. Asset — Critical Difference

| CI (Configuration Item) | Asset |
|------------------------|-------|
| Logical or physical infrastructure component | Financial/inventory record |
| Focus: configuration, relationships, dependencies | Focus: cost, ownership, depreciation |
| Table: `cmdb_ci` | Table: `alm_asset` |
| Managed by CMDB/ITOM team | Managed by ITAM team |
| Exists when CI is "in service" | Exists from procurement |
| Supports: incident, change, problem management | Supports: financial tracking, compliance |

> A server has BOTH an Asset record (what you paid for it, when it was bought) AND a CI record (what it runs, what depends on it). They are linked but serve different purposes.

---

## 6. ServiceNow Discovery

**Discovery** is ServiceNow's tool for automatically finding and documenting CIs in your environment.

### MID Server
The **MID (Management, Instrumentation, and Discovery) Server** is a Java application installed on your network that acts as a secure bridge between ServiceNow (cloud) and your internal infrastructure.

```
ServiceNow Cloud ←→ MID Server ←→ Internal Network (Servers, Devices, Apps)
```

**Why MID Server?** Your internal infrastructure is behind firewalls. The MID Server can reach it; the ServiceNow cloud cannot directly.

### How Discovery Works

```
1. Schedule triggers Discovery for an IP range
2. MID Server scans the network (ICMP ping, port scan)
3. Devices found → MID Server sends Probes
4. Probes gather raw data (OS, running services, installed apps)
5. Sensors process raw data → structured CI records
6. Identification rules check: does this CI already exist?
7. Reconciliation rules decide: which data source wins if conflict?
8. CMDB updated with accurate, current CI data
```

### Horizontal vs. Vertical Discovery

| Type | Finds | Example |
|------|-------|---------|
| **Horizontal** | Infrastructure components | Servers, network devices, storage |
| **Vertical** | Applications and dependencies | Tomcat running on Linux with Oracle DB |

### Probes and Sensors

| Component | Role |
|-----------|------|
| **Probe** | Gathers raw data from target systems (like a data collector) |
| **Sensor** | Processes probe data and writes structured CI records to CMDB |
| **Pattern** | Reusable discovery logic for complex applications — defines what to find, classify, and populate |

---

## 7. Identification and Reconciliation Rules

### Identification Rules
**"Does this discovered record match an existing CI?"**

Before creating a new CI, Discovery checks:
- Does a CI with this hostname already exist?
- Does a CI with this serial number already exist?
- → If match found: UPDATE the existing CI
- → If no match: CREATE a new CI

> Without good identification rules: every Discovery run creates duplicate CIs!

### Reconciliation Rules
**"When multiple sources provide different values for the same field — who wins?"**

Example: Discovery says a server has 16GB RAM. ServiceNow says 8GB. Which is correct?

Reconciliation rules define source priority:
- Source 1 (Discovery) > Source 2 (Manual Entry) > Source 3 (Import)
- Priority-based: highest priority source wins per attribute

> Without reconciliation rules: last writer wins — data quality deteriorates.

---

## 8. CMDB Health

ServiceNow measures CMDB health using three KPIs:

| KPI | Question | Example |
|-----|----------|---------|
| **Completeness** | Are all required fields populated? | Does every server have a support group? |
| **Correctness** | Is the data accurate? | Does the recorded IP match the actual IP? |
| **Compliance** | Does the CMDB match defined standards? | Are all CIs classified correctly? |

**CMDB Health Dashboard:** Visualizes KPI scores across CI classes. Used to identify and prioritize data quality improvements.

---

## 9. Service Mapping

**Service Mapping** builds on Discovery to create **Business Service maps** — showing exactly which CIs support a specific business service end-to-end.

```
Business Service: "Online Banking Portal"
  ↓ depends on
Web Server (Apache) [cmdb_ci_server]
  ↓ depends on
Application Server (Tomcat) [cmdb_ci_appl]
  ↓ depends on
Database (Oracle) [cmdb_ci_db_instance]
  ↓ runs on
Physical Server [cmdb_ci_server]
```

**Value:** When the database slows down, you immediately know the Online Banking Portal is at risk — and so does the business.

---

## 10. CSDM — Common Service Data Model

The **CSDM** is ServiceNow's blueprint for how data should be organized in the CMDB to support all ServiceNow products.

**CSDM layers (simplified):**
```
Business Capability (what the business does)
  ↓
Business Application (the software supporting it)
  ↓
Application Service (the running instance)
  ↓
Technical Service (the infrastructure supporting it)
  ↓
Infrastructure CIs (servers, databases, networks)
```

**Why CSDM matters:** If your CMDB follows CSDM, all ServiceNow products (ITSM, ITOM, SPM, IRM, SecOps) can share and leverage the same data consistently.

---

## 11. Key Roles

| Role | Responsibilities |
|------|----------------|
| **CMDB Manager** | Governance, data quality standards, policy enforcement |
| **Configuration Manager** | Day-to-day CMDB operations and data quality |
| **Discovery Admin** | Maintains Discovery schedules, MID Servers, credentials |
| **Service Mapping Engineer** | Builds and maintains Business Service maps |
| **Data Steward** | Owns specific CI classes; accountable for data quality |
| **Change Manager** | Uses CMDB for change impact analysis |

---

## 12. CI Lifecycle States

| State | Meaning |
|-------|---------|
| **Ordered** | CI on order, not yet received |
| **Installed** | CI deployed and in service |
| **In Maintenance** | CI temporarily offline for maintenance |
| **Absent** | CI missing or cannot be discovered |
| **Stolen** | CI reported stolen |
| **Retired** | CI decommissioned, out of service |

---

## 13. Glossary

| Term | Simple Explanation |
|------|-------------------|
| **CI** | Configuration Item — any manageable component |
| **CMDB** | Database storing all CIs and their relationships |
| **MID Server** | Bridge between ServiceNow cloud and your internal network |
| **Probe** | Data collector sent to target systems |
| **Sensor** | Processor that turns probe data into CI records |
| **Pattern** | Reusable discovery logic for complex application discovery |
| **Identification Rule** | Determines if a discovered CI already exists |
| **Reconciliation Rule** | Decides which data source wins when conflict exists |
| **Horizontal Discovery** | Finds infrastructure components |
| **Vertical Discovery** | Maps applications and their dependencies |
| **Service Mapping** | Builds end-to-end maps of business services |
| **CSDM** | Common Service Data Model — blueprint for CMDB structure |
| **CMDB Health** | Measures completeness, correctness, and compliance |
| **CI Class** | Category of CI (Server, Application, Network Device) |
| **Dependency** | Relationship showing one CI relies on another |
