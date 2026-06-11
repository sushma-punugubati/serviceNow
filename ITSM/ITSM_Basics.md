# ServiceNow ITSM — Fundamentals & Study Guide

> Sources: ServiceNow Docs | ServiceNow Community | ITIL v4 Framework | Now Learning

---

## 1. What is ITSM?

**IT Service Management (ITSM)** is ServiceNow's foundational module that applies structured processes to managing IT services. It implements ITIL (IT Infrastructure Library) best practices to handle the full lifecycle of IT requests, incidents, problems, and changes — replacing email-based, ad-hoc IT support with governed, trackable, SLA-measured service delivery.

> **Think of it as:** Giving IT the same discipline that manufacturing uses for production. Every request has a ticket number, an owner, a deadline, and a documented resolution. Nothing falls through the cracks and every interaction is auditable.

**Core ITSM modules:**
- **Incident Management** — Restore normal service as quickly as possible
- **Problem Management** — Find and eliminate root causes of recurring incidents
- **Change Management** — Implement changes safely with controlled risk
- **Service Catalog** — Structured, self-service IT request fulfillment
- **Request Management** — Track catalog requests through approval and fulfillment
- **Knowledge Management** — Capture and share solutions to avoid repeat effort
- **Service Level Management (SLA)** — Measure and enforce service commitments

---

## 2. Incident Management

### What is an Incident?
An **Incident** is an unplanned interruption to an IT service or reduction in the quality of an IT service. The goal of Incident Management is to restore normal service operation as quickly as possible.

### Incident Priority Matrix

Priority is calculated from **Impact** (breadth of effect) × **Urgency** (time sensitivity):

| Priority | Impact | Urgency | Response Target | Resolution Target |
|----------|--------|---------|-----------------|-------------------|
| P1 - Critical | High | High | 15 min | 4 hours |
| P2 - High | High | Medium or Medium/High | 1 hour | 8 hours |
| P3 - Medium | Medium | Medium | 4 hours | 3 days |
| P4 - Low | Low | Low | 8 hours | 7 days |

### Incident States
| State | Meaning |
|-------|---------|
| New | Just created, not yet assigned |
| In Progress | Being actively worked |
| On Hold | Waiting for customer, third party, or change |
| Resolved | Fix applied, waiting for customer confirmation |
| Closed | Confirmed resolved or auto-closed after X days |
| Cancelled | Not a valid incident |

### Key Incident Fields
| Field | Purpose |
|-------|---------|
| Caller | The person experiencing the issue |
| Assignment Group | Team responsible for resolution |
| Assigned To | Individual owner |
| Category / Subcategory | Classification for routing and reporting |
| Configuration Item (CI) | The affected system from CMDB |
| Impact / Urgency | Drive priority calculation |
| SLA | Tracks response and resolution commitments |

### Major Incident Management
A **Major Incident (MI)** is a P1 incident with significant business impact. MI process typically includes:
- Dedicated bridge/war room
- Separate major incident coordinator role
- Regular stakeholder communications on defined cadence
- Post-Incident Review (PIR) after resolution
- Promotion to Problem record for root cause investigation

### Key Table: `incident`

---

## 3. Problem Management

### What is a Problem?
A **Problem** is the underlying cause of one or more incidents. Problem Management has two goals:
1. **Reactive:** Identify root cause after incidents occur
2. **Proactive:** Identify and eliminate weaknesses before incidents occur

### Problem States
| State | Meaning |
|-------|---------|
| Open | Under investigation |
| Known Error | Root cause identified, workaround documented |
| Closed/Resolved | Permanent fix applied |
| Closed/Cancelled | Not a valid problem |

### Problem vs. Known Error vs. Workaround
| Concept | Definition |
|---------|------------|
| **Problem** | Unknown root cause — investigation in progress |
| **Known Error** | Root cause identified but permanent fix not yet applied |
| **Workaround** | Temporary resolution that restores service without fixing root cause |

> **Interview tip:** A Problem becomes a Known Error when root cause is identified. It stays a Known Error until the permanent fix is implemented.

