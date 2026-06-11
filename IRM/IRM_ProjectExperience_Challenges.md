# ServiceNow IRM — Interview Notes
## "What did you do in IRM?" + Challenges Faced

---

## 1. OVERVIEW — What is IRM?

IRM (Integrated Risk Management), formerly known as GRC (Governance, Risk and Compliance), is ServiceNow's module for managing enterprise risk, compliance, and audit processes. It moves risk management from spreadsheets and disconnected email workflows into a structured, trackable, continuously monitored platform.

The core value: instead of a once-a-year compliance audit that takes 3 months and involves collecting evidence via email, IRM continuously monitors control effectiveness, tracks policy acknowledgments, manages vendor risk, and integrates with SecOps so security vulnerabilities are automatically reflected in the risk register. Risks are not just documented — they are owned, tracked, and tied to business impact.

Key modules worked on:
- Policy and Compliance Management
- Risk Management and Risk Assessment
- Audit Management
- Vendor Risk Management (VRM)
- SecOps integration for security risk visibility
- Performance Analytics for compliance reporting

---

## 2. WHAT I DID — Project Work (Scenario-Based)

### 2.1 Policy and Compliance Management

- Configured **Policy Statements** mapped to external frameworks (ISO 27001, SOC 2, NIST CSF) — each policy statement had a clear owner, effective date, and review schedule
- Set up the **Control framework**: each policy mapped to one or more controls, each control had a test procedure, evidence requirements, and a responsible team assigned as the control owner
- Configured **Policy Acknowledgment** workflows: all employees in scope for a policy (defined by HR Criteria) received an acknowledgment task via the Employee Center. Completion was tracked in IRM with automated escalation to managers for non-completion after 14 days
- Built **attestation workflows** for quarterly and annual reviews — control owners received tasks to confirm their control was still operating effectively, provide evidence, and escalate exceptions for remediation

### 2.2 Risk Assessment and Risk Register

- Designed **Risk Assessment Templates** for different risk domains: IT risk, vendor risk, project risk, operational risk — each template had a consistent scoring methodology (likelihood x impact) with domain-specific risk categories
- Configured the **Risk Register** with automated risk scoring: as new risks were identified and scored, the register automatically calculated residual risk (inherent risk adjusted for control effectiveness) and flagged risks above the defined risk appetite threshold for executive review
- Set up **Risk Acceptance workflows** for risks where the cost of remediation exceeded the risk value — documented the business justification, obtained the required approval level based on risk score, and tracked the acceptance expiration date for re-evaluation
- Built **Key Risk Indicators (KRIs)** connected to Performance Analytics data sources — when an SLA breach rate exceeded 15%, it automatically elevated the associated operational risk score in the risk register

### 2.3 Audit Management

- Configured **Audit Engagements** in IRM — each audit had a scope, timeline, audit team assignments, and a set of audit tests mapped to the relevant controls
- Set up the **evidence collection workflow**: control owners received tasks to upload evidence documents within a defined window. Evidence was stored in the IRM record with version history so auditors could track what was submitted when
- Built automated **finding creation** from failed test results: when an audit test failed, an IRM Finding record was automatically created, assigned to the control owner, and tracked through remediation to closure
- Configured the **audit report template** — findings, risk ratings, and remediation timelines were auto-populated from the IRM records, reducing the time to produce the final audit report from weeks to hours

### 2.4 Vendor Risk Management (VRM)

- Configured the **Vendor Risk Assessment** process — new vendors above a defined spend or data access threshold were required to complete a risk assessment before onboarding. The assessment was sent as a questionnaire (via the Vendor Portal) and responses were automatically scored by IRM
- Set up **periodic reassessment** schedules: high-risk vendors were reassessed annually, medium-risk vendors every 2 years. IRM automatically launched the reassessment workflow on the anniversary date
- Configured **risk tiers** for vendors: Tier 1 (critical/high data access) required full due diligence including SOC 2 report upload; Tier 2 (standard vendors) used an abbreviated questionnaire
- Built a **vendor risk dashboard** showing the current risk posture of all active vendors by tier, assessment status, and outstanding risk exceptions

### 2.5 SecOps Integration

- Integrated IRM with the **SecOps module** — security incidents and vulnerabilities flagged in SecOps automatically created or updated risk records in IRM so the risk team was not manually tracking security incidents separately from the risk register
- Configured **vulnerability data flows** from external scanners (Qualys, Tenable) feeding into SecOps, which then triggered risk record creation in IRM based on CVSS score and affected CI criticality — ensured high-CVSS vulnerabilities on business-critical systems were reflected in the risk register within hours of discovery
- Built **automated risk scoring uplift**: when a SecOps Security Incident was created against a high-criticality CI, the associated IRM risk record's likelihood score was automatically increased by 1 level pending investigation

### 2.6 Performance Analytics for Compliance Reporting

- Built **PA dashboards** for the compliance team: control effectiveness rates by domain, open findings by severity and age, policy acknowledgment completion rates, and risk heatmap by business unit
- Configured **automated PDF report generation** for the quarterly board compliance report — the report pulled live data from IRM dashboards and was generated with a single click rather than manually assembled from multiple spreadsheets
- Set up **scheduled email delivery** of weekly compliance snapshots to control owners and their managers — kept risk and compliance on the radar without requiring stakeholders to log in to ServiceNow

