# ServiceNow ESP — Enterprise Services Portfolio: Basics

---

## 1. What Is the Enterprise Services Portfolio?

The **Enterprise Services Portfolio (ESP)** is the discipline and platform capability for managing the full lifecycle of services that an organization offers — from concept through active delivery to retirement.

**Think of it this way:**
- A restaurant has a **menu** (what's available to order today = **Service Catalog**)
- But the restaurant also has **dishes in development** (new services being built = **Pipeline**)
- And **dishes they used to serve but removed** (retired services)

The **Service Portfolio** = ALL three combined. The **Service Catalog** = just the current menu.

### Key Goals of ESP
1. **Visibility:** What services does the organization actually offer? Who owns them?
2. **Value transparency:** What does each service cost? What value does it deliver?
3. **Lifecycle governance:** When should a service be built, changed, or retired?
4. **Quality accountability:** Service Owners responsible for SLAs and customer satisfaction
5. **Enterprise-wide:** Extends beyond IT to HR, Finance, Legal, and Facilities services

---

## 2. Service Portfolio vs. Service Catalog

This distinction is critical in interviews:

| Aspect | Service Portfolio | Service Catalog |
|--------|-----------------|----------------|
| **What it contains** | ALL services: pipeline + active + retired | Only ACTIVE services available for request |
| **Who uses it** | IT leadership, Service Owners, CIO | End users, business requestors |
| **Purpose** | Strategic management of service lifecycle | Fulfillment and request management |
| **Visibility** | Internal governance view | Public-facing or employee-facing |
| **In ServiceNow** | Service Portfolio module + service CI hierarchy | Service Catalog (`sc_category`, `sc_cat_item`) |

**Simple rule:** Everything in the Catalog is in the Portfolio, but not everything in the Portfolio is in the Catalog (pipeline and retired items are portfolio-only).

---

## 3. Service Lifecycle Stages

```
PIPELINE → CATALOG (Active) → RETIRED
```

| Stage | Meaning | Examples |
|-------|---------|---------|
| **Pipeline** | Service being designed/built — not yet available | New cloud storage service in development |
| **Active / Catalog** | Live service available for request or consumption | Laptop provisioning, VPN access |
| **Retired** | No longer offered — historical record maintained | Legacy VPN replaced by Zero Trust |

**In ServiceNow:** Service CI records (`cmdb_ci_service`) have a **Lifecycle Stage** field tracking this.

---

## 4. Key Concepts in ESP

### Service Offering
A Service Offering is a specific, scoped package of a service delivered to a particular customer segment with defined terms.

**Example:**
- **Service:** "Cloud Storage"
- **Service Offering A:** 1TB storage, standard support, $50/user/year (for general staff)
- **Service Offering B:** 10TB storage, premium support, SLA, $200/user/year (for data science teams)

One service can have multiple offerings targeting different audiences.

### Service Owner
The individual accountable for the quality, cost, performance, and lifecycle of a service. Not the same as a project manager — the Service Owner has ongoing accountability, not just during delivery.

**Responsibilities:**
- Define and maintain service catalog entries
- Set and monitor SLAs
- Approve changes to the service
- Make retire/replace decisions
- Report on service health to leadership

### Service Consumer
The business unit or individual that receives and uses a service. In the ESP model, consumer satisfaction is a key metric alongside technical performance.

### Service Level Agreement (SLA)
Contractual commitments on service performance — uptime, response time, resolution time. In ServiceNow, SLAs are tracked against Service Offerings in the Catalog.

---

## 5. The CSDM Hierarchy and ESP

ESP only works with properly structured CMDB data following the **Common Service Data Model (CSDM)**:

```
Business Capability
        ↓
Business Application (e.g., "HR Recruiting Platform")
        ↓
Application Service (e.g., "HR Recruiting Platform - Production")
        ↓
Technical Service (e.g., "SAP HCM Technical Service")
        ↓
Infrastructure CIs (servers, databases, network devices)
```

**Why this matters for ESP:**
- Service Portfolio records link to **Application Services** in CSDM
- Impact analysis: if server fails → Application Service degraded → Service Portfolio entry affected
- Cost attribution: infrastructure cost rolls up to Application Service rolls up to Business Application

Without CSDM alignment, ESP data is disconnected from actual IT reality.

---

## 6. Key Tables in ServiceNow ESP

| Table | What It Stores |
|-------|---------------|
| `service_offering` | Service Offering records — specific packaged offerings to consumers |
| `cmdb_ci_service` | Service CI — technical service record in CMDB hierarchy |
| `cmdb_ci_business_app` | Business Application CI — application supporting business functions |
| `sc_category` | Service Catalog categories (groupings) |
| `sc_cat_item` | Catalog Items — what users can actually request |
| `service_portfolio` | Service Portfolio records |
| `sla_definition` | SLA definitions linked to service offerings |
| `cmdb_ci_service_auto` | Application Service CI (auto-populated by Service Mapping) |

---

## 7. Service Catalog Management

The **Service Catalog** is the user-facing front end of the Service Portfolio — the store where employees and business users request services.

### Catalog Structure
```
Service Catalog
├── Category (e.g., "IT Hardware")
│   ├── Catalog Item (e.g., "Request New Laptop")
│   │   ├── Variables (form fields for the request)
│   │   ├── Workflows (fulfillment steps)
│   │   └── SLA (how fast it should be fulfilled)
│   └── Catalog Item (e.g., "Request Monitor")
└── Category (e.g., "Software & Access")
    ├── Catalog Item (e.g., "Request Software License")
    └── Catalog Item (e.g., "Request VPN Access")
```

### Catalog Item Types
- **Request Items:** Tangible deliverables (laptop, software)
- **Service Items:** Ongoing services (cloud storage subscription)
- **Record Producers:** Creates ITSM records (incident, change) from the catalog
- **Order Guides:** Bundles of items ordered together (new employee onboarding kit)
- **Content Items:** Information only — no fulfillment workflow

---

## 8. Enterprise Service Management (ESM)

**ESM** is the strategic direction where ServiceNow extends the IT service management model to ALL enterprise departments:

| Department | Service Examples |
|-----------|----------------|
| **IT** | Laptop provisioning, VPN, software licenses |
| **HR** | Onboarding, leave requests, policy questions |
| **Finance** | Expense reports, budget requests, vendor payments |
| **Legal** | Contract review, NDA requests, compliance questions |
| **Facilities** | Office access, parking, building maintenance |
| **Security** | Physical access cards, background checks |

**ESP in ESM context:** A unified Service Portfolio spanning ALL departments gives leadership a single view of everything the enterprise offers to its employees and customers.

---

## 9. Service Portfolio Financial Management

A mature ESP tracks the **cost** and **value** of each service:

### Cost Tracking
- Infrastructure costs allocated to services (via CMDB and ITAM)
- Labor costs allocated (via ITSM time tracking and SPM timecards)
- License costs allocated (via ITAM SAM)
- Total Cost of Service = Infrastructure + Labor + Licenses

### Value Tracking
- Business outcomes delivered by the service
- User satisfaction (CSAT scores from service catalog requests)
- Usage metrics (how many employees use this service?)
- Business unit adoption rate

### Decision Support
When service cost and value are tracked together, Portfolio Managers can ask:
- "This service costs $2M/year but only 50 of our 10,000 employees use it — should we retire it?"
- "This service has 99.9% satisfaction and serves 8,000 employees — let's invest in it"

---

## 10. ESP Integration with Other ServiceNow Modules

| Integration | How It Works |
|------------|-------------|
| **CMDB/CSDM** | Services linked to Application Services and infrastructure CIs — impact analysis works |
| **ITSM** | Incidents reference affected services; change requests must link to service being changed |
| **SPM** | Projects in SPM are associated with the service being built or changed |
| **ITAM** | Software licenses and hardware assets supporting each service tracked |
| **IRM** | Services have risk posture; compliance requirements mapped to services |
| **SecOps** | Vulnerability findings linked to services; business criticality from service portfolio drives VR priority |
| **HRSD** | HR services in the catalog; employee onboarding items link to HRSD workflows |

---

## 11. Service Health and Reporting

ServiceNow ESP provides service health visibility through:

### Service Health Dashboard
- **Availability:** Is the service currently up?
- **Open Incidents:** Active incidents affecting this service
- **Change Risk:** Upcoming changes that could impact the service
- **SLA Compliance:** Are fulfillment SLAs being met?
- **Customer Satisfaction:** CSAT scores from recent requests

### Executive Service Portfolio Report
- Portfolio-level view: How many services in each lifecycle stage?
- Which services are meeting their SLAs?
- Which services have the most incidents? (Candidates for remediation)
- Which services are candidates for retirement?

---

## 12. Roles in ESP

| Role | Responsibilities |
|------|----------------|
| **Service Portfolio Manager** | Governs the portfolio lifecycle; retires and adds services strategically |
| **Service Owner** | Accountable for one service's quality, cost, and lifecycle |
| **Catalog Manager** | Maintains the catalog items — content, workflows, categories |
| **Service Consumer (Business Owner)** | Receives the service; provides feedback and requirements |
| **IT Operations** | Delivers the underlying technical services |
| **Enterprise Architect** | Ensures ESP/CSDM alignment; governs service model design |

---

## 13. ESP vs. SPM — A Key Distinction

A very common interview question:

| Aspect | ESP | SPM |
|--------|-----|-----|
| **Focus** | Managing services that EXIST | Managing projects/initiatives to CREATE or CHANGE |
| **Time horizon** | Ongoing (run the business) | Time-bounded (change the business) |
| **Key question** | "What services do we offer? Are they healthy?" | "What projects are we investing in?" |
| **Primary user** | Service Owners, IT leadership | Portfolio Managers, PMs |
| **Output** | Service Portfolio health reports, catalog | Project delivery, roadmaps |
| **Relationship** | SPM projects transition INTO the ESP when complete | SPM manages the work; ESP manages the result |

**They are complementary:** SPM delivers the project → the new/changed service lands in the Service Portfolio → ESP manages it going forward.

---

## 14. Common Glossary

| Term | Definition |
|------|-----------|
| **Service Portfolio** | Complete catalog of all services (pipeline + active + retired) |
| **Service Catalog** | Active services available for end-user request |
| **Service Offering** | A specific scoped package of a service for a defined consumer segment |
| **Service Owner** | Individual accountable for a service's quality and lifecycle |
| **CSDM** | Common Service Data Model — blueprint for organizing service data |
| **ESM** | Enterprise Service Management — extending service management beyond IT |
| **Pipeline** | Services planned or in development, not yet in the catalog |
| **Retired** | Services no longer offered |
| **SLA** | Service Level Agreement — performance commitments |
| **CSAT** | Customer Satisfaction score |
| **Record Producer** | Catalog item that creates a record (incident, case) in ServiceNow |
| **Order Guide** | Bundle of catalog items ordered together as a set |
