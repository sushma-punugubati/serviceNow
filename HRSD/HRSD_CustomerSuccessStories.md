# ServiceNow HRSD — Customer Success Stories & Real-World Examples

Case studies based on ServiceNow published case studies, Knowledge conference sessions, and documented implementations.

---

## Story 1: Global Financial Services Firm — Onboarding at Scale

**Industry:** Financial Services  
**Company Size:** 80,000 employees, 40 countries  
**Challenge:**
New hire onboarding involved 47 manual steps coordinated across HR, IT, Facilities, Compliance, and Legal. HR coordinators spent 30% of their time chasing completion status from other departments. In highly regulated financial services, incomplete onboarding (missing compliance training, background check, system access) created audit findings. Average time-to-productivity for new hires was 18 days.

**What HRSD Solved:**
- Implemented Lifecycle Events for onboarding: one trigger in Workday fired 47 coordinated tasks across 6 departments simultaneously
- Activity conditions filtered tasks by country (US vs. EU vs. APAC) and employment type (full-time, contractor, intern)
- Activity offsets ensured IT access tasks fired only after background check clearance (conditional: wait for HR field = "Background Cleared")
- Employee Center gave new hires a "Journey Map" showing their onboarding progress in real time
- HR manager dashboard showed every new hire in flight, status of each activity, and overdue tasks

**Results:**
- Onboarding time-to-productivity: 18 days → 7 days
- Manual coordination emails eliminated: 92%
- Compliance completion rate on mandatory training: 64% → 99%
- HR coordinator time on onboarding admin: 30% → 8%

**Key HRSD Features Used:**
- Lifecycle Events with conditional activities
- Workday integration via IntegrationHub
- Employee Journey Management — Journey Maps
- HR manager workload dashboard

---

## Story 2: Healthcare System — HIPAA-Compliant HR Case Management

**Industry:** Healthcare  
**Company Size:** 28,000 employees (3 hospitals, 40 clinics)  
**Challenge:**
HR teams were managing employee cases via email — performance concerns, accommodations requests, disciplinary actions, benefits appeals. No audit trail existed. When legal disputes arose, HR couldn't produce documentation proving when actions were taken and who was notified. HIPAA and Joint Commission compliance required documented evidence of policy enforcement.

**What HRSD Solved:**
- Implemented Employee Relations COE with restricted access (only ER specialists and HR VPs could view ER cases)
- Enabled Employee Document Management with column-level encryption for sensitive documents (accommodation requests, medical information)
- Every case action (update, approval, communication) time-stamped and audit-logged automatically
- DocuSign integration: disciplinary documentation signed electronically, stored in EDM with immutable audit trail
- Restricted Caller Access (RCA) enforced on ER COE — employees couldn't directly create ER cases; all went through HR intake

**Results:**
- Legal hold response time (pulling documentation): 4 days → 45 minutes
- HR email for case management: reduced by 87%
- Compliance audit findings on HR documentation: 0 in the 18 months post-go-live
- Employee Relations cases visible to unauthorized HR staff: eliminated (verified by access audit)

**Key HRSD Features Used:**
- COE Security Policies (ER COE restricted to ER group only)
- Employee Document Management with encryption
- Restricted Caller Access
- DocuSign integration
- Full audit logging

---

## Story 3: Technology Company — Global LOA Management Automation

**Industry:** Technology  
**Company Size:** 12,000 employees, 22 countries  
**Challenge:**
Leave of Absence management was a nightmare — different leave laws by country (FMLA in the US, statutory sick pay in the UK, parental leave in Germany), tracked in spreadsheets by local HR admins. Employees didn't know their leave balances, couldn't track their LOA status, and HR missed reinstatement deadlines — causing compliance violations in multiple countries.

**What HRSD Solved:**
- Built LOA Lifecycle Events per country/region with country-specific activity sets
- US FMLA: activities for FMLA notice (5 business days after request), medical certification (15 days), benefit continuation, payroll notification
- EU countries: country-specific statutory notice periods, benefit rules, return-to-work protocols
- Employee Center LOA request: employee submits request → triggers correct lifecycle event based on their location (HR Criteria: work country)
- Automated HR profile updates: when LOA approved → Workday updated via integration → payroll suspended or adjusted per country rules

