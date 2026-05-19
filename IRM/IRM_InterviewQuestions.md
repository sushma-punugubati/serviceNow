# ServiceNow IRM — Interview Questions & Answers

---

## PART 1: Core Concepts

**Q1: What is ServiceNow IRM and how does it relate to GRC?**
**A:** IRM (Integrated Risk Management) IS ServiceNow's GRC (Governance, Risk & Compliance) product. IRM is the modern name — it was rebranded from "GRC" to reflect a more integrated, enterprise-wide approach. Both terms are used interchangeably in the field.

IRM is a unified platform covering: governance, risk management, compliance, audits, third-party risk, policy management, and operational resilience.

---

**Q2: What is the difference between Inherent Risk and Residual Risk?**
**A:**
- **Inherent Risk:** The risk level WITHOUT any controls — what is the raw, unmitigated exposure?
- **Residual Risk:** The risk level AFTER controls are applied — what is the remaining exposure?

**Why it matters:** The gap shows control effectiveness. If inherent risk is CRITICAL and residual risk is still HIGH, your controls aren't strong enough. If residual risk = LOW, controls are working.

**Formula:** Residual Risk = Inherent Risk − Control Effectiveness

---

**Q3: What is the difference between Risk Appetite and Risk Tolerance?**
**A:**
- **Risk Appetite:** The broad statement of how much risk the organization is willing to accept in pursuit of its objectives. Strategic in nature. Example: "We accept moderate operational risk to enable innovation."
- **Risk Tolerance:** Specific, measurable thresholds that trigger action when crossed. Example: "If residual risk score exceeds 15, escalate to the Risk Committee."

---

**Q4: What is "Test Once, Comply Many"?**
**A:** A foundational IRM concept where a single control is mapped to multiple regulatory requirements. When you test that control once, the test results satisfy compliance evidence for ALL mapped regulations simultaneously.

**Example:**
- Control: "Quarterly privileged access review"
- Mapped to: SOX 404 + PCI-DSS Req 7 + ISO 27001 A.9 + GDPR Art 32
- One test = four compliance requirements satisfied

**Business value:** Dramatically reduces audit burden. Instead of separate evidence collection for each regulation, one controlled process covers all.

---

**Q5: What is the difference between a Design Test and an Operating Effectiveness Test?**
**A:**
| Test Type | Question It Answers | How Done |
|-----------|-------------------|---------|
| **Design Test** | Is this control designed correctly to address the risk? | Review control documentation, process design |
| **Operating Effectiveness Test** | Is the control actually working in practice? | Sample real transactions, review system logs, conduct interviews |

Both tests are needed — a well-designed control that nobody follows is not effective.

---

**Q6: What is a Key Risk Indicator (KRI)?**
**A:** A KRI is a measurable metric that provides early warning of increasing risk — before an actual incident occurs.

**Example:**
- Metric: Number of failed login attempts to financial systems
- Threshold: If > 50 per day → alert triggered
- Action: Security team investigates potential brute force attack

**Value:** KRIs shift risk management from reactive (after the incident) to proactive (before it happens).

---

**Q7: What is an Entity in IRM?**
**A:** An Entity is an organizational partition that scopes governance data:
- Business unit, legal entity, location, department, or vendor
- Policies, risks, and controls are mapped to entities
- Enables granular reporting: "Show me compliance status for the Finance department in Germany only"
- **Entity Mapping** = the act of linking governance objects to specific entities

---

**Q8: What is a Policy Attestation?**
**A:** A formal acknowledgment by an employee or manager that they have read, understood, and agreed to comply with a specific policy. Tracked in IRM to ensure accountability and provide audit evidence.

---

**Q9: What is a Control Deficiency?**
**A:** A control deficiency is when a control is not working as designed or is not operating effectively. Types:
- **Design Deficiency:** The control is not designed to adequately address the risk
- **Operating Deficiency:** The control is designed correctly but not executed properly
- **Significant Deficiency / Material Weakness:** SOX-specific severity classifications

---

**Q10: What is the difference between a Risk and an Issue in IRM?**
**A:**
- **Risk:** A potential future event that MIGHT happen and would negatively impact the org
- **Issue:** An actual problem that HAS already happened or a control failure that is confirmed

IRM manages both — risks proactively and issues reactively.

---

## PART 2: Modules Deep Dive

**Q11: How does Policy & Compliance Management work end-to-end?**
**A:**
1. **Policy created:** Owner drafts policy (e.g., Data Retention Policy)
2. **Review cycle:** Legal/compliance reviews; approvals obtained
3. **Published:** Policy goes live; attestation sent to affected employees
4. **Mapped to regulations:** Policy linked to GDPR, HIPAA, etc.
5. **Controls defined:** Specific, testable mechanisms that satisfy the policy
6. **Control testing:** Design and operating effectiveness tested; evidence collected
7. **Compliance reporting:** Dashboard shows compliance % across all policies
8. **Review cycle:** Policy reviewed annually or when regulations change
9. **Retire:** When policy is no longer needed

---

**Q12: How does Audit Management work in IRM?**
**A:**
1. **Risk-based audit planning:** Use risk scores to identify highest-risk areas for audit
2. **Audit engagement created:** Scope, auditors assigned, timeline set
3. **Evidence collection:** Auditors request documents, system reports, interviews
4. **Findings documented:** Issues and deficiencies recorded with severity ratings
5. **Findings linked to controls/policies:** Provides traceability
6. **Remediation tracking:** Issues assigned to owners with deadlines
7. **Closure verification:** Evidence of remediation reviewed before closing
8. **Trend reporting:** Recurring findings flagged for systemic issues

