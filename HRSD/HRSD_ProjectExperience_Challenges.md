# ServiceNow HRSD — Interview Notes
## "What did you do in HRSD?" + Challenges Faced

---

## 1. OVERVIEW — What is HRSD?

HRSD (HR Service Delivery) applies ITSM principles to Human Resources. It is **not** a replacement for an HRIS (like Workday or SAP SuccessFactors) — it sits on top and manages **how** employees interact with HR services. The core goal: one portal, one system of record for all HR interactions from hire to retire.

Key components worked on:
- Employee Center (self-service portal)
- HR Case & Knowledge Management
- Lifecycle Events (Onboarding / Offboarding / Transfers)
- HR Services and HR Criteria
- COE (Center of Excellence) setup
- Document Management & Templates
- Integrations (Workday, Active Directory, ITSM)
- Virtual Agent / Now Assist AI
- Role-based access / ACL design

---

## 2. WHAT I DID — Project Work (Scenario-Based)

### 2.1 Employee Center Setup

- Configured the **Employee Center** as the single self-service portal for employees to submit HR requests, browse the knowledge base, check case status, and receive targeted content based on their department, location, and employment type
- Set up **topic categories and subtopics** (Benefits, Payroll, Onboarding, Leaves, Company Policies) mapped to the correct HR Services
- Configured **user criteria** and **HR criteria** to control which employees see which services — for example, country-specific leave policies visible only to employees in that region
- Built **knowledge articles** for common queries (how to update personal info, how to apply for leave, payroll cut-off dates) — reduced tier-1 HR case volume by deflecting self-service answers

> **Key distinction to know:** HR Criteria uses HR-specific fields (department, location, employment type, business unit) to control access to HR content and services. User Criteria is more general and can be created from HR Criteria when the same conditions are needed outside of HRSD.

---

### 2.2 HR Case Management

- Defined **HR Services** (the case templates) with required fields, assignment rules, SLA configurations, and response templates for each COE (Benefits, Payroll, Talent, Employee Relations)
- Configured **Case Types** with appropriate security — sensitive cases (workplace investigations, disciplinary actions) were restricted using HR Criteria so only the assigned COE agents could view case details
- Set up **assignment rules** to auto-route cases to the correct COE queue based on the HR Service selected and the employee's profile (location, business unit, employment type)
- Built **SLA definitions** — first response within 2 business hours for P1 HR cases, full resolution within 3 business days for standard requests
- Configured **Agent Workspace** for HR agents — contextual sidebars showing employee profile, past cases, open tasks, and related knowledge articles in one view
- Wrote **Business Rules** and **Client Scripts** for field validation, auto-population of fields (manager, department, location pulled from the employee record on case creation), and conditional UI behavior

---

### 2.3 Lifecycle Events — Onboarding & Offboarding

- Designed and configured **Lifecycle Events** for:
  - **New hire onboarding**: triggered when offer accepted in Workday → created tasks across HR (paperwork, policy acknowledgment), IT (laptop provisioning, account creation, system access), Facilities (badge, desk assignment), and Payroll (bank details, tax setup)
  - **Offboarding**: triggered on termination date → tasks for revoking 22+ system access groups via REST call to Azure AD, equipment retrieval, final paycheck review, exit interview scheduling
  - **Internal transfer**: triggered on job change in Workday → updated cost center, manager, location, triggered new access grants and revoked old ones
  - **Leave of Absence**: triggered on approved LOA → suspended system access, notified payroll, updated manager dashboard
- Built **Activity Sets** with proper sequencing — some tasks ran in parallel (IT + Facilities), others were sequential (HR paperwork must complete before IT provisioning)
- Used **Journey Accelerators** to guide employees through multi-step life events (parental leave, relocation, return-to-work)

> **Tip for interviews:** Always mention that Lifecycle Events coordinate across multiple departments in one workflow — that's the key differentiator vs. ITSM service catalog items.

---

### 2.4 Document Management