### Problem Tasks
Problems spawn **Problem Tasks** — specific investigation activities assigned to individuals or teams (e.g., "Review Apache error logs for the past 7 days," "Test database connection pooling under load").

### Post-Incident Review (PIR)
For Major Incidents, a **PIR** is conducted after resolution to document:
- Timeline of events
- Root cause
- What worked well / what didn't
- Action items to prevent recurrence

### Key Table: `problem`

---

## 4. Change Management

### What is a Change?
A **Change** is the addition, modification, or removal of anything that could have an effect on IT services. Change Management ensures changes are implemented safely with appropriate risk assessment and approval.

### Change Types
| Type | Description | Approval |
|------|-------------|----------|
| **Standard Change** | Pre-approved, low-risk, routine (e.g., password reset) | Pre-approved template |
| **Normal Change** | Planned change requiring CAB review | CAB approval required |
| **Emergency Change** | Urgent change to restore service | ECAB (Emergency CAB) expedited approval |

### Change Lifecycle (Normal Change)
```
Draft → Assess → Authorize → Scheduled → Implement → Review → Closed
```

### Change Advisory Board (CAB)
The **CAB** is the group that reviews and approves Normal Changes. Typically meets weekly. Reviews:
- Risk assessment
- Implementation plan
- Rollback plan
- Conflict check (no overlapping maintenance windows)

### Change Risk Assessment
Changes are scored for risk based on:
- Affected CI criticality (from CMDB)
- Change complexity
- Number of affected users
- Time of implementation (peak hours vs. maintenance window)
- Previous failed changes for same CI

### Key Change Fields
| Field | Purpose |
|-------|---------|
| Type | Standard / Normal / Emergency |
| Risk | Low / Medium / High / Critical |
| Impact | Scope of effect |
| Implementation Plan | Step-by-step instructions |
| Backout Plan | How to rollback if change fails |
| Test Plan | How to verify success |
| Conflict CI | CMDB CI that is being changed |

### Key Table: `change_request`

---

## 5. Service Catalog

### What is the Service Catalog?
The **Service Catalog** is the self-service menu of IT services available to employees. Instead of emailing IT helpdesk, employees browse the catalog and submit structured requests — which creates a Request (REQ) with Request Items (RITMs) that route to the correct fulfillment teams.

### Catalog Item Types
| Type | Use Case |
|------|----------|
| **Catalog Item** | Standard request form — one item, one fulfillment team |
| **Record Producer** | Creates a non-catalog record (Incident, Change, Custom table) from a catalog-like form |
| **Order Guide** | Groups multiple catalog items into a bundled request (e.g., "New Employee Setup" creates laptop + software + badge items) |
| **Content Item** | Informational item — links to KB articles or external resources |

### Variables
Each catalog item is configured with **Variables** that collect information from the requester:
| Variable Type | Use Case |
|---------------|----------|
| Single Line Text | Short free-text input |
| Multi-Line Text | Long description or notes |
| Select Box | Single selection from a dropdown |
| Check Box | Yes/No boolean |
| Reference | Lookup to a ServiceNow table record |
| Attachment | File upload |
| Date | Date picker |

### Approval Workflows
Catalog items can have one or more approval steps before fulfillment:
- **Requested for** manager approval
- **IT manager** approval for expensive items
- **Security team** approval for privileged access requests

### Execution Plans and Catalog Tasks
Complex catalog items can have **Execution Plans** (sets of catalog tasks) that route different parts of the fulfillment to different teams:
- Task 1: IT — provision laptop (assigned to hardware team)
- Task 2: IT — create AD account (assigned to identity team)
- Task 3: Facilities — assign desk and badge (assigned to facilities team)

### Key Tables
| Table | Label |
|-------|-------|
| `sc_request` | Request (REQ) |
| `sc_req_item` | Request Item (RITM) |
| `sc_task` | Catalog Task |
| `sc_cat_item` | Catalog Item |
| `sc_variable` | Variable |

---

## 6. Request Lifecycle

