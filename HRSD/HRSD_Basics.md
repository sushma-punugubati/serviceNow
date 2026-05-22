# ServiceNow HRSD — Fundamentals & Study Guide

> Sources: ServiceNow Docs | ServiceNow Community | HRSD Implementation Guide | Now Learning

---

## 1. What is HRSD?

**HR Service Delivery (HRSD)** is ServiceNow's module that transforms HR operations into a structured, trackable service delivery model — bringing the same case management, routing, and self-service capabilities that IT uses for ITSM into the HR department.

> **Think of it as:** Giving HR the same discipline IT has. Instead of employees emailing HR inboxes or walking up to desks, every HR request becomes a tracked case with an owner, SLA, and audit trail.

**Core value:**
- Employees get consistent, timely answers regardless of which HR rep handles their case
- HR teams get workload visibility, SLA tracking, and compliance audit trails
- Management gets reporting on request volumes, resolution times, and satisfaction scores

---

## 2. Core Plugin

| Plugin | ID |
|--------|----|
| Human Resources Scoped App: Core | `com.sn_hr_core` |
| Human Resources: Employee Document Management | `com.sn_hr_core.edm` |
| Human Resources: Lifecycle Events | `com.sn_hr_core.lifecycle_events` |
| Employee Center | `com.sn_hr_core.esc` |
| Employee Center Pro | `com.sn_hr_core.esc_pro` |
| Employee Journey Management | `com.sn_hr_core.ejm` |

---

## 3. Centers of Excellence (COE) — The Core Architecture

### What is a COE?
A **Center of Excellence (COE)** is a functional grouping of HR services that maps to a specific HR department or function. Each COE acts as a logical routing destination — cases are routed to a COE, then to an assignment group within that COE.

### Standard OOTB COEs
| COE | Scope |
|-----|-------|
| **Benefits** | Benefits enrollment, changes, COBRA, FSA/HSA |
| **Payroll** | Pay inquiries, direct deposit changes, garnishments |
| **Talent Management** | Performance, development plans, goal setting |
| **Workforce Administration** | Job data changes, LOA, termination processing |
| **Employee Relations** | Investigations, accommodations, ethics cases |
| **Recruiting** | Job requisitions, candidate tracking |
| **Learning & Development** | Training requests, certifications |

### COE Hierarchy
```
COE
 └── Topic Category (high-level grouping)
      └── Topic Detail (specific request type)
           └── HR Service (what the employee actually requests)
```

**Example:**
```
Benefits (COE)
 └── Health Insurance (Topic Category)
      └── Add Dependent (Topic Detail)
           └── Add Dependent to Medical Plan (HR Service)
```

### Why COEs Matter for Routing
The COE determines which assignment group sees the case. Each COE has its own:
- Assignment groups
- SLA policies
- Case templates
- Knowledge Base articles
- Security policies (who can see what)

---

## 4. HR Case Management

### HR Case Table
- Base table: `sn_hr_core_case`
- All HR cases extend this base table
- Each COE can have its own extended case table

### HR Case States
| State | Meaning |
|-------|---------|
| **Draft** | Case being created, not yet submitted |
| **Ready** | Submitted and waiting for assignment |
| **Work in Progress** | Actively being worked |
| **Awaiting Approval** | Pending HR manager or VP sign-off |
| **Awaiting Acceptance** | Waiting for employee to accept/acknowledge |
| **Suspended** | On hold (waiting for external input) |
| **Closed Complete** | Resolved successfully |
| **Closed Incomplete** | Closed without resolution |
| **Cancelled** | Cancelled before resolution |

### Case vs. Task
- **HR Case**: The parent record — tracks the overall request
- **HR Task**: Child records — individual work items within a case (especially in Lifecycle Events)

---

## 5. HR Roles

| Role | What They Can Do |
|------|-----------------|
| `sn_hr_core.admin` | Full HR admin — configure, manage all cases, override security |
| `sn_hr_core.manager` | View all cases in their COE, run reports, manage queues |
| `sn_hr_core.case_writer` | Create and update cases assigned to their group |
| `sn_hr_core.case_reader` | Read-only access to cases |
| `sn_hr_core.basic` | Submit HR requests as an employee (self-service) |

> **Important:** HR cases have strict security. An HR case_writer in Benefits cannot see Payroll cases even if on the same platform — COE-level isolation is enforced by default.

---

## 6. HR Services

An **HR Service** is the specific offering employees can request. Each HR Service is linked to:
- A COE (for routing)
- A Topic Detail (for categorization)
- An HR Service template (for auto-populating case fields)
- Fulfillment instructions (what the HR agent does to fulfill it)