- Created **HR Document Templates** using PDF/Word templates with merge fields — generated offer letters, employment verification letters, and separation agreements directly from the HR Case
- Configured **document security** — generated documents stored in the HR Case related list, visible only to the assigned HR agent and the subject employee
- Set up **e-signature integration** (via DocuSign REST spoke) for offer letters and separation agreements — document status tracked back into the HR Case

---

### 2.5 Integrations (REST & Spoke-Based)

- **Workday → ServiceNow (inbound)**: configured the Workday Spoke via IntegrationHub — employee hire, termination, and job change events in Workday triggered Lifecycle Events in HRSD. Used scheduled polling (nightly sync) for profile updates and real-time webhooks for hire/term events
- **ServiceNow → Active Directory (outbound)**: on offboarding case closure, Flow Designer triggered REST call to Azure AD Graph API to disable account, revoke group memberships, and reset password — all automated, no manual IT ticket
- **HRSD → ITSM**: onboarding Lifecycle Event auto-created ITSM Request Items (laptop, VPN, software access) using the Universal Request framework — one case ID tracked across both HR and IT, employee saw one unified status
- **ServiceNow → DocuSign (outbound REST)**: scripted outbound REST Message to send document for e-signature, webhook from DocuSign called Scripted REST API in ServiceNow to update signature status on the HR document record

---

### 2.6 Security & ACL Design

- Designed role hierarchy: `sn_hr_core.admin` → `sn_hr_core.case_writer` → `sn_hr_core.manager` → `sn_hr_core.basic`
- Used **HR Criteria** to restrict visibility of sensitive case types (Employee Relations, Payroll) to only the relevant COE — even HR generalists in other COEs could not see these cases
- Built **data separation** for global rollout — country HR teams could only view cases for employees in their country, enforced via HR Criteria on location field

---

### 2.7 Virtual Agent / Now Assist AI

- Configured **Virtual Agent** NLU topics for common HR queries — "what is my leave balance", "how do I update my address", "when is the next payroll date" — handled ~35% of tier-1 inquiries without human agent
- Implemented **Now Assist** for HR agents: auto-summarized long case threads for agents picking up handed-off cases, generated draft response templates, and auto-populated resolution notes on closure

---

## 3. CHALLENGES FACED

### Challenge 1: HR Data Sensitivity & ACL Complexity

**Problem:** HRSD holds some of the most sensitive data in an organization — payroll, disciplinary records, medical information. Getting ACLs right was complex. Initial configuration allowed HR generalists to see Employee Relations cases they should not have had visibility into.

**Solution:** Rebuilt ACL design using HR Criteria layered with table-level ACLs. Ran a full access audit using the `sys_security_acl` table and tested every role combination before go-live. Key lesson: test with real user personas (not just admin), because admin bypasses all ACLs.

---

### Challenge 2: Workday Integration — Data Compatibility & Sync Timing

**Problem:** Workday sent employee records with different field formats (date formats, job title codes, cost center IDs) that did not match ServiceNow's data model. Also, Workday fired hire events before the employee profile was fully populated, causing incomplete onboarding cases to be created with missing fields.

**Solution:** Built a staging table (Import Set + Transform Map) as a buffer — Workday data landed in the staging table first, a scheduled script validated required fields, and only then triggered the Lifecycle Event. Added coalesce fields on Employee ID to prevent duplicate records on re-hire scenarios.

---

### Challenge 3: Lifecycle Event Sequencing & Task Dependencies

**Problem:** Some activity sets ran in the wrong order — IT was provisioning accounts before HR received the signed employment contract. This created security and compliance risks.

**Solution:** Redesigned activity sets with explicit sequencing: Group A (HR paperwork) must reach "Complete" status before Group B (IT provisioning) unlocked. Used **condition-based activity triggers** rather than purely time-based sequencing. Added automated checks so the Lifecycle Event coordinator got a notification if any activity was blocked for more than 24 hours.

---

### Challenge 4: HR Team Adoption & Resistance

**Problem:** HR agents were used to email-based case handling and informal Slack/Teams communication. They found the structured case types restrictive and often bypassed the system, sending direct emails to employees instead of using case comments.