A typical request flows through these stages:

```
Employee submits catalog item → REQ created (Request)
                                    ↓
                             RITM created (Request Item) — one per catalog item
                                    ↓
                          Approval workflow (if required)
                                    ↓
                       Catalog Tasks created (for multi-team fulfillment)
                                    ↓
                    All tasks complete → RITM auto-closes
                                    ↓
                    All RITMs closed → REQ auto-closes
```

### Request States
| State | Meaning |
|-------|---------|
| Draft | Not yet submitted |
| Requested | Submitted, awaiting approval |
| Approved | Approved, fulfillment in progress |
| In Fulfillment | Active work underway |
| Completed | All tasks done |
| Cancelled | Request withdrawn |

---

## 7. Knowledge Management

### What is Knowledge Management?
**Knowledge Management** captures proven solutions, how-to guides, and reference information that agents and end-users can search rather than re-solving the same problems repeatedly.

### Knowledge Base Structure
```
Knowledge Base
  └── Category
        └── Knowledge Article
```

### Article States
| State | Meaning |
|-------|---------|
| Draft | Being authored |
| Review | Awaiting approval before publish |
| Published | Live and searchable |
| Retired | Removed from search, kept for history |

### Knowledge Block
A **Knowledge Block** is a reusable content component that can be embedded in multiple articles — useful for standard disclaimer text, common troubleshooting steps, or contact information that appears in many articles.

### Feedback and Improvement
- Employees can rate articles (Helpful / Not Helpful)
- Feedback creates improvement tasks for the article owner
- Articles with low ratings are flagged for review

### KB Deflection
Catalog items can display related KB articles during the request process — if the employee finds the answer in a KB article, they do not need to submit a request. This is the primary self-service deflection mechanism in ITSM.

---

## 8. Service Level Management (SLA)

### What is an SLA?
A **Service Level Agreement (SLA)** defines the commitment for responding to and resolving service requests or incidents within a specified timeframe.

### Key SLA Concepts
| Concept | Definition |
|---------|------------|
| **SLA Definition** | The template that defines the conditions, timer, and target for an SLA |
| **SLA Task (SLA record)** | The instance created on a specific ticket — tracks elapsed time |
| **Response SLA** | Measures time to first substantive response to the requester |
| **Resolution SLA** | Measures time from creation to resolution |
| **Breach** | SLA target exceeded — action not completed in time |
| **Pause Condition** | State/condition under which the SLA clock stops (e.g., On Hold, Awaiting Customer) |

### SLA Stages
| Stage | Meaning |
|-------|---------|
| In Progress | Timer running |
| Paused | Timer stopped (On Hold state or other pause condition) |
| Complete | Target met — task completed within SLA |
| Breached | Target missed — task not completed in time |

### Retroactive Pause
**Retroactive pause** allows an SLA to be paused back in time if the pause condition was set after-the-fact. For example, if a ticket was put On Hold at 2 PM but the pause condition wasn't configured until 3 PM, retroactive pause credits the pause back to 2 PM.

### Business Hours
SLAs can run on **business hours** (e.g., 9am-5pm Monday-Friday) rather than 24/7. The SLA timer only counts down during configured business hours. Timezone is also configurable per SLA definition.

---

## 9. Key ITSM Tables Reference

| Table | Label | Key Notes |
|-------|-------|-----------|
| `incident` | Incident | Core incident record |
| `problem` | Problem | Root cause investigation |
| `change_request` | Change | Change record |
| `sc_request` | Request | REQ — parent of RITMs |
| `sc_req_item` | Request Item | RITM — individual catalog item request |
| `sc_task` | Catalog Task | Fulfillment task from catalog item |
| `sc_cat_item` | Catalog Item | Self-service request form |
| `kb_knowledge` | Knowledge | KB articles |
| `contract_sla` | SLA Definition | SLA template/definition |
| `task_sla` | SLA (task record) | Instance of SLA on a specific ticket |
| `sys_user_group` | Group | Assignment groups |
| `cmn_schedule` | Schedule | Business hours definitions |
