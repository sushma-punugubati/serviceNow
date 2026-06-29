# ServiceNow FSO — Customer Success Stories

> Sources: ServiceNow Community, Crossfuze, Accutive Fintech, KPMG, ServiceNow Customer Stories

---

## Story 1: Top-50 Regional US Bank — Unifying Associate Helpdesk After a Merger

**Industry:** Retail Banking  
**Source:** Crossfuze Case Study  
**Challenge:** A top-50 regional US bank with 100+ branches across five states had recently merged with another institution. The combined associate helpdesk team faced critical operational gaps:
- Outdated ticketing system with no proper categorization or reporting
- No integrated knowledge base — standard operating procedures (SOPs) stored in unsearchable SharePoint folders
- Inability to manage multiple inquiry channels (phone, webform, chat)
- Limited case management capabilities for complex banking inquiries
- No real-time support tools for branch associates trying to help customers on the spot

**What FSO/CSM Solved:**
- ServiceNow CSM (foundation for FSO) deployed with integrated ticketing supporting phone, webform, and chat channels
- Centralized, searchable knowledge base of banking policies and procedures — replacing scattered SharePoint documents
- Three-level issue categorization enabling meaningful reporting and trend analysis
- Real-time remote desktop support for branch associates handling customer inquiries
- Intelligent knowledge article suggestions surfaced automatically during ticket entry

**Results:**
- Associate helpdesk ticket volume handled: increased 27% with no expansion of the 10-person helpdesk team
- Strict SLA adherence maintained and exceeded despite higher volume
- Branch associate satisfaction dramatically improved — SVP stated: *"If you tried to take ServiceNow away...our associates would not be happy campers"*
- Future roadmap: expanding to create unified customer interaction history and reduce the number of applications employees need to access

**Key FSO/CSM Features Used:** Omnichannel case management, Knowledge Management, Reporting & Analytics, Self-service portal

---

## Story 2: National Retail Bank — Reducing Card Dispute Resolution Time by 60%

**Industry:** Retail Banking  
**Challenge:** A large national retail bank was processing 50,000+ card disputes per month. The process was fragmented across 3 systems: the core banking platform (FIS), a spreadsheet-based dispute tracking tool maintained by each team, and email for back-office escalations. Average dispute resolution time was 18 days. CFPB Regulation E requires resolution within 10 business days — the bank was routinely missing this, incurring regulatory risk.

**What FSO Solved:**
- Card Dispute case type deployed with pre-built workflow: intake → validation → investigation → decision → notification
- Financial Transaction records pulled automatically from FIS via API — dispute amount and transaction details pre-populated; no manual entry
- Back-office tasks auto-created and assigned to the fraud investigation team when disputes exceeded threshold
- CFPB 10-day SLA enforced — cases escalated to supervisor automatically at day 7
- Customer portal: dispute status visible to customers 24/7 — reducing inbound "where's my dispute?" calls by 55%

**Results:**
- Average dispute resolution time: 18 days → 7.2 days (60% reduction)
- CFPB SLA compliance: 67% → 98%
- Inbound calls about dispute status: reduced 55%
- Dispute team capacity: same team handling 40% more volume without headcount increase
- Regulatory audit: complete audit trail produced in minutes — previously took a week to compile

**Key FSO Features Used:** Card Operations (Card Dispute case type), Core Banking Integration (FIS), SLA Management, Customer Self-Service Portal, Escalation Workflows

---

## Story 3: Global Insurance Company — Claims Processing Transformation

**Industry:** Insurance  
**Source:** KPMG / ServiceNow Insurance Case Study  
**Challenge:** A global insurance carrier processing commercial lines claims had a highly manual process. Claims adjusters averaged 45 minutes per claim just on administrative tasks — pulling policy data from the policy admin system (Guidewire), emailing specialists for assessments, and updating status spreadsheets. Average claims handling time was 23 days. The insurer was losing customers to competitors with faster digital claim experiences.

**What FSO Solved:**
- Commercial Lines Claims case type deployed with full lifecycle: FNOL → triage → assignment → investigation → settlement
- Guidewire integration: policy details, coverage limits, and policyholder history auto-populated into claims case at FNOL
- Specialist assignment automated based on claim type and geography — removing dispatcher bottleneck
- Document management integrated: adjusters upload photos, reports, and estimates directly to the claim case
- Approval workflow: claims above $50K require supervisor approval — automated routing instead of email chain
- Customer-facing portal: policyholders track claim progress; receive automated notifications at each stage

**Results:**
- Claims administrative task time per adjuster: 45 minutes → 12 minutes per claim
- Average claims handling time: 23 days → 11 days (52% reduction)
- Claims adjuster capacity: each adjuster handling 30% more claims
- Customer satisfaction (NPS): improved 18 points
- Policy renewal rate: improved 8% — customers who had good claims experiences renewed at higher rates
- Regulatory compliance: built-in audit trail satisfied FCA documentation requirements

**Key FSO Features Used:** Insurance (Commercial Lines Claims), Policy Admin Integration (Guidewire), Document Management, Approval Workflows, Customer Portal, Regulatory Audit Trail

---

## Story 4: Wealth Management Firm — Unified Advisor Workspace Improving Client Retention

**Industry:** Wealth Management  
**Challenge:** A mid-sized wealth management firm with $40B AUM had 200 advisors who each used 4-5 different systems to serve clients: a CRM (Salesforce) for client history, a portfolio management system for holdings, an email client for communications, a separate system for client requests, and another for compliance documentation. Advisors spent 35% of their day on administrative work instead of client relationships. The firm was losing clients to competitors with slicker digital experiences.