---

## 3. CHALLENGES FACED

### Challenge 1: Risk Assessment Template Adoption — Teams Building Their Own

**Problem:** After deploying the standard risk assessment templates, several business units began building their own assessment forms outside IRM (in Excel, SharePoint) because they felt the standard templates did not capture domain-specific risk factors relevant to their business. This created fragmented risk data that could not be aggregated at the enterprise level.

**Solution:** Facilitated working sessions with each business unit to extend the standard templates with domain-specific fields rather than replacing them. The core scoring methodology remained consistent across all templates (ensuring enterprise aggregation worked), but each template had a domain-specific section that captured relevant context. Also configured a template governance process: new custom fields required IRM admin approval to maintain consistency.

---

### Challenge 2: VRM Questionnaire Response Rate — Vendors Not Completing Assessments

**Problem:** Only 40% of vendors sent a risk assessment questionnaire completed it within the defined 30-day window. The remaining 60% required multiple follow-ups, and assessments were frequently incomplete, stalling the vendor onboarding process.

**Solution:** Simplified the Tier 2 questionnaire from 45 questions to 18 core questions — with an option for vendors to upload a completed third-party assessment (SOC 2, ISO 27001 certificate) to replace answering the questionnaire. Added automated reminders at day 7, day 14, and day 21 with escalation to the internal vendor owner on day 21. Completion rate improved to 75% within the window after these changes.

---

### Challenge 3: SecOps Integration Flooding the Risk Register

**Problem:** After connecting SecOps vulnerability data to IRM risk creation, the risk register went from 200 records to over 3,000 in the first week. Every Qualys vulnerability scan finding was creating a new IRM risk record, making the risk register unusable for governance purposes.

**Solution:** Added a filter to the integration: only vulnerabilities with CVSS score 7.0+ AND affecting CIs with a Business Criticality of "High" or "Critical" (from CMDB) created IRM risk records. Lower-severity vulnerabilities were tracked in SecOps only. Also configured deduplication: if a vulnerability already had an open IRM risk record, the integration updated the existing record rather than creating a new one. Risk register stabilized at ~150 active records — manageable for governance.

---

### Challenge 4: Control Evidence Collection — Chasing Owners at Audit Time

**Problem:** Despite evidence collection tasks being sent automatically, many control owners submitted evidence at the last minute or not at all. During the first audit cycle, 35% of evidence was submitted within 48 hours of the audit deadline, creating a review bottleneck.

**Solution:** Redesigned the evidence collection cadence: instead of a single task with a 30-day window, broke it into two steps — a "preliminary evidence" submission at day 15 (partial evidence to confirm the control was operating) and "final evidence" at day 30. Added SLA tracking to evidence collection tasks so late submissions were visible to the audit manager in real time. Also added a "continuous evidence" option for certain controls (automated test execution) that collected evidence automatically without requiring manual submission.

---

### Challenge 5: Policy Acknowledgment Fatigue

**Problem:** Employees received too many policy acknowledgment tasks — in some months, employees were asked to acknowledge 8-10 policies simultaneously, which led to click-through behavior (acknowledging without reading) and undermined the purpose of the program.

**Solution:** Implemented an acknowledgment calendar: policy acknowledgments were spread across the year with no more than 2-3 policies per quarter per employee. For high-priority policies, required a quiz component (5 questions from the policy content) rather than just a checkbox — this reduced click-through behavior and gave the compliance team a defensible record of employee understanding.

---

## 4. KEY CONCEPTS TO KNOW (Quick Reference)

| Concept | What to Say |
|---|---|
| **IRM (formerly GRC)** | ServiceNow module for enterprise risk, compliance, audit, and vendor risk management |
| **Policy Statement** | Formal statement of an organizational policy mapped to a control framework |
| **Control** | A process or mechanism that mitigates a risk — has an owner, test procedure, and evidence requirements |
| **Risk Register** | Repository of all identified risks with scores, owners, and remediation status |
| **Residual Risk** | Inherent risk adjusted for the effectiveness of controls — what risk remains after controls are applied |
| **KRI (Key Risk Indicator)** | Metric that signals when a risk is trending upward — triggers risk score updates automatically |
| **Audit Engagement** | A formal audit record in IRM with scope, timeline, tests, and findings |
| **VRM (Vendor Risk Management)** | Assessing and tracking the risk posed by third-party vendors who have access to data or systems |
| **SecOps-IRM Integration** | Security incidents and vulnerabilities in SecOps automatically create/update risk records in IRM |

---

## 5. METRICS / OUTCOMES TO QUOTE

- Control evidence collection timeline improved: late submissions reduced from **35% to 12%** after phased collection redesign
- Risk register volume controlled: reduced from **3,000+ records to ~150 actionable records** after SecOps integration filtering
- VRM questionnaire completion rate improved from **40% to 75%** within the 30-day window after simplification and reminders
- Policy acknowledgment meaningful completion improved after quiz component: **click-through rate reduced by ~60%**
- Audit report preparation time reduced from **3 weeks to 2 days** with automated evidence collection and report generation
- Vendor onboarding cycle reduced by **40%** after Tier 2 questionnaire simplification and automated reminders