**HR Service vs. Service Catalog Item:**
- HR Services are the native HRSD construct — simpler to create, tied to HR Case creation
- Service Catalog Items can also create HR Cases (for complex multi-step requests)
- Best practice: use HR Services for simple requests; Service Catalog Items for multi-step approvals

---

## 7. HR Criteria vs. User Criteria

This is one of the most frequently tested distinctions:

| | HR Criteria | User Criteria |
|--|-------------|---------------|
| **Where it evaluates** | HR Profile fields | Standard user/group fields |
| **Example fields** | Location, Department, Job Code, Employment Type, HR Profile attributes | Role, Group, Company, Department (user table) |
| **Use case** | Show/hide HR Services based on employment type (e.g., only show "Parental Leave" to full-time employees) | General visibility (e.g., show a catalog item to everyone in a group) |
| **Platform** | HR-scoped — available within HRSD module | Global — used across all ServiceNow modules |
| **Table** | `sn_hr_core_criteria` | `user_criteria` |

**When to use HR Criteria:** Any time visibility depends on employment status, job function, HR-specific attributes, or data from the HR Profile record.

---

## 8. Lifecycle Events

### What Are Lifecycle Events?
A **Lifecycle Event** is a bundled set of tasks (HR tasks, Service Catalog tasks, approvals) triggered automatically when a significant HR event occurs — onboarding, offboarding, transfer, promotion, LOA.

The power: one event trigger coordinates work across multiple teams (IT, Facilities, HR, Finance, Security) automatically.

### Key Lifecycle Event Types
| Event | Trigger | Common Tasks |
|-------|---------|--------------|
| **Onboarding** | Hire date | Create accounts, provision laptop, assign badge, assign training |
| **Offboarding** | Termination date | Revoke access, recover assets, process final pay, exit interview |
| **Transfer** | Job data change | Update access/location, reassign parking, notify new manager |
| **Leave of Absence** | LOA start | Suspend benefits, update payroll, set auto-responder |
| **Return from LOA** | Return date | Restore access, re-enroll benefits |
| **Promotion** | Grade change | Update salary, access rights, notifications |

### Lifecycle Event Architecture
```
Lifecycle Event Definition
  └── Activity Sets (logical phases — e.g., "Pre-Start", "Day 1", "First 30 Days")
       └── Activities (individual tasks — HR Task, Catalog Task, Approval)
            └── Activity Configuration (assignee, timing, dependencies)
```

### Key Concepts in Lifecycle Events
- **Activity Sets**: Group activities that happen in the same phase/time window
- **Offset**: When the activity fires relative to the event date (e.g., −7 days before start date)
- **Conditions on Activities**: Activities only fire if criteria are met (e.g., only create laptop request for US employees)
- **Dependent Tasks**: An activity can be blocked until another activity completes
- **Auto-close**: Tasks auto-close on a schedule if not completed (configurable)

---

## 9. Employee Document Management (EDM)

Manages sensitive HR documents (offer letters, I-9s, performance reviews) with:
- **Column-level encryption**: Documents encrypted by default
- **Role-based access**: Only authorized HR roles can view/download
- **Document Templates**: Auto-populated offer letters, standard forms
- **Retention Policies**: Documents retained/purged per legal requirements
- **Edge Encryption**: For highest security — encrypted before leaving the server

---

## 10. Employee Center (ESC)

### What Is It?
The **Employee Center** is the self-service portal where employees submit HR requests, check case status, read Knowledge Base articles, and complete onboarding activities.

- Built on the ServiceNow portal framework
- Replaces the older "Employee Service Center" and "HR Service Portal"
- Configurable branding, topic pages, and personalized tiles

### Employee Center vs. Employee Center Pro
| Feature | Employee Center | Employee Center Pro |
|---------|----------------|---------------------|
| Knowledge Base | ✓ | ✓ |
| Case submission | ✓ | ✓ |
| Personalized content | Basic | Advanced (employee profile-based) |
| Journey Maps | — | ✓ |
| Campaigns | — | ✓ |
| Analytics | Basic | Advanced |
| Pricing | Included in HRSD | Additional license |

---

## 11. Employee Journey Management (EJM)

### What Is It?
**Employee Journey Management** is the personalization and orchestration layer on top of Lifecycle Events. Where Lifecycle Events coordinate the work, EJM coordinates the employee experience.

Key components:
- **Journey Templates**: Reusable onboarding/offboarding experience blueprints
- **Experience Packs**: Industry or persona-specific content bundles (Healthcare Onboarding Pack, Tech Onboarding Pack)
- **Campaigns**: Proactive HR communications and check-ins (e.g., "30-day check-in" survey)
- **Journey Maps**: Visual view of the employee's experience milestones
- **Moments**: Point-in-time nudges delivered through the Employee Center