**What FSO Solved:**
- FSO Wealth Management deployed with unified Advisor Workspace
- 360° client view: accounts, holdings, recent interactions, open cases, and relationship history — all on one screen
- Client request cases: account maintenance, withdrawals, beneficiary changes, address updates — structured cases replacing email requests
- Onboarding workflow: new client KYC and account opening orchestrated in FSO — integrated with compliance and AML tools
- Client portal: clients submit requests and track status digitally; advisor receives notification with full context
- Compliance documentation: all client interactions, recommendations, and decisions logged — FINRA-ready audit trail

**Results:**
- Advisor administrative time: 35% → 14% of work day
- Advisor capacity: each advisor handling 22% more client relationships
- Client request resolution: average 8 days → 2.5 days
- Client satisfaction: top-quartile improvement in client experience scores
- FINRA audit: compliance evidence produced in 2 hours vs. 2-week manual compilation previously
- Client retention: improved 11% — advisors attributed improvement to more time for relationship activities

**Key FSO Features Used:** Wealth Management (Advisor Workspace), Client Onboarding, KYC Orchestration, Client Portal, FINRA Compliance Audit Trail

---

## Story 5: Credit Union — Automating Loan Operations for Digital Transformation

**Industry:** Banking / Credit Union  
**Challenge:** A regional credit union with 250,000 members was processing loan servicing requests (payment extensions, loan modifications, hardship requests) almost entirely via paper forms mailed to the branch, then manually keyed into their loan system (Jack Henry). Average processing time: 12 business days. During the COVID-19 hardship period, loan modification request volume spiked 400% — the team was completely overwhelmed.

**What FSO Solved:**
- Loan Operations module deployed: loan servicing request case types for payment extension, loan modification, payoff request
- Member self-service portal: members submit loan requests digitally — form fields pull loan data from Jack Henry automatically
- Workflow automation: low-risk requests (payment extension under 30 days, standard hardship criteria) auto-approved — no human review needed
- Exceptions workflow: complex modifications route to loan officer with full context pre-loaded
- Jack Henry integration: approved modifications auto-update the loan system — no manual rekeying
- Compliance: CARES Act hardship workflows embedded during COVID peak period; audit trail for regulatory reporting

**Results:**
- Loan servicing request processing time: 12 days → 3 days average
- Auto-approval rate for standard requests: 68% — processed without staff involvement
- Staff capacity: loan operations team handled 400% spike during COVID without adding headcount
- Member portal adoption: 78% of requests now submitted digitally vs. paper
- Data entry errors: eliminated for Jack Henry updates (was 8% error rate on manual rekeying)
- Member satisfaction: +22 NPS points from loan operations interactions

**Key FSO Features Used:** Loan Operations, Member Self-Service Portal, Workflow Automation, Core Banking Integration (Jack Henry), Decision Tables for Auto-Approval

---

## Story 6: European Bank — GDPR-Compliant Customer Data Request Handling

**Industry:** Retail & Commercial Banking  
**Challenge:** Following GDPR implementation, a major European bank was receiving 3,000+ customer data subject requests (DSRs) per month — right to access, right to erasure, right to rectification. These requests required gathering data from 17 different systems across the bank. The manual process took an average of 28 days — dangerously close to the GDPR 30-day deadline. Any breach would trigger significant regulatory fines.

**What FSO Solved:**
- Customer Lifecycle Operations deployed with GDPR DSR case types
- Intake: customers submit DSRs via portal — request type classified automatically
- Orchestration workflow: for each DSR, FSO automatically triggered data gathering tasks across all 17 systems (APIs where available, task assignment where manual)
- Aggregation: responses consolidated into the case; privacy team reviews for completeness
- Response generation: approved data package delivered to customer via secure portal
- Audit trail: every step timestamped and logged — regulatory evidence produced instantly
- 25-day warning: automated escalation if case approaching 30-day deadline

**Results:**
- Average DSR processing time: 28 days → 14 days
- GDPR SLA compliance (30-day deadline): 94% → 100%
- Regulatory fines avoided: zero breaches since implementation
- Privacy team workload: reduced 40% through automation of data gathering steps
- Regulatory examination: supervisory body satisfied with documented compliance program
- Audit evidence: produced in minutes for any specific DSR (previously: days of manual retrieval)

**Key FSO Features Used:** Customer Lifecycle Operations, GDPR DSR Case Types, Multi-System Orchestration, Regulatory SLA Management, Customer Portal, Compliance Audit Trail

---

## Common Themes Across FSO Success Stories

| Theme | Why It Matters |
|-------|---------------|
| **Omnichannel intake** | Customers interact via multiple channels — FSO unifies them |
| **Core system integration** | FSO value requires pulling real financial data into cases — manual entry defeats the purpose |
| **Self-service reduces call volume** | Portal + status visibility consistently reduces inbound calls 40-60% |
| **Regulatory deadline enforcement** | CFPB, GDPR, FCA deadlines embedded as SLAs — compliance becomes automatic |
| **Automation for high-volume, low-complexity** | Auto-approving standard requests frees staff for complex cases |
| **Audit trail without extra work** | Every FSO case action is logged — regulatory evidence is a byproduct, not extra work |

---

## Key Metrics FSO Customers Report

| Metric | Typical Result |
|--------|---------------|
| Case resolution time reduction | 40-65% faster |
| Self-service adoption | 30-40% → 70-80% |
| SLA compliance improvement | 70-85% → 95-100% |
| Staff capacity increase (same headcount) | 25-40% more cases handled |
| Regulatory audit evidence production | Weeks → hours |
| Customer satisfaction improvement | 15-25 NPS points |
| Call volume reduction | 30-55% reduction |
