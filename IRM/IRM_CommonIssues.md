# ServiceNow IRM — Common Issues, Pitfalls & Troubleshooting

Issues encountered during IRM (Integrated Risk Management / GRC) implementation and ongoing support, based on ServiceNow community posts, partner experience, and real-world implementations.

---

## Issue 1: Risk Register Becomes a Risk List — No Prioritization or Action

**Category:** Process / Governance  
**Severity:** Critical — the most common IRM failure mode

**The Problem:**
After IRM implementation, the Risk Register grows to 500+ risks over 12 months. Nobody knows which risks matter most. Risk owners don't review or update their risks. The Register is treated as a compliance checkbox ("we have a Risk Register") rather than a management tool. When a risk actually materializes into an incident, it wasn't in the Register or wasn't being managed.

**Why It Happens:**
Risk identification is encouraged; risk management is not enforced. There's no process for prioritizing the risk register, escalating high residual risks, or retiring risks that are no longer relevant.

**Root Cause:**
A Risk Register without governance is just a list. IRM requires active management processes: regular risk review cycles, escalation triggers, risk retirement processes, and leadership accountability.

**Fix / Prevention:**
- Define risk review frequency per risk rating: CRITICAL = monthly, HIGH = quarterly, MEDIUM = semi-annually
- Automated reminders: IRM sends notifications to Risk Owners when review is due
- KRI dashboards: trends in KRIs surface risks that need immediate attention, even between scheduled reviews
- Risk Retirement workflow: close risks that have been below threshold for 12+ months after documented review
- Executive Risk Committee: regular meeting using IRM Risk Register as agenda — leadership must actively engage

---

## Issue 2: "Test Once, Comply Many" Not Configured — Duplicate Evidence Collection

**Category:** Configuration  
**Severity:** High — one of IRM's biggest ROI features not realized

**The Problem:**
The organization has multiple regulations (SOX, PCI-DSS, ISO 27001, GDPR). Controls are tested separately for each regulation — the same control test is performed 4 times and the evidence is collected 4 times. Compliance teams are overwhelmed. "Test Once, Comply Many" was part of the justification for buying IRM, but it was never configured.

**Why It Happens:**
"Test Once, Comply Many" requires controls to be properly mapped to multiple regulatory requirements. Without this mapping, each regulation has its own isolated control set. The mapping work is significant and often skipped.

**Root Cause:**
Control-to-regulation mapping must be configured manually or via content packs. It doesn't happen automatically by installing IRM.

**Fix / Prevention:**
- Control Mapping Workshop: document which controls satisfy which requirements across ALL regulations
- Use ServiceNow's Regulatory Content Packs (available for SOX, PCI-DSS, ISO 27001, HIPAA) — pre-built mappings
- Start with 2-3 regulations: "If we map SOX controls to PCI-DSS, how much testing can we eliminate?"
- Make control mapping a release 1 deliverable — not phase 2

---

## Issue 3: Risk Scores Not Calibrated — Everything Is CRITICAL

**Category:** Configuration  
**Severity:** High — risk prioritization breaks down

**The Problem:**
After go-live, 70% of risks in the Register are rated CRITICAL. Leadership stops trusting the ratings — "if everything is critical, nothing is critical." Risk owners rate everything high to avoid accountability for non-action. The Risk Register loses credibility.

**Why It Happens:**
Risk scoring criteria (Impact × Likelihood matrices) are not properly calibrated to the organization's actual risk appetite. Or risk owners have an incentive to rate risks high (to get resources) rather than accurately.

**Root Cause:**
Risk scoring requires organizational calibration: what does CRITICAL mean for this company? Revenue loss of $1M? $100M? Regulatory fine? Reputation damage? Without shared definitions, scores are subjective.

**Fix / Prevention:**
- Define quantitative criteria for each risk rating: "CRITICAL = potential loss > $10M or regulatory sanction"
- Risk Rating Workshop: bring risk owners together to calibrate — "is this scenario really CRITICAL or HIGH?"
- Management review: monthly review of CRITICAL risks only — owners must justify CRITICAL ratings
- Distribution check: if >30% of risks are CRITICAL, recalibrate. A healthy distribution has 5-10% CRITICAL.

---

## Issue 4: Control Testing Not Completed — Controls Appear Untested for 18+ Months

**Category:** Process / Adoption  
**Severity:** High — audit exposure

