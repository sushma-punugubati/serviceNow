# ServiceNow HRSD — Complete Interview Preparation Notes

> Comprehensive study guide covering HRSD architecture, configuration, Lifecycle Events, Employee Journey Management, integrations, and real-world implementation patterns.

---

## TABLE OF CONTENTS

1. [HRSD Overview & Value](#1-hrsd-overview--value)
2. [Module Architecture](#2-module-architecture)
3. [Centers of Excellence (COE) — Deep Dive](#3-centers-of-excellence-coe--deep-dive)
4. [HR Case Management](#4-hr-case-management)
5. [HR Criteria vs. User Criteria](#5-hr-criteria-vs-user-criteria)
6. [Lifecycle Events — Complete Guide](#6-lifecycle-events--complete-guide)
7. [Employee Center & Self-Service](#7-employee-center--self-service)
8. [Employee Journey Management (EJM)](#8-employee-journey-management-ejm)
9. [Employee Document Management (EDM)](#9-employee-document-management-edm)
10. [Security Model](#10-security-model)
11. [Integrations](#11-integrations)
12. [Virtual Agent & Now Assist](#12-virtual-agent--now-assist)
13. [Reporting & Analytics](#13-reporting--analytics)
14. [Implementation Best Practices](#14-implementation-best-practices)
15. [Key Tables & Fields Reference](#15-key-tables--fields-reference)
16. [10 Most Common Interview Scenarios](#16-10-most-common-interview-scenarios)

---

## 1. HRSD Overview & Value

### What HRSD Is
ServiceNow HR Service Delivery transforms HR operations from an email-and-spreadsheet model into a structured service delivery operation. The core insight: HR has the same problem IT had before ITSM — requests arrive through untracked channels, there's no visibility into workload or SLAs, and cross-functional coordination is purely manual.

**Before HRSD (typical HR operation):**
- Employees email hr@company.com for everything
- HR team manages Excel spreadsheets of open requests
- Onboarding coordination: HR sends 12 separate emails to IT, Facilities, Payroll, Security
- No audit trail — if a legal dispute arises, HR can't prove when they did what
- SLAs? Nobody knows how long cases take

**After HRSD:**
- Every request is a case with a case number, owner, SLA, and status
- Lifecycle Events automate multi-department coordination
- Employee Center: employees self-serve for Tier 1 questions
- Full audit trail on every case action
- Dashboards show queue depth, SLA performance, case types by volume

### HRSD vs. ITSM
| Aspect | ITSM | HRSD |
|--------|------|------|
| Who uses it | IT staff + end users | HR staff + all employees |
| Core record | Incident, Change, Problem | HR Case |
| Routing unit | Assignment Group | COE → Assignment Group |
| Self-service | Service Catalog + Portal | Employee Center |
| Sensitive data | Generally not | Always — compensation, medical, personal |
| Multi-team automation | Change Advisory Board | Lifecycle Events |
| Security model | ITIL roles + ACLs | COE security policies + RCA + HR roles |

---

## 2. Module Architecture

### Plugin Stack
```
com.sn_hr_core                    ← Core (required)
com.sn_hr_core.lifecycle_events   ← Lifecycle Events
com.sn_hr_core.edm                ← Employee Document Management
com.sn_hr_core.esc                ← Employee Center
com.sn_hr_core.esc_pro            ← Employee Center Pro (additional license)
com.sn_hr_core.ejm                ← Employee Journey Management
```

### Key Architectural Concepts
- **Scoped App**: HRSD runs in a ServiceNow scoped application (`sn_hr_core`). All HRSD tables are prefixed with `sn_hr_core_`.
- **HR Profile**: Extends `sys_user` — every employee gets an HR Profile record with HR-specific fields (employment type, job code, pay grade, manager). HR Criteria evaluate against HR Profile, not sys_user.
- **Case extends base**: `sn_hr_core_case` extends `task` (like all ServiceNow task-based tables). This gives it state management, assignments, SLAs, and work notes for free.

---

## 3. Centers of Excellence (COE) — Deep Dive

### COE as the Routing Backbone
The COE is the most important routing concept in HRSD. Every HR case is associated with a COE, and the COE determines:
- Which assignment group(s) can work the case
- Which security policy applies (who can see the case)
- Which SLA policy applies
- Which KB articles and templates are associated

### Standard COE Hierarchy
```
COE (e.g., Benefits)
  ├── Topic Category (e.g., Health Insurance)
  │     ├── Topic Detail (e.g., Add Dependent)
  │     │     └── HR Service (e.g., Add Dependent to Medical Plan)
  │     └── Topic Detail (e.g., Remove Dependent)
  └── Topic Category (e.g., Retirement / 401k)
        ├── Topic Detail (e.g., Contribution Change)
        └── Topic Detail (e.g., Investment Allocation Change)
```

### Building a COE from Scratch
1. Navigate to **HR Administration > Centers of Excellence**
2. Create COE record: Name, Description, assignment group(s)
3. Create Topic Categories under the COE
4. Create Topic Details under each Topic Category
5. Create HR Services under each Topic Detail
6. Configure COE Security Policy — which roles can view/edit cases in this COE
7. Enable Restricted Caller Access (RCA) if needed

### COE Security Policy
Each COE has a Security Policy that answers:
- **Who can create cases** in this COE?
- **Who can view cases** in this COE?
- **Who can update cases** in this COE?
- **Is RCA enabled?** (employees must use the portal; can't create directly)

A case_writer in Benefits literally cannot view Payroll cases — not a permissions gap, an intentional COE isolation boundary. This is critical for sensitive HR data (compensation, investigations).

### OOTB COEs
| COE | Common HR Services |
|-----|--------------------|
| Benefits | Enroll, change plan, add dependent, COBRA |
| Payroll | Direct deposit, pay inquiry, W-2, garnishment |
| Talent Management | Performance plan, goal setting, development |
| Workforce Administration | Job change, promotion, LOA, termination |
| Employee Relations | Investigation, accommodation, ethics complaint |
| Recruiting | Job requisition, referral, interview scheduling |
| Learning & Development | Training enrollment, certification tracking |

---

## 4. HR Case Management

### HR Case vs. Other ServiceNow Cases
The HR Case (`sn_hr_core_case`) inherits from `task` but has HR-specific additions:
- **COE field**: Drives routing and security
- **HR Service**: What the employee requested
- **Subject Person**: The employee the case is about (may differ from the caller — an HR agent can open a case on behalf of an employee)
- **Caller**: Who initiated the case (could be employee, manager, or HR agent)
- **Sensitive flag**: Marks cases where extra security is needed
- **HR Profile link**: Links to the subject person's HR Profile

### HR Case States (detailed)
```
Draft
  ↓ (submitted)
Ready
  ↓ (assigned/accepted)
Work in Progress
  ↓ (waiting for approval or external input)
Awaiting Approval  ←→  Suspended
  ↓ (approved or input received)
Work in Progress
  ↓
Awaiting Acceptance  (employee must acknowledge)
  ↓ (accepted)
Closed Complete    OR    Closed Incomplete    OR    Cancelled
```

### Case Templates
Templates pre-populate case fields to speed up agent data entry and ensure consistent data:
- Standard routing information (COE, Topic Detail, assignment group)
- Priority level for the service type
- Description template ("Please provide the following information: ...")
- Fulfillment instructions for the HR agent

**Linking templates to HR Services:** Open the HR Service record → set the Case Template field. When an agent creates a case from this HR Service, the template auto-applies.

### Bulk Case Creation
Used when one HR event affects many employees simultaneously:
- Open Enrollment: create one case per employee for benefits enrollment reminder
- Policy Update: create cases for all employees to acknowledge a new policy
- Training mandate: create cases for a required training cohort

**How it works:** HR admin selects a list of employees (using a filter/group), chooses an HR Service, and triggers bulk creation. One case per employee is created. For large populations (>500), use scheduled/async processing to avoid performance issues.

---

## 5. HR Criteria vs. User Criteria

This distinction comes up constantly in HRSD interviews and implementation.

### HR Criteria
- **Table**: `sn_hr_core_criteria`
- **Evaluates against**: `sn_hr_core_profile` (HR Profile fields)
- **Used for**: Controlling which HR Services are visible on the Employee Center
- **Example fields**: `employment_type`, `work_country`, `job_code`, `pay_grade`, `hr_manager`

**Building an HR Criteria record:**
1. Navigate to HR Administration > HR Criteria
2. Create: Name, Conditions (field conditions against HR Profile)
3. Attach to HR Service: open the HR Service → set "HR Criteria" field

**Gotcha**: If the employee's HR Profile field is NULL, the HR Criteria condition cannot be evaluated. Depending on configuration, this defaults to VISIBLE (not excluded). This means incomplete HR Profile data causes services to appear for the wrong people.

### User Criteria
- **Table**: `user_criteria`
- **Evaluates against**: `sys_user`, `sys_user_has_role`, `sys_user_grmember` (roles, groups)
- **Used for**: General visibility across any ServiceNow module
- **Example conditions**: User is in group X, User has role Y, User's company is Z

### When to Use Which
| Scenario | Use |
|----------|-----|
| Show "Parental Leave" only to full-time permanent employees | HR Criteria (employment_type on HR Profile) |
| Show a catalog item only to managers | User Criteria (member of "Managers" group) |
| Show "US FMLA" only to US employees | HR Criteria (work_country on HR Profile) |
| Show a portal announcement to all employees | Neither — just set it active |
| Show "Executive Benefits" only to VP+ | HR Criteria (job_grade >= VP) |

---

## 6. Lifecycle Events — Complete Guide

### What Makes Lifecycle Events Powerful
Before Lifecycle Events, onboarding coordination looked like this: HR sends email to IT, email to Facilities, email to Security, email to Payroll — and then manually follows up on each. Each team works in their own system. HR has no visibility into what's done.

With Lifecycle Events: one trigger → all tasks created automatically in the correct teams' queues → HR has a single view of all completion statuses → SLAs enforced per activity.

### Lifecycle Event Architecture (detailed)

```
Lifecycle Event Definition
  ├── Trigger: [record insert/update condition on HR Profile or HR Case]
  ├── Subject Type: Employee / Requester / Other
  └── Activity Sets
        ├── Activity Set: "Pre-Start" (offset: -14 to -1 days)
        │     ├── Activity: "Create AD Account" [Catalog Task → IT]
        │     │     ├── Offset: -7 days
        │     │     ├── Condition: work_country = 'US'
        │     │     └── Assignment: IT Provisioning group
        │     └── Activity: "Order Laptop" [Catalog Task → IT]
        │           ├── Offset: -10 days
        │           └── Assignment: IT Hardware group
        ├── Activity Set: "Day 1"
        │     ├── Activity: "Welcome Meeting with Manager" [HR Task]
        │     └── Activity: "Badge Pickup" [HR Task → Facilities]
        └── Activity Set: "First 30 Days"
              ├── Activity: "Benefits Enrollment" [HR Task]
              └── Activity: "Complete Required Training" [Catalog Task]
```

### Activity Types
| Type | Used For | Works In |
|------|----------|---------|
| HR Task | HR-specific work | HR Workspace / HR case view |
| Catalog Task | IT, Facilities, non-HR team work | ITSM / Catalog task queue |
| Approval | Sign-off required before proceeding | Approval dashboard |
| Notification | Email/message to employee or stakeholder | No queue — fires automatically |

### Activity Offsets
- **Positive offset** (+7): fires 7 days AFTER the lifecycle event trigger date
- **Negative offset** (−7): fires 7 days BEFORE the trigger date (common for onboarding pre-work)
- **Zero offset** (0): fires same day as trigger — critical for offboarding access revocation
- **Business days vs calendar days**: Configurable per activity — always set business days for tasks involving human work

### Activity Conditions
Activities only fire when their condition is true. Common patterns:

```javascript
// Only for US employees
current.work_country == 'US'

// Only for employees receiving company laptops (not contractors)
current.u_employment_type != 'Contractor'

// Only if role requires elevated access
current.u_job_code.contains('Manager') || current.u_job_code.contains('Director')

// Only if background check completed (field must be populated by HR after clearance)
current.u_background_cleared == true
```

**Critical**: If the condition references a null field, the activity skips silently. No error. No alert. This is the most common debugging issue in lifecycle events.

### Dependent Activities
Activities can be configured to only fire after another activity completes:
- "Provision system access" should not fire until "Background Check Cleared" is complete
- "Set up manager introduction meeting" should not fire until "AD account created" is complete

Configure via the "Dependent on" field on the activity record.

### Lifecycle Event Trigger Configuration
The trigger is a condition on a table (usually `sn_hr_core_profile` or `sn_hr_core_case`):
- **Onboarding trigger**: HR Profile insert where employment_type = 'Full-Time' AND hire_date is set
- **Offboarding trigger**: HR Profile update where u_termination_date is populated
- **Transfer trigger**: HR Profile update where work_location changes

### Common Lifecycle Event Types

**Onboarding:**
```
Pre-hire (−14 to −1 days):
  - Background check initiation
  - IT account creation
  - Laptop/equipment order
  - Parking/badge setup
  
Day 1:
  - Welcome communication to employee
  - Manager orientation task
  - Badge pickup confirmation
  - Employee Center access confirmation
  
First 30 Days:
  - Benefits enrollment task (30-day window)
  - Mandatory compliance training
  - 30-day check-in with manager
  - HR direct line introduction
```

**Offboarding:**
```
Day of termination decision:
  - HRBP notification
  - HR case creation for separation documentation
  
Last day (offset: 0 or −1):
  - IT access revocation (critical: same-day or before)
  - Badge deactivation
  - Asset recovery task to manager
  - Final paycheck confirmation to Payroll
  
Post-departure:
  - Exit interview scheduling
  - Benefits COBRA notification
  - Reference check policy notification
```

### Monitoring Lifecycle Events
Best practice: build a dashboard showing:
- All active lifecycle events by type (onboarding / offboarding / transfer)
- Expected activities vs. created activities (gap = failed conditions)
- Overdue activities by activity name and assignee
- Completion rate by event type

---

## 7. Employee Center & Self-Service

### Employee Center Structure
```
Employee Center (Portal)
  ├── Home Page (personalized content, announcements)
  ├── Topic Pages (by COE: Benefits, Payroll, etc.)
  │     ├── HR Services (filterable, searchable)
  │     └── Knowledge Base Articles
  ├── My Requests (case status tracking)
  ├── My Profile (personal info, HR Profile view)
  └── Journey Map (if EJM enabled)
```

### Configuring the Employee Center
1. **Portal page configuration**: Navigate to Employee Center > Administration > Pages
2. **Topic page setup**: Each COE gets a topic page — associate COE to page
3. **HR Service display**: HR Services with portal availability enabled appear under their Topic Detail
4. **Knowledge Base integration**: KB articles tagged to a COE/Topic appear on that topic page
5. **Branding**: Logo, color scheme, company-specific welcome message
6. **Personalization**: Employee Center Pro adds profile-based personalization (employee sees content relevant to their role/location)

### Employee Center Pro (Advanced Features)
- **Personalized content tiles**: Different employees see different tiles based on HR Criteria
- **Campaign management**: HR can push targeted messages to specific employee groups
- **Advanced analytics**: Employee experience scoring, content effectiveness, deflection rates
- **Journey integration**: Journey Maps appear natively in the portal

### Employee Center vs. Service Portal
HRSD initially used the older Service Portal framework. Employee Center is the modern replacement:
- Employee Center: built for employee self-service, native HR theming, EJM integration
- Service Portal: generic, requires heavier customization to match HR use cases
- **Best practice**: Always use Employee Center for new HRSD implementations; migrate Service Portal HRSD implementations when upgrading

---

## 8. Employee Journey Management (EJM)

### EJM Conceptual Model
Lifecycle Events = **operations** (making sure work gets done)  
EJM = **experience** (how the employee feels through the process)

EJM wraps the operational lifecycle with:
- Personalized content delivery
- Proactive HR communication
- Visual progress tracking
- Analytics on employee experience quality

### EJM Components

**Journey Templates:**
- Reusable blueprints for employee journeys (onboarding, offboarding, promotion)
- Define milestones, content, campaigns
- Can be linked to Lifecycle Events (EJM and Lifecycle Events work together)

**Experience Packs:**
- Pre-built, industry-specific journey bundles from ServiceNow
- Examples: "Healthcare New Hire Experience," "Remote Employee Onboarding"
- Reduces configuration time — activate the pack, customize to your needs

**Campaigns:**
- Proactive HR communications pushed to employees at specific milestones
- Examples: "30-day check-in survey," "60-day benefits reminder," "1-year anniversary recognition"
- Target audience: defined by HR Criteria or employee group

**Journey Maps:**
- Visual timeline showing the employee their onboarding/offboarding milestones
- Employee sees: ✓ Complete | 🔄 In Progress | ⏳ Upcoming
- Reduces "where's my laptop?" emails because the employee can self-serve status

**Moments:**
- Point-in-time nudges delivered at meaningful milestones
- "Welcome to your first day! Here's what to expect..."
- "You've been here 90 days — here's your 90-day goals template"

### EJM vs. Lifecycle Events: Which Controls What?

| | Lifecycle Events | EJM |
|--|-----------------|-----|
| Creates HR Tasks | ✓ | — |
| Creates IT provisioning tasks | ✓ | — |
| Employee-facing journey view | — | ✓ |
| HR campaigns and check-ins | — | ✓ |
| Experience analytics | — | ✓ |
| Requires Lifecycle Events | — | Recommended but not required |

They are complementary. Best practice: implement Lifecycle Events for operational tasks; add EJM for employee experience layer.

---

## 9. Employee Document Management (EDM)

### What EDM Manages
- Offer letters, employment agreements
- I-9 / right-to-work documents
- Performance reviews and PIPs
- Medical accommodation letters
- Disciplinary action documentation
- Separation agreements

### Encryption Architecture

**Column-Level Encryption (default):**
- Documents encrypted at the database column level
- The encryption is transparent to authorized users — they see the document
- Unauthorized users see encrypted data (unreadable)
- Requires `Explicit Roles` to be defined per document category

**Edge Encryption:**
- Encrypts data before it leaves the employee's organization's environment
- Even ServiceNow cannot read the encrypted content
- Used for the highest-sensitivity documents (medical, legal, investigations)
- Requires additional setup — Edge Encryption appliance

### Document Categories
EDM organizes documents by category, and access to a category is controlled by:
1. Role (which roles can access this category)
2. COE association (which COE owns this category)

**Best practice:** Create one document category per document type per COE. Don't share categories across COEs.

### Retention Policies
- Define how long documents are retained per category
- After retention period: auto-archive or purge
- Required for GDPR, HIPAA, SOX compliance
- Configure under HR Administration > Document Management > Retention Policies

---

## 10. Security Model

### Layered Security in HRSD

```
Layer 1: HR Roles
  sn_hr_core.admin / manager / case_writer / case_reader / basic

Layer 2: COE Security Policy
  Each COE defines which roles can view/create/update cases in that COE

Layer 3: Restricted Caller Access (RCA)
  Prevents direct case creation — forces portal intake

Layer 4: ACLs
  Table and field level access controls (inherited from ServiceNow platform)

Layer 5: Document Encryption
  Column-level or Edge Encryption for sensitive document fields
```

### HRSD Data Isolation from ITSM
By default, ITIL-role users cannot see HR cases. This is critical — if an ITSM developer can query `sn_hr_core_case`, they'd have access to compensation, disciplinary, and medical data.

The isolation is achieved through:
- `sn_hr_core_case` table ACLs: require `sn_hr_core.case_reader` or higher
- ITIL roles do not include `sn_hr_core.*` roles
- HR workspace only accessible to HR roles

**Interview point**: "How is HR data isolated from ITSM?" → ACLs on the HR case table require HR-specific roles; ITIL roles don't have those roles by default.

### Sensitive Cases
Individual HR cases can be marked "Sensitive" (checkbox on the case). This further restricts visibility — even HR case_writers who normally see all cases in the COE may not see sensitive cases without explicit elevation.

Use for: investigations, accommodation requests, executive cases.

---

## 11. Integrations

### Workday Integration (Most Common in Interviews)

**Architecture:**
```
Workday (HCM)
  ↓ [Hire event / job data change / termination]
IntegrationHub (REST or SOAP spoke)
  ↓ [Transform Workday fields → ServiceNow HR Profile fields]
ServiceNow HR Profile (created/updated)
  ↓ [Triggers lifecycle event condition]
Lifecycle Event fires → Activities created
```

**Common failure points:**
1. Workday sends hire event but HR Profile doesn't get all required fields → lifecycle event fires without complete data → activities with conditions fail
2. Workday hire date is in the future → lifecycle event fires correctly but offset activities (−14 days from start) might have already passed
3. Integration fails silently — Workday fires the event, IntegrationHub receives it, but transform fails → no HR Profile created → no lifecycle event

**Monitoring:** Build an integration health dashboard:
- Workday events received per day
- HR Profiles created/updated per day
- Lifecycle events triggered per day
- Alert if events received but no lifecycle event fires within 1 hour

### Active Directory / Azure AD Integration (for Lifecycle Events)
When the "Create AD Account" activity fires in onboarding:
- An IntegrationHub spoke (Active Directory or Azure AD) provisions the account
- The activity can auto-close when provisioning confirms success
- Or: the activity routes to an IT team as a manual task with instructions if automation isn't set up

### ITAM Integration (Onboarding/Offboarding)
- Onboarding: "Provision Laptop" Catalog Task created in the lifecycle event → links to ITAM stock order
- Offboarding: "Recover Asset" task created → ITAM asset status updated when task complete → asset returned to stockroom

### DocuSign Integration (EDM)
- Offer letters generated from template → sent for eSignature via DocuSign
- Signed document returned → stored in EDM with signature timestamp
- Available OOTB via IntegrationHub DocuSign spoke

### Microsoft Teams / Slack
- Lifecycle event notifications sent to employee's Teams/Slack channel
- "Your onboarding starts tomorrow — here's your checklist link"
- Now Assist case updates: HR agent updates a case → notification sent to employee in Teams

---

## 12. Virtual Agent & Now Assist

### Virtual Agent for HRSD
The Virtual Agent (chatbot) deployed in the Employee Center handles:
- **Case status**: "What's the status of my benefits case?" → queries HR cases for the employee
- **Information queries**: "What's the 401k contribution limit?" → KB article response
- **Simple transactions**: "Change my direct deposit" → creates an HR case pre-populated from the conversation
- **LOA inquiry**: "How do I apply for FMLA?" → guided conversation, creates case, notifies HR

**Topic blocks** (pre-built for HRSD):
- PTO balance lookup (API to time management system)
- Benefits plan comparison
- Pay stub explanation
- Open case status
- Manager self-service (approve LOA, view team cases)

### Now Assist for HRSD (Generative AI)
Applied AI capabilities in HRSD:

**Case Summarization:** HR agent opens a case with 45 work notes from a 3-week resolution process. Now Assist generates a 3-sentence summary: "Employee requested FMLA. Medical certification received 3/15. Approved by HR VP on 3/20. Leave scheduled 4/1-6/30."

**Resolution Note Generation:** When closing a case, Now Assist generates the resolution note from the work note history. Agent reviews and edits — reduces closing time from 10 minutes to 2 minutes.

**KB Article Suggestion:** As an HR agent reads a case, Now Assist surfaces the 3 most relevant KB articles from the knowledge base that previous agents used to resolve similar cases.

**Employee Deflection:** Before an employee creates a case on the Employee Center, Now Assist evaluates their typed description and suggests KB articles that might answer their question. If the question is answered → no case created. Deflection reduces case volume by 20-40%.

---

## 13. Reporting & Analytics

### HR Manager Workspace
The HR Workspace provides:
- **Queue view**: All cases in my COE's queues by state
- **Workload view**: Cases per agent — who's overloaded, who has capacity
- **SLA view**: Cases at risk (approaching breach) and breached cases
- **Lifecycle event view**: Active lifecycle events by type, activities overdue

### Employee Center Analytics (Employee Center Pro)
- **Deflection rate**: % of portal visits that did NOT result in a case (higher = better self-service)
- **Most searched topics**: What employees look for most → expand KB in those areas
- **CSAT scores**: Satisfaction scores on closed cases
- **Time-to-close by COE**: Benchmark SLA performance across HR departments

### Reporting on Lifecycle Events
Key reports for HRSD admins:
- **Onboarding completion rate**: % of onboarding lifecycle events with all activities completed by day 1
- **Activity failure rate**: Which activities are most commonly skipped (conditions failing) → fix conditions or data
- **Days-to-completion**: How long does the average onboarding take from trigger to all activities complete?
- **Offboarding access revocation SLA**: Is access revoked before or after last day?

---

## 14. Implementation Best Practices

### Phase 1: Foundation
1. **Define your COEs first** — interview HR stakeholders to map their org structure to COEs before any configuration
2. **Map your HR Services** — catalog all HR request types; count them (most companies have 50-200 distinct request types)
3. **Data migration plan** — HR Profile data must be complete before go-live (employment type, location, job code required for HR Criteria)
4. **Security policy decisions** — which COEs need RCA? Which need cross-COE visibility (HRBPs who work across all COEs)?

### Phase 2: Lifecycle Events
1. **Interview process owners, not just HR admins** — the people who know when tasks are due are the ones doing the work (IT, Facilities, Payroll)
2. **Map dependencies explicitly** — build an activity dependency diagram before configuration
3. **Plan for edge cases** — what if a hire is entered 2 days before start date (negative offset tasks already past)? What if a termination is rescinded?
4. **Test with complete HR Profile data** — blank test profiles cause activities to silently skip; test represents reality

### Phase 3: Employee Center
1. **Get employees to test, not HR** — HR knows what's there; employees find what's missing
2. **Launch with top 20 HR Services** — not everything at once; adds then gradually
3. **KB articles before go-live** — Employee Center without content = frustration; employees click "contact HR" immediately

### Phase 4: Operations
1. **SLA policies per COE** — Benefits SLAs may differ from Payroll SLAs
2. **Escalation rules** — who gets notified when a Benefits case breaches SLA? Not a global HR admin; the Benefits team lead
3. **Monthly data quality audit** — HR Profile completeness report; catch null fields before they cause criteria failures

### Common Anti-Patterns to Avoid
| Anti-Pattern | Better Approach |
|-------------|----------------|
| One giant "HR General" COE for everything | Map COEs to actual HR team boundaries |
| Activity conditions that reference HR Profile fields not in the Workday feed | Only condition on fields that are guaranteed to be populated |
| Building custom tables for HR data that exists in HR Profile | Use and extend HR Profile before creating custom tables |
| Skipping templates — HR agents fill everything manually | Build templates for top 20 HR Services; save 10+ minutes per case |
| No testing with real employee personas | Impersonation test with employees from every population |

---

## 15. Key Tables & Fields Reference

### Core Tables
| Table | Label | Key Fields |
|-------|-------|-----------|
| `sn_hr_core_case` | HR Case | coe, hr_service, subject_person, caller, state, sensitive |
| `sn_hr_core_task` | HR Task | parent (→ case), state, assignment_group, due_date |
| `sn_hr_core_profile` | HR Profile | user (→ sys_user), employment_type, work_country, job_code, pay_grade, manager |
| `sn_hr_core_coe` | HR COE | name, assignment_group, security_policy, restricted_caller_access |
| `sn_hr_core_topic_category` | Topic Category | coe, name, active |
| `sn_hr_core_topic_detail` | Topic Detail | topic_category, name, active |
| `sn_hr_core_service` | HR Service | topic_detail, name, hr_criteria, case_template, available_via_portal |
| `sn_hr_core_criteria` | HR Criteria | name, conditions (against HR Profile) |
| `sn_hr_core_lifecycle_event` | Lifecycle Event | name, trigger_table, trigger_condition, activity_sets |
| `sn_hr_core_le_activity_set` | Activity Set | lifecycle_event, name, offset_type |
| `sn_hr_core_le_activity` | Lifecycle Activity | activity_set, activity_type, offset, condition, assignment_group |
| `sn_hr_core_document` | HR Document | subject_person, document_category, encrypted, file_attachments |
| `sn_hr_core_document_category` | Document Category | coe, allowed_roles, retention_period |
| `sn_hr_core_template` | HR Case Template | name, coe, topic_detail, default_assignment_group, description |

### Key Relationships
```
sys_user ←── sn_hr_core_profile (one-to-one: every employee has one HR Profile)
sn_hr_core_profile ←── sn_hr_core_case (subject_person: who the case is about)
sn_hr_core_coe ←── sn_hr_core_topic_category ←── sn_hr_core_topic_detail ←── sn_hr_core_service
sn_hr_core_case ──→ sn_hr_core_task (one case → many tasks)
sn_hr_core_lifecycle_event ──→ sn_hr_core_le_activity_set ──→ sn_hr_core_le_activity
```

---

## 16. 10 Most Common Interview Scenarios

### Scenario 1: Lifecycle Event Activities Not Firing
**Setup:** Onboarding lifecycle event for 3 new hires. 2 hires get all 12 activities. 1 hire gets only 4 activities.

**Root Cause:** The 1 hire's HR Profile has null values for `work_country` and `employment_type`. 8 activities have conditions that reference those fields — null evaluation = activity skips.

**Answer approach:** Check HR Profile completeness → identify which fields are null → either fix the data and re-trigger the event, or modify the conditions to handle null gracefully (add `|| current.work_country == null` to US-targeted conditions as a safe default).

---

### Scenario 2: Wrong Employees Seeing an HR Service
**Setup:** "Executive Deferred Compensation" HR Service is visible to all employees, not just VP+.

**Root Cause:** HR Criteria is not configured on the HR Service. Or, HR Criteria exists but the job_grade field is null for most employees.

**Answer approach:** Add HR Criteria with condition `job_grade >= 'VP'` → verify all VP+ employee HR Profiles have job_grade populated → test by impersonating a non-VP employee (service should be gone) and a VP employee (service should appear).

---

### Scenario 3: HR Agent Can't See Cases
**Setup:** New Benefits HR agent with `sn_hr_core.case_writer` role cannot see any cases in the Benefits queue.

**Root Cause:** Role granted but agent not added to the Benefits COE assignment group. Both are required.

**Answer approach:** Check sys_user_grmember for the agent → if missing from Benefits assignment group, add them → verify they can now see cases. Document in onboarding checklist: role + group = 2 separate steps.

---

### Scenario 4: Offboarding Access Revocation Too Late
**Setup:** Terminated employees retain system access for 2-3 days after their last day.

**Root Cause:** The offboarding lifecycle event is triggered on last day (offset 0) but the IT access revocation activity is assigned to an IT team that takes 2-3 days to work their queue.

**Answer approach:** Change the access revocation activity offset to −1 (day before last day). Integrate the activity with Active Directory via IntegrationHub so it auto-completes rather than going to a manual queue. Add an escalation rule: if not complete by last day EOD, escalate to CISO.

---

### Scenario 5: Benefits Enrollment Cases Piling Up With No Resolution
**Setup:** 40% of Benefits cases stay in "Ready" state for more than 5 days.

**Root Cause:** Cases in "Ready" state have no assignee. The Benefits team is looking at their own assigned cases but not the group queue.

**Answer approach:** Configure the Benefits COE assignment rule to auto-assign cases from the group queue based on workload (round-robin or least-loaded). Add a dashboard widget showing unassigned "Ready" cases visible to the Benefits team lead. Set an SLA on "Ready" → "Work in Progress" transition (e.g., 4 hours to assign).

---

### Scenario 6: Workday Integration — New Hires Not Getting Lifecycle Events
**Setup:** Workday fires hire events. HR Profiles are being created in ServiceNow. But onboarding lifecycle events aren't triggering.

**Root Cause:** Lifecycle event trigger condition: `hire_date != null AND employment_type = 'Full-Time'`. The Workday integration populates `hire_date` but doesn't map `employment_type`. Condition never evaluates to true.

**Answer approach:** Check the Workday field mapping in IntegrationHub → identify that `employment_type` isn't mapped → add the mapping → re-test with a test hire. Also check that the `employment_type` value in Workday matches what the condition expects (Workday might send "Regular" where ServiceNow expects "Full-Time").

---

### Scenario 7: I-9 Documents Accessible by Payroll Agents
**Setup:** A Payroll HR agent can open I-9 documents stored in EDM, which should be restricted to HR Compliance and Legal only.

**Root Cause:** The I-9 document category doesn't have role restrictions — it's accessible to all `sn_hr_core.case_reader` users.

**Answer approach:** Open the I-9 document category record → add explicit role restrictions: only `hr_compliance_agent` and `hr_legal_agent` roles → remove the generic `sn_hr_core.case_reader` role from that category → test by logging in as a Payroll agent (should see no I-9 documents).

---

### Scenario 8: Employee Center — Service Not Showing Up
**Setup:** HR has configured the "Change 401k Contribution" HR Service but employees can't find it in the Employee Center.

**Systematic check:**
1. Is the HR Service Active? → Check Active flag
2. Is "Available via portal" checked? → Common miss
3. Is the HR Service linked to a Topic Detail? → If not, it won't appear in the portal hierarchy
4. Is the Topic Detail linked to a Topic Category linked to a COE? → Full hierarchy must be complete
5. Do HR Criteria exclude this employee? → Impersonate the employee; check their HR Profile fields
6. Is the Topic page in the Employee Center configured to show this COE's services? → Check portal page configuration

---

### Scenario 9: 500 Employees Need to Acknowledge a New HR Policy
**Setup:** A new remote work policy requires all 500 employees to acknowledge receipt. How do you do this in HRSD?

**Answer:** Bulk Case Creation:
1. Create an HR Service: "Remote Work Policy Acknowledgement" under Workforce Administration COE
2. HR admin navigates to Bulk Case Creation
3. Filter: all active employees
4. Select the HR Service
5. Trigger: creates one case per employee
6. Each case template pre-populated with the policy document link and "Acknowledge" action
7. Employees receive notification → click link → open case → mark "Acknowledged" → case closes
8. HR dashboard shows % acknowledged vs. outstanding

For 500 employees, run as async to avoid performance issues.

---

### Scenario 10: Onboarding Coordination Across 6 Departments
**Setup:** A company needs onboarding to coordinate HR (paperwork, benefits), IT (laptop, accounts, VPN), Facilities (desk, badge, parking), Payroll (direct deposit setup), Security (background check), and the hiring manager (first-week plan).

**Answer:** Lifecycle Event with one Activity Set per phase:

- **Pre-hire Activities** (IT, Facilities, Security fire first):
  - Background check task → Security group [−14 days]
  - Laptop order → IT Hardware [−10 days]
  - AD account creation → IT Provisioning [conditional: background cleared, −5 days]
  - Desk assignment → Facilities [−5 days]
  - Badge setup → Facilities [−3 days]

- **Day 1 Activities**:
  - Welcome email → Notification [0 days]
  - Manager first-week plan task → Hiring Manager [0 days]
  - Badge pickup confirmation → Facilities [0 days]

- **First 30 Days**:
  - Benefits enrollment case → Benefits COE [+1 day]
  - Direct deposit setup → Payroll COE [+1 day]
  - Required training enrollment → L&D [+3 days]
  - 30-day check-in with HR → HR task [+28 days]

Total: one lifecycle event trigger in Workday → all tasks created automatically across all 6 departments.

---

## Quick Reference Cheat Sheet

```
HRSD = HR Service Delivery
COE = Center of Excellence (routing backbone)
HR Profile = employee's HR-specific record (extends sys_user)
HR Criteria = visibility rules using HR Profile fields
Lifecycle Event = multi-team task automation for HR events
Activity = individual task within a lifecycle event (HR Task, Catalog Task, Approval)
Activity Offset = when the activity fires (relative to event trigger date)
Employee Center = self-service portal for employees
EJM = Employee Journey Management (employee experience layer)
EDM = Employee Document Management (encrypted secure docs)
RCA = Restricted Caller Access (forces portal intake, blocks direct case creation)
Now Assist = GenAI features (summarization, resolution notes, deflection)

Base case table: sn_hr_core_case
HR Profile table: sn_hr_core_profile
HR Service table: sn_hr_core_service
HR Criteria table: sn_hr_core_criteria
Core plugin: com.sn_hr_core
Lifecycle Events plugin: com.sn_hr_core.lifecycle_events
```