---

## 12. Case Security

### Restricted Caller Access (RCA)
When RCA is enabled on a COE, only HR agents (not employees themselves) can directly create cases in that COE. Employees must go through the Employee Center self-service flow — this prevents direct table manipulation and enforces routing rules.

### COE Security Policy
Each COE can define:
- Who can view cases (sn_hr_core.case_reader)
- Who can update cases (sn_hr_core.case_writer)
- Data classification (whether cases are visible to ITIL roles)

### ACLs and Role-Based Visibility
- HR data is isolated from ITSM by default — an ITIL role does NOT see HR cases
- Field-level visibility can restrict sensitive fields (e.g., salary) even within HR

---

## 13. Key Tables

| Table | Label | Purpose |
|-------|-------|---------|
| `sn_hr_core_case` | HR Case | Base HR case table |
| `sn_hr_core_task` | HR Task | Child tasks of HR cases |
| `sn_hr_core_coe` | HR Center of Excellence | COE definitions |
| `sn_hr_core_topic_category` | Topic Category | First level of categorization |
| `sn_hr_core_topic_detail` | Topic Detail | Second level of categorization |
| `sn_hr_core_service` | HR Service | Service offerings employees can request |
| `sn_hr_core_profile` | HR Profile | Employee HR profile (beyond sys_user) |
| `sn_hr_core_criteria` | HR Criteria | Visibility rules based on HR attributes |
| `sn_hr_core_lifecycle_event` | Lifecycle Event | Event definitions |
| `sn_hr_core_activity` | Lifecycle Activity | Individual tasks within an event |
| `sn_hr_core_document` | HR Document | Encrypted HR documents |
| `sn_hr_core_template` | HR Case Template | Pre-populated case templates |

---

## 14. Virtual Agent for HRSD

The Virtual Agent (chatbot) integrates with HRSD to:
- Answer common HR questions using KB articles
- Create HR cases without logging into the portal
- Check case status ("What's the status of my PTO request?")
- Route complex questions to an HR agent via live chat handoff

**Common HRSD Virtual Agent topics:**
- PTO balance inquiry
- Benefits enrollment question
- Direct deposit change
- Leave of absence request
- Org chart lookup

---

## 15. Now Assist for HRSD (AI)

ServiceNow's generative AI capabilities applied to HR:
- **Case Summarization**: HR agent opens a case and gets a one-paragraph summary of history
- **Resolution Note Generation**: AI drafts the resolution note based on the case history
- **KB Article Suggestions**: Similar cases surface relevant KB articles to the HR agent
- **Employee Deflection**: Now Assist in Employee Center suggests answers before a case is created

---

## 16. Integrations

| Integration | Purpose |
|------------|---------|
| **Workday** | Bidirectional sync — hire events in Workday trigger lifecycle events in ServiceNow |
| **SAP SuccessFactors** | HR Master data sync |
| **ADP** | Payroll data exchange |
| **Okta / Azure AD** | Account provisioning/deprovisioning via lifecycle events |
| **ITAM** | Laptop provisioning tasks in onboarding; asset reclaim in offboarding |
| **ITSM** | IT access provisioning tasks within lifecycle events |
| **Security Operations** | Badge/access revocation on offboarding |
| **DocuSign** | eSignature integration for offer letters, NDAs |
| **Microsoft Teams / Slack** | Notifications to employees and HR agents |

---

## 17. Glossary

| Term | Definition |
|------|-----------|
| **COE** | Center of Excellence — functional HR grouping that routes cases |
| **Topic Category** | First-level categorization under a COE |
| **Topic Detail** | Second-level categorization — specific request type |
| **HR Service** | The actual offering an employee selects |
| **HR Criteria** | Visibility rules using HR Profile attributes |
| **HR Profile** | Extended employee record with HR-specific fields |
| **Lifecycle Event** | Automated multi-team task bundle for HR events |
| **Activity Set** | Phase grouping within a lifecycle event |
| **Employee Center** | Self-service HR portal for employees |
| **EJM** | Employee Journey Management — personalization layer |
| **RCA** | Restricted Caller Access — forces case creation through portal |
| **EDM** | Employee Document Management — secure document storage |
| **Now Assist** | Generative AI features in ServiceNow |
| **Bulk Case Creation** | Create one case per employee from a list (LOA, policy updates) |
| **Case Template** | Pre-populated case fields to speed up HR agent data entry |
