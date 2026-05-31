# ServiceNow CSDM — Fundamentals & Study Guide

> Sources: ServiceNow Docs | CSDM Implementation Guide | ServiceNow Community | Now Learning CSDM Whitepaper

---

## 1. What is CSDM?

The **Common Service Data Model (CSDM)** is ServiceNow's prescriptive framework for structuring CMDB data. It defines a standard way to classify, relate, and organize Configuration Items — so that all ServiceNow modules (ITSM, ITOM, SecOps, IRM, CSM) consistently read from and write to the same structured data model.

> **Think of it as:** A universal blueprint for your CMDB. Without CSDM, every team draws their own map of the IT landscape differently. With CSDM, everyone uses the same map — and every ServiceNow feature that depends on CMDB data works as designed.

**Why CSDM matters:**
- **Service Mapping** depends on CSDM relationship types to build accurate topology maps
- **Change Management** depends on CSDM to calculate correct impact and risk
- **Event Management** depends on CSDM to correlate alerts to business services
- **Reporting** depends on CSDM for consistent, cross-module dashboards
- **SecOps** depends on CSDM to prioritize vulnerabilities by business service criticality

**Without CSDM:** Teams use different CI classes, different relationship types, different naming conventions. Features like impact analysis, BSM maps, and service health dashboards produce unreliable or empty results.

---

## 2. CSDM Layers and Domains

CSDM organizes the IT landscape into four domains, each representing a different perspective on IT:

### The Four CSDM Domains

```
Business Management
        ↓
Service Portfolio Management
        ↓
Application Portfolio Management
        ↓
Technical / Infrastructure Management
```

### CSDM Layers in Detail

| Layer | Key Entity | What it represents |
|-------|-----------|-------------------|
| **Business Capability** | Business Capability CI | What the business does (e.g., "Process Payments") |
| **Business Application** | `cmdb_ci_business_app` | The application that enables the capability (e.g., "Payment Processing App") |
| **Application Service** | `cmdb_ci_service` | The running, deployed instance of a Business Application (e.g., "Payment App - Production") |
| **Technical Service** | `cmdb_ci_service_technical` | The technical platform layer supporting Application Services (e.g., "Payment App DB Cluster") |
| **Infrastructure** | `cmdb_ci_server`, `cmdb_ci_vm_instance`, etc. | The servers, VMs, and hardware that everything runs on |

### The Critical Bridge — Application Service

The **Application Service** is the most important CSDM entity. It is the bridge between the business layer (Business Application → Business Capability) and the technical layer (Technical Services → Infrastructure).

- Looking **up**: Application Service is what business stakeholders care about — "Is our payment processing working?"
- Looking **down**: Application Service depends on Technical Services and Infrastructure CIs

This is why Service Mapping targets Application Services — they are the starting point for tracing technical dependencies.

---

## 3. Key CSDM Entities

### Business Application (`cmdb_ci_business_app`)
Represents an application as a business asset — the software product itself (not a specific deployment).

**Examples:** Salesforce CRM, Oracle ERP, Custom Order Management System

**Key attributes:** Application Portfolio, Business Owner, IT Owner, Application Lifecycle State, Vendor

### Application Service (`cmdb_ci_service`)
Represents a specific deployment/instance of a Business Application in a specific environment.

**Examples:** "Salesforce CRM - Production," "Order Management - UAT"

**Key attributes:** Business Application (parent), Environment, Operational Status, Assignment Group, Service Portfolio

> **Key distinction:** Business Application = what you bought. Application Service = what you deployed and run.

### Service Offering (`service_offering`)
Defines how a service is offered to consumers — the terms, conditions, price, and commitments for a service.

**Examples:** "Standard Support - 8x5," "Premium Support - 24x7," "Free Tier - Self-Service Only"

**Relationship:** Service Portfolio → Service Offering → Application Service

### Technical Service (`cmdb_ci_service_technical`)
Represents a shared technical platform or infrastructure grouping that multiple Application Services depend on.

**Examples:** "Oracle RAC Database Cluster," "Apache Web Server Farm," "Message Queue Infrastructure"

### Business Service (`cmdb_ci_service`)
A service that the business defines as valuable and wants to track — often aligned with Service Catalog offerings.

**Examples:** "Email Service," "Remote Access VPN," "Finance Reporting"

---

## 4. CSDM Maturity Model

ServiceNow defines five maturity levels for CSDM adoption:

| Level | Name | What it means |
|-------|------|---------------|
| **Foundation** | Crawl | Basic CMDB populated — hardware and software CIs exist but no service relationships |
| **Crawl** | Walk | Service relationships beginning — Application Services linked to infrastructure CIs |
| **Walk** | Run | Application Portfolio established — Business Applications mapped to Application Services |
| **Run** | Fly | Service Offerings and Business Capabilities defined — full CSDM model operational |
| **Fly** | Optimize | Continuous improvement — CSDM data drives business decisions, cost allocation, and AI |

**Most organizations start at Foundation** (hardware CIs in CMDB, no service layer) and target Walk/Run for operational benefits. Full Fly requires significant business engagement beyond IT.