**Solution:** Involved HR agents in the design of case templates from day one (not just the HR leadership). Reduced the number of mandatory fields — only the absolute minimum required for routing and SLA. Created short "how-to" video guides for the 5 most common case types. Ran a 2-week pilot with 5 volunteer HR agents who became internal champions.

---

### Challenge 5: Multi-Country Localization

**Problem:** The organization operated in 12 countries. Each country had different leave policies, mandatory HR documents (German works council notices, US FMLA forms, UK P45 process), and different approval chains. Building one global Lifecycle Event that served all countries without breaking any was extremely difficult.

**Solution:** Used **HR Criteria** on HR Services to show country-specific services and documents. Built a base global Lifecycle Event and used **conditional activity sets** — the Germany activity set only fired if `employee.location.country = Germany`. Maintained a country-specific configuration document reviewed by local HR and legal teams before each country go-live.

---

### Challenge 6: ITSM ↔ HRSD Handoff — SLA Ownership

**Problem:** When an onboarding Lifecycle Event created an ITSM Request for IT provisioning, both the HR case SLA and the ITSM request SLA were tracking separately. Employees saw two different status updates (one from HR, one from IT) and were confused about who to contact.

**Solution:** Implemented the **Universal Request** framework — one parent UR record tied together the HR case and the ITSM request. Employee saw a single "Your onboarding is in progress" status on the Employee Center. HR and IT both updated their respective child records. Only the HR case SLA was exposed to the employee; the ITSM SLA was internal.

---

### Challenge 7: API Responsiveness Issues with Third-Party Integrations

**Problem:** The DocuSign REST integration had intermittent timeout failures — ServiceNow would send the document for signature, DocuSign accepted it, but the callback webhook occasionally failed to update the HR Case status, leaving cases stuck in "Awaiting Signature" indefinitely.

**Solution:** Added a **scheduled job** that ran every 4 hours to check the DocuSign API for status of any HR document pending signature for more than 24 hours — if a discrepancy was found, it updated the ServiceNow record and notified the HR agent. Also implemented retry logic (3 attempts, 30-second delay) on the initial outbound REST call using Flow Designer error handling.

---

## 4. KEY CONCEPTS TO KNOW (Quick Reference)

| Concept | What to Say |
|---|---|
| **HR Criteria** | HR-specific access control using fields like department, location, employment type — controls who sees which HR Services, knowledge, and case data |
| **User Criteria** | General ServiceNow access control — can be derived from HR Criteria when same conditions needed in non-HR contexts |
| **COE** | Center of Excellence — functional HR team (Payroll, Benefits, ER, Talent) — each has its own queue, agents, and case types |
| **Lifecycle Event** | Template of coordinated tasks across multiple departments triggered by an HR event (hire, term, transfer) |
| **Activity Set** | Group of tasks within a Lifecycle Event — can be sequential or parallel |
| **Universal Request** | Framework that links an HR Case to related ITSM/FM/other module records under one parent ticket — single employee-facing status |
| **Employee Center** | The self-service portal for employees — replaces the old "HR Service Portal" |
| **Journey Accelerators** | Guided multi-step experiences for complex moments (parental leave, relocation) |
| **Document Templates** | PDF/Word templates with merge fields — generate HR documents directly from a case |
| **Now Assist for HR** | AI that summarizes cases, drafts responses, suggests next-best actions for HR agents |

---

## 5. METRICS / OUTCOMES TO QUOTE

- New-hire onboarding time reduced from **3 days of manual coordination to same-day automated** workflow
- Employee HR call volume reduced by **35%** after Employee Center + Virtual Agent deployment
- Tier-1 HR query deflection via Virtual Agent: **~35–60%** (industry benchmark)
- Offboarding access revocation: reduced from **3-day manual IT ticket** to **automated same-day** via REST to Azure AD
- A global IT firm using HRSD cut remote employee setup time by **70%** and eliminated Day 1 access failures

---

*Sources: ServiceNow Community, LMTEQ Case Study, SPOC.eu, ServiceNow Spectaculars, Essenn Associates, SNReady, Jade Global, A3Logics, SnowGeek Solutions, KPMG HRSD Leaflet 2025*