**Results:**
- LOA compliance violations: eliminated (previously averaging 3 per quarter)
- Reinstatement deadline breaches: 0 post-go-live (previously 12% of LOAs)
- Employee satisfaction with LOA process: 2.1/5 → 4.3/5 (CSAT survey)
- HR admin time per LOA case: 4.2 hours → 1.1 hours

**Key HRSD Features Used:**
- Country-specific Lifecycle Event definitions
- HR Criteria for country-based routing
- Workday bidirectional integration
- Employee Center self-service LOA request
- SLA policies per country (business days, not calendar days)

---

## Story 4: Retail Chain — Rapid Seasonal Onboarding

**Industry:** Retail  
**Company Size:** 45,000 permanent employees; hires 15,000 seasonal workers each holiday season  
**Challenge:**
Hiring 15,000 seasonal employees in 8 weeks required completing onboarding tasks (background check, I-9 verification, system access, uniform issuance, training) at a pace the manual process couldn't handle. HR coordinators worked 80-hour weeks. New hires frequently started without system access, causing day-1 failures and turnover.

**What HRSD Solved:**
- Built a "Seasonal Hire" onboarding lifecycle event — stripped-down version of the full onboarding event with only tasks required for seasonal workers
- Bulk hiring: when 200 offers were extended on the same day, HR triggered lifecycle events for all 200 via bulk action — 200 sets of tasks created in under 2 minutes
- Store manager tasks (uniform, locker assignment, training schedule) routed to the specific store location's assignment group using HR Profile "work location" field
- I-9 verification tasks tracked with document upload — HR can see which stores have incomplete I-9s in a single dashboard view
- Pre-boarding Employee Center access: seasonal hires could complete forms and review training materials before day 1

**Results:**
- Onboarding throughput: 15,000 seasonal hires in 8 weeks (previously required 60% more HR staff)
- Day-1 system access failures: from 23% → 2%
- I-9 compliance: 99.8% on-time completion
- HR overtime during peak hiring season: reduced by 55%

**Key HRSD Features Used:**
- Bulk Lifecycle Event triggering
- Location-based routing via HR Profile work location
- Bulk Case Creation for communications
- Employee Document Management (I-9 storage)
- Pre-boarding Employee Center access

---

## Story 5: Manufacturing Firm — Offboarding and Access Revocation

**Industry:** Manufacturing  
**Company Size:** 18,000 employees  
**Challenge:**
A departing employee retained system access for 12 days after their last day — a security audit finding. Investigation revealed IT didn't learn about terminations until HR sent a manual email — often days late. In one instance, a terminated employee accessed the ERP system after their departure. The company needed a guaranteed, immediate access revocation process.

**What HRSD Solved:**
- Offboarding lifecycle event triggered by HR on the day termination is confirmed (not the last day)
- "Immediate" activities (offset 0) fired on trigger: IT access revocation task to IT Security, badge deactivation task to Facilities
- IT access revocation integrated with Active Directory via IntegrationHub — when IT marks the task complete, AD account is disabled automatically
- Badge deactivation task integrated with physical security system — when Facilities marks complete, badge is deactivated in the access control system
- Hard stop: if "Access Revoked" activity not completed by last day, ServiceNow escalates automatically to the CISO and HR VP
- Asset recovery: laptop recovery task assigned to employee's manager with instructions and pre-printed shipping label via email

**Results:**
- Access revocation time post-termination: 12 days average → same day (within 4 hours)
- Security audit findings on access revocation: eliminated
- Asset recovery rate: 78% → 97% (within 5 business days)
- IT manually learning about terminations: eliminated — 100% triggered by HR lifecycle event

**Key HRSD Features Used:**
- Offboarding Lifecycle Event with same-day (offset 0) activities
- Active Directory integration via IntegrationHub
- Escalation rules for overdue critical activities
- ITAM integration for asset recovery tasks