---

## 5. CSDM Relationship Types

CSDM defines standard relationship types that replace ad-hoc or custom relationship names:

| Relationship Type | Used Between | Meaning |
|------------------|-------------|---------|
| **Runs on / Hosts** | Application → Server | Application runs on this server |
| **Depends on / Used by** | Service → Service | This service depends on another service |
| **Hosted on / Hosts** | Database → Server | Database instance hosted on this server |
| **Managed by / Manages** | CI → Assignment Group | This team manages this CI |
| **Virtualized by / Virtualizes** | VM → Hypervisor | VM runs on this hypervisor |
| **Connected to** | Network Device → Network Device | Devices are physically connected |
| **Member of / Contains** | Server → Cluster | Server is a member of this cluster |

**Why standardizing relationship types matters:** Service Mapping, Event Management, and Impact Analysis all traverse CI relationships using specific relationship types. If relationships use non-standard types, these features cannot navigate them correctly.

---

## 6. Service Graph Connectors

**Service Graph Connectors** are pre-built integrations that populate CMDB following CSDM standards. They replace raw import sets that created CI records without relationships or CSDM alignment.

### Service Graph Connector vs. Raw Import Set

| Raw Import Set | Service Graph Connector |
|----------------|------------------------|
| Creates CI records — no relationships | Creates CIs AND standard CSDM relationships |
| CI class may not match CSDM | CSDM-aligned CI classes |
| Reconciliation rules not pre-configured | Pre-configured IRE identification and reconciliation |
| Manual mapping required | Out-of-box mapping for the data source |
| Relationship types: custom or none | Standard CSDM relationship types |

### Available Service Graph Connectors

ServiceNow provides connectors for major data sources:
- **SCCM / Intune** — Windows hardware and software inventory
- **VMware vSphere** — Virtual machine inventory and hypervisor relationships
- **AWS** — Cloud resource inventory
- **Azure** — Azure VM, storage, network inventory
- **Jamf** — macOS/iOS device inventory
- **CrowdStrike** — Endpoint security data
- **ServiceNow CMDB Health** — Internal CMDB quality assessment

### How Service Graph Connectors Work

1. Data source (SCCM, vSphere) exports data in the connector's expected format
2. Connector transformation maps source data to CSDM CI classes and attributes
3. IRE identification rules (pre-configured by connector) find existing CIs or create new ones
4. Relationship records created using CSDM-standard relationship types
5. Reconciliation rules determine which data source wins for each attribute

---

## 7. CSDM Health Score

ServiceNow provides a **CMDB Health Dashboard** that measures CSDM compliance across four dimensions:

| Dimension | What it measures |
|-----------|-----------------|
| **Completeness** | Are required CI attributes populated? |
| **Correctness** | Are attribute values valid (no nulls in mandatory fields, correct formats)? |
| **Compliance** | Do CIs use the correct CSDM CI classes and relationship types? |
| **Orphaned CIs** | Are there CIs with no relationships (isolated records with no context)? |

**Health Score interpretation:**
- 85%+: Good — CSDM well-maintained, features relying on CMDB data work reliably
- 60-84%: Moderate — Some modules may have unreliable results, improvement needed
- Below 60%: Poor — CMDB data quality too low for reliable impact analysis or service mapping

**Improving the score:** CSDM health improvement is most effective when broken into CI-class-level targets assigned to specific team owners (not a single "CMDB team" responsible for everything).

---

## 8. CSDM and ITOM Integration

CSDM and ITOM are deeply interdependent:

- **Discovery** populates the infrastructure layer (servers, network devices) — feeds CSDM's bottom layer
- **Service Graph Connectors** populate with CSDM-aligned relationships — better than raw Discovery alone
- **Service Mapping** builds Application Service → Technical Service → Infrastructure relationships — populates the middle CSDM layers
- **Event Management** binds alerts to CIs and traverses CSDM relationships upward to determine business service impact

The quality of CSDM directly determines the quality of ITOM outputs:
- Poor CSDM → Event Management cannot identify which business services are affected by an alert
- Good CSDM → Event Management immediately shows business service impact, affected team, and SLA implications

---

## 9. Key CSDM Tables Reference

| Table | Label | CSDM Layer |
|-------|-------|-----------|
| `cmdb_ci_business_app` | Business Application | Application Portfolio |
| `cmdb_ci_service` | Application Service / Business Service | Service Portfolio |
| `cmdb_ci_service_technical` | Technical Service | Technical |
| `service_offering` | Service Offering | Service Portfolio |
| `cmdb_ci_server` | Server | Infrastructure |
| `cmdb_ci_vm_instance` | Virtual Machine | Infrastructure |
| `cmdb_ci_appl` | Application | Infrastructure / Technical |
| `cmdb_rel_ci` | CI Relationship | All layers (relationship store) |
| `cmdb_health_result` | CMDB Health Result | Health scoring |
| `sn_cmdb_csdm_mapping` | CSDM Mapping | CSDM compliance tracking |