---

**Q13: How does Third-Party Risk Management (TPRM) work?**
**A:**
1. Vendor onboarded into IRM vendor inventory
2. Risk questionnaire sent automatically (based on vendor tier/category)
3. Vendor completes questionnaire in vendor portal
4. IRM auto-scores questionnaire responses
5. High-risk findings → remediation plan required
6. Ongoing monitoring: SLA performance, incidents involving vendor
7. Annual reassessment on a defined schedule
8. Vendor risk linked to organizational impact via CMDB services

---

**Q14: What is Operational Resilience in IRM?**
**A:** Operational Resilience ensures critical business processes can survive and recover from disruptions:
- Identify critical processes and their dependencies (CMDB-linked)
- Define **RTO** (Recovery Time Objective) and **RPO** (Recovery Point Objective) for each
- Create business continuity and disaster recovery plans
- Test plans (tabletop exercises, simulated failures)
- Capture gaps and monitor remediation
- Regulatory driver: DORA (EU financial firms), BCBS 239 (banks)

---

## PART 3: Technical & Configuration

**Q15: What is the role of CSDM in IRM?**
**A:** CSDM (Common Service Data Model) connects IRM data to business services in the CMDB:
- Risks and controls can be linked to specific business services
- Cascading risk: if Business Service X fails, which regulatory obligations are impacted?
- Operational resilience: CMDB dependency maps inform recovery planning
- SecOps + IRM: Technical vulnerabilities linked to compliance obligations

Without CSDM alignment, IRM exists in isolation — disconnected from actual business impact.

---

**Q16: How does IRM integrate with ITSM?**
**A:**
- Change requests reference risk assessments before approval
- Incidents can trigger risk events ("this outage revealed a control gap")
- Problem records can drive risk register updates ("systemic problem = risk to add")
- Security incidents feed into compliance reporting

---

**Q17: What are best practices for customizing IRM without breaking upgrades?**
**A:**
1. Use **configuration** (scoped apps, business rules) instead of modifying base tables
2. Avoid direct customizations to core IRM tables
3. Use **Scoped Applications** for modularity
4. Test all customizations in sub-production before upgrades
5. Follow "configuration over customization" principle
6. Document all changes — know what to retest after each upgrade

---

## PART 4: Scenario-Based Questions

**Q18: A new regulation (e.g., DORA) is enacted. How does IRM help the organization respond?**
**A:**
1. Add the DORA regulation to the Regulatory Change Management module
2. Use IRM's regulation library or manually map DORA requirements
3. Perform **gap analysis:** Which existing controls already satisfy DORA? Which are missing?
4. Create an implementation plan for missing controls
5. Update policy library to reference DORA
6. Map new controls to DORA requirements
7. Test controls and collect evidence
8. Report compliance posture to leadership before regulatory deadline

---

**Q19: An auditor finds that a key financial control has not been tested for 18 months. How would IRM have prevented this?**
**A:** IRM prevents this through:
- **Control Test Schedules:** Every control has a defined testing frequency (quarterly, annually)
- **Automated reminders:** IRM sends notifications when tests are due or overdue
- **Dashboard alerts:** Overdue control tests are highlighted in compliance dashboards
- **Escalation rules:** If a control test is 30+ days overdue, escalate to Control Owner and Manager
- **Audit evidence trail:** Auditors can see exactly when each control was last tested

---

**Q20: A manager asks why IRM needs to connect to the CMDB. Can't risk just be managed in spreadsheets?**
**A:** Spreadsheet-based risk management cannot:
- Show which specific systems are affected by each risk (CMDB does)
- Automatically update risk posture when infrastructure changes
- Connect a vulnerability (SecOps) to a compliance obligation (IRM)
- Show cascading impact — if Business Service X fails, which regulations are impacted?
- Scale across thousands of CIs and services

CMDB connection makes risk contextual, current, and actionable — not a static document that's outdated the moment it's saved.

---

## PART 5: Exam-Style Quick Facts

| Question | Answer |
|----------|--------|
| IRM = GRC? | Yes — same product, IRM is the current name |
| Inherent risk = | Risk without controls |
| Residual risk = | Risk after controls applied |
| Risk appetite vs tolerance | Appetite = general level; Tolerance = specific measurable threshold |
| Test Once, Comply Many | One control test satisfies multiple regulations |
| Design test question | Is the control designed correctly? |
| Operating effectiveness test | Is the control actually working? |
| KRI = | Key Risk Indicator — early warning metric |
| Entity = | Organizational partition (business unit, location, vendor) |
| Policy lifecycle = | Draft → Review → Approve → Publish → Attest → Review → Retire |
| RTO = | Recovery Time Objective |
| RPO = | Recovery Point Objective |

---

## Study Checklist
- [ ] Know: Inherent vs Residual risk clearly
- [ ] Know: Risk Appetite vs Risk Tolerance distinction
- [ ] Know: Test Once, Comply Many concept
- [ ] Know: Design Test vs Operating Effectiveness Test
- [ ] Know: KRI definition and example
- [ ] Know: Entity concept and entity mapping
- [ ] Know: Policy lifecycle stages
- [ ] Know: RTO and RPO definitions
- [ ] Know: How IRM connects to CMDB/CSDM
- [ ] Know: TPRM (vendor risk) process end-to-end