---

## Story 6: Insurance Company — HR Self-Service Deflection

**Industry:** Insurance / Financial Services  
**Company Size:** 9,000 employees  
**Challenge:**
HR team of 45 agents was handling 1,800 cases per month, but 40% were simple inquiries that could be self-served: PTO balance, benefits coverage information, paycheck date questions, org chart lookups. HR agents spent half their time on Tier 1 questions that didn't require human judgment, leaving complex cases backlogged.

**What HRSD Solved:**
- Deployed Employee Center with integrated Knowledge Base — 200+ HR KB articles written and tagged by COE
- Virtual Agent configured with 25 HRSD topic blocks: PTO balance (API call to time tracking system), benefits plan details, paycheck dates, policy lookups
- Now Assist integrated into Virtual Agent: employees who ask questions outside the 25 scripted topics get AI-generated answers from KB content before being routed to HR
- Case deflection reporting: track what % of Virtual Agent conversations resolved without creating a case
- For questions Virtual Agent couldn't answer: one-click escalation to HR live chat (integrated with Workspace)

**Results:**
- Case volume: 1,800/month → 1,100/month (39% deflection)
- Time-to-answer for Tier 1 questions: 4 hours (HR response time) → instant
- Employee self-service satisfaction score: 3.8/5 → 4.6/5
- HR agent capacity freed for complex cases: ~700 cases/month — equivalent to 6 FTE hours/day
- HR headcount avoided: 3 FTEs (ROI achieved in 14 months)

**Key HRSD Features Used:**
- Employee Center with Knowledge Base
- Virtual Agent with HRSD topic blocks
- Now Assist for generative AI responses
- HR Workspace with live chat handoff
- Case deflection analytics

---

## Story 7: University System — Faculty and Staff Onboarding Differentiation

**Industry:** Higher Education  
**Company Size:** 22,000 employees (faculty, staff, adjuncts, student workers)  
**Challenge:**
Onboarding for tenured faculty required 35 steps; adjunct professors needed 8 steps; staff needed 20 steps; student workers needed 5 steps. One-size-fits-all onboarding created confusion — faculty received IT tasks irrelevant to them, student workers received benefits tasks that didn't apply. HR managed 4 different onboarding spreadsheets.

**What HRSD Solved:**
- Built 4 distinct onboarding lifecycle events mapped to employment category (HR Criteria: Employment Type field)
- Each lifecycle event contained only relevant activities for that population
- Shared activities (background check, ID card) existed in all 4 events — maintained centrally via shared activity templates
- Department-specific variants: Nursing faculty needed additional credentialing tasks; IT staff needed elevated access provisioning tasks — handled by conditions within the Staff event
- Pre-boarding portal differentiated by employee type: faculty portal vs. staff portal vs. student worker portal

**Results:**
- Onboarding irrelevant task rate: from 35% → 4%
- New hire confusion complaints: down 72%
- HR coordinator onboarding admin time: from 3.5 hours → 45 minutes per hire
- Compliance tracking for faculty credentialing: automated (previously manual follow-up)

**Key HRSD Features Used:**
- Multiple Lifecycle Event definitions per employment type
- HR Criteria for employment-category-based event selection
- Shared Activity Templates across events
- Differentiated Employee Center portal pages by persona

---

## Common Themes Across Success Stories

| Theme | What Drives Value |
|-------|-----------------|
| **Lifecycle Event automation** | Replaces manual coordination emails across departments |
| **HR Criteria targeting** | Right tasks go to right people — no irrelevant work |
| **Integration with HCM (Workday)** | Automation triggered by real HR data changes |
| **Employee Center self-service** | Deflects Tier 1 HR questions; reduces case volume |
| **Audit trail / compliance** | Every action timestamped — satisfies legal and regulatory requirements |
| **Security policies / COE isolation** | Sensitive case data stays within the right HR team |
| **Now Assist (AI)** | Deflection and efficiency — agents and employees both benefit |