**The Problem:**
Controls have defined testing schedules (annual, quarterly) but the tests aren't being completed. The IRM system shows dozens of overdue control tests. When an external auditor asks for evidence that a control was tested, none exists for the past 18 months. This is an audit finding.

**Why It Happens:**
Control owners are assigned but haven't accepted responsibility. Testing tasks arrive via IRM notifications but are deprioritized against operational work. No one escalates missed tests until the auditor asks.

**Root Cause:**
Control testing requires time commitment from control owners who are usually operational staff (not compliance staff). Without management enforcement, tests get skipped.

**Fix / Prevention:**
- IRM escalation rules: control test overdue by 30 days → escalate to Control Owner's Manager
- Calendar blocking: schedule control testing as recurring calendar events (not just IRM notifications)
- Quarterly compliance review: include "overdue control tests" on the agenda — management visibility drives action
- Make control testing part of performance objectives for control owners

---

## Issue 5: Third-Party Risk Questionnaires Ignored by Vendors

**Category:** Process / TPRM  
**Severity:** Medium-High — TPRM program loses effectiveness

**The Problem:**
Vendor questionnaires are sent through the TPRM portal. Critical vendors take 60-90 days to respond. Some never respond. The TPRM team chases vendors manually via email. High-risk vendors have incomplete assessments because their questionnaire has been pending for 6 months.

**Why It Happens:**
Vendors have their own priorities. Without contractual obligations, service delivery performance metrics, or relationship leverage, there's no consequence for ignoring questionnaires.

**Root Cause:**
TPRM effectiveness requires vendor cooperation. Vendor cooperation requires either contractual obligation or relationship leverage — neither comes automatically from deploying IRM.

**Fix / Prevention:**
- Contract language: new vendor contracts include a clause requiring annual risk questionnaire completion within 30 days
- Procurement hold: vendors with overdue questionnaires are flagged to Procurement — renewals may be delayed
- SLA in IRM: automated escalation after 10 business days of no response — email to vendor's account manager
- Vendor Portal experience: make questionnaire completion as easy as possible — pre-filled responses from previous year, simple UI

---

## Issue 6: IRM Not Connected to CMDB — Risk Data Lacks Business Context

**Category:** Integration  
**Severity:** High — risk management is disconnected from actual IT

**The Problem:**
Risks are documented in IRM but not linked to specific business services or CIs in the CMDB. When a cybersecurity risk is identified, nobody knows which specific systems it affects. Business impact scoring is estimated manually rather than derived from CMDB data.

**Why It Happens:**
IRM and CMDB are separate modules often owned by separate teams (GRC team vs. IT operations team). Integration requires deliberate configuration and cross-team collaboration.

**Root Cause:**
IRM's Entity concept needs to be connected to CSDM Business Services in CMDB for full business impact context. Without this, IRM is a standalone compliance tool rather than an integrated risk management platform.

**Fix / Prevention:**
- Map IRM Entities to CMDB Business Services: "Finance Risk" entity → linked to "Financial Systems" Application Service in CMDB
- Risk records: add CI reference field — link risk to the specific systems that introduce or are affected by it
- SecOps integration: security incidents automatically linked to relevant risks in IRM
- CSDM alignment: follow CSDM structure in CMDB so IRM can reference clean, structured service data

---

## Issue 7: Policy Attestations Not Completed — Accountability Gap

**Category:** Process / Adoption  
**Severity:** Medium — audit evidence gap

**The Problem:**
Policies are published in IRM and attestation campaigns are launched requiring all employees to confirm they've read and understood the policy. 60 days later, 40% of employees haven't completed the attestation. Compliance reports show the policy as "not fully attested."

**Why It Happens:**
Employees don't prioritize policy reading. Attestation campaigns compete with operational work. Without management follow-up, employees delay indefinitely.

**Root Cause:**
Attestation completion requires management enforcement. IRM can send reminders but cannot force behavior change.

**Fix / Prevention:**
- Manager visibility: IRM dashboard shows each manager their team's attestation completion rate
- HR consequence: incomplete attestations after deadline reported to HR as a policy violation
- Attestation deadline: 15 business days maximum (not 60 days — urgency matters)
- Simplified attestation: keep policy summaries short; link to full policy for those who want detail — reduce friction
- Escalation: 5 days before deadline, manager receives list of non-compliant team members

---

## Issue 8: Audit Findings Not Linked to Controls — No Systemic Pattern Visibility

**Category:** Configuration / Audit Management  
**Severity:** Medium — audit program loses analytical value

**The Problem:**
Audit findings are documented in IRM but not linked to the specific controls that failed. After 3 years and 500 audit findings, nobody knows which controls are repeatedly failing. The same issues appear audit after audit because there's no systemic analysis connecting findings to root causes.

**Why It Happens:**
Linking findings to controls requires discipline from auditors — it's additional work during fieldwork. Without training and process enforcement, auditors just document the finding without the control linkage.

**Root Cause:**
Audit findings and control deficiencies must be explicitly linked in IRM for trend analysis to work. This linking is not automatic.

**Fix / Prevention:**
- Make control linkage mandatory in the audit finding form — auditors cannot close a finding without selecting the related control
- Quarterly audit committee: review "controls with most findings" — identify systemic weaknesses
- Control deficiency workflow: finding + control linkage automatically creates a Control Deficiency record for the control owner
- Annual pattern analysis: report showing top 10 repeatedly failing controls — input to control redesign program

---

## Issue 9: KRIs Not Monitored — Early Warning System Inactive

**Category:** Configuration / Process  
**Severity:** High — KRI value not realized

**The Problem:**
KRIs are defined in IRM (e.g., "Number of failed privileged access attempts per day") but nobody is monitoring the dashboard. KRI thresholds are breached without anyone noticing. The early warning system is configured but not consuming.

**Why It Happens:**
KRIs require both configuration (thresholds, data feed) AND a consumption process (who reviews the dashboard? Who gets alerted?). Often the technical setup is done but the process isn't.

**Root Cause:**
KRIs are passive data without a defined review cadence and alert recipient process.

**Fix / Prevention:**
- Automated alerts: when a KRI breaches threshold, automatically notify the Risk Owner and their manager
- Weekly KRI review: include KRI dashboard in weekly security/risk operations meeting
- KRI-to-Risk linkage: when a KRI breaches, automatically flag the linked risk for immediate review
- Reduce noise: too many KRIs = nobody monitors any. Start with 5-10 high-impact KRIs and refine

---

## Issue 10: Regulatory Content Packs Not Used — Manual Mapping Required

**Category:** Configuration  
**Severity:** Medium — significant implementation effort avoidable

**The Problem:**
The implementation team manually enters all regulatory requirements and maps controls without using ServiceNow's Regulatory Content Packs. This takes 3-4 months and introduces mapping errors. Customers who purchased content packs for SOX or PCI-DSS didn't activate them.

**Why It Happens:**
Content Packs are a separate purchase or activation step. Implementation teams sometimes aren't aware they exist or that the customer has access.

**Root Cause:**
Regulatory Content Packs are pre-built, tested, and maintained by ServiceNow and partners. Customers who manually map regulations are doing unnecessary work.

**Fix / Prevention:**
- At project start: inventory what Regulatory Content Packs are included in the customer's IRM license
- Activate content packs FIRST, then customize — don't build from scratch
- Available packs: SOX, PCI-DSS, ISO 27001, HIPAA, GDPR, NIST CSF, DORA, and more
- Gap approach: use the content pack as the baseline; add custom requirements on top

---

## Quick Troubleshooting Reference

| Symptom | Likely Cause | First Step |
|---------|-------------|-----------|
| Risk Register has 500+ unreviewed risks | No review cycle governance | Define review frequency by risk rating; assign owners |
| 70% of risks rated CRITICAL | Uncalibrated scoring criteria | Run calibration workshop; define quantitative thresholds |
| Control tests months overdue | No escalation; no management follow-up | Configure 30-day escalation to manager; add to QBR agenda |
| TPRM questionnaires ignored by vendors | No contractual obligation | Update contracts; configure procurement holds for non-compliance |
| Attestation completion below 80% | No management enforcement | Show manager completion rates; add HR consequence |
| Audit findings repeat year after year | Findings not linked to controls | Make control linkage mandatory; quarterly pattern analysis |
| KRI thresholds breached unnoticed | No alert recipients configured | Add automated notification; include in weekly risk meeting |

---

## Sources and References

- ServiceNow Community: community.servicenow.com — IRM, GRC, Risk Management, Audit Management forums
- ServiceNow Docs: IRM Implementation Guide, Regulatory Content Pack documentation
- ServiceNow Knowledge Conference: "GRC/IRM Best Practices" sessions
- ISACA publications: GRC process design best practices
- LinkedIn: ServiceNow GRC practitioners sharing implementation post-mortems
- ServiceNow Now Create IRM methodology documentation
