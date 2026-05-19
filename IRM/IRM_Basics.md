# ServiceNow IRM — Fundamentals & Study Guide

> IRM = Integrated Risk Management (also called GRC — Governance, Risk & Compliance)
> Note: You may see this referred to as "IRA" — the official ServiceNow module name is IRM.
>
> Sources: [IRM Interview Questions](https://servicenowspectaculars.com/servicenow-integrated-risk-management-interview-questions-2025/) | [ServiceNow GRC Community](https://www.servicenow.com/community/grc-articles/grc-integrated-risk-management-knowledge-amp-troubleshooting/ta-p/3133662)

---

## 1. What is IRM?

**Integrated Risk Management (IRM)** is ServiceNow's unified platform for managing governance, risk, compliance, audits, vendor/third-party risk, policy management, and operational resilience — all in one place.

> **Think of it as:** The system that keeps the company out of regulatory trouble, financial risk, and security exposure. Instead of tracking compliance in spreadsheets and risk in email chains, everything is connected, automated, and visible to leadership in real time.

**Key shift IRM enables:** From reactive ("we found a compliance gap during the audit") to proactive ("we detected the control weakness 6 months before the audit and fixed it").

---

## 2. IRM Core Modules

| Module | What It Does | Simple Example |
|--------|-------------|----------------|
| **Policy & Compliance** | Create policies, map to regulations, collect evidence | "Our Data Retention Policy satisfies GDPR Article 17" |
| **Risk Management** | Identify, assess, and monitor risks | "Database misconfiguration is a HIGH risk — linked to 3 controls" |
| **Audit Management** | Plan and execute audits using risk data | "Prioritize audit of Payment Processing based on high risk score" |
| **Vendor/Third-Party Risk** | Assess risk from suppliers and partners | "Vendor ABC failed security questionnaire — remediation required" |
| **Regulatory Change** | Track new laws and assess their impact | "New SEC rule impacts 12 of our controls — action plan created" |
| **Operational Resilience** | Ensure critical processes can survive disruptions | "Business Continuity Plan tested for Core Banking service" |

---

## 3. Core Concepts Explained Simply

### 3.1 Risk
A risk is a potential event that could negatively impact the organization.

**Risk has two key dimensions:**
- **Likelihood:** How probable is it that this risk materializes?
- **Impact:** How bad would it be if it did?

**Risk score = Likelihood × Impact**

### 3.2 Inherent Risk vs. Residual Risk

| Type | Definition | Example |
|------|-----------|---------|
| **Inherent Risk** | Risk level WITHOUT any controls in place | "Unprotected database = CRITICAL risk" |
| **Residual Risk** | Risk level AFTER controls are applied | "Database with encryption + access controls = LOW risk" |

> **Why it matters:** The gap between inherent and residual risk tells you whether your controls are actually working. If residual risk is still high, controls are insufficient.

### 3.3 Control
A **control** is a specific mechanism or process that reduces risk.

**Two types of control tests:**
| Test | Question | How Done |
|------|----------|---------|
| **Design Test** | Is the control designed correctly to address the risk? | Review control documentation, process design |
| **Operating Effectiveness Test** | Is the control actually working in practice? | Sample transactions, system logs, interviews |

### 3.4 Policy
A **policy** is an organization's formal statement of rules and requirements.

**Policy lifecycle:**
```
Draft → Review → Approval → Published → Attestation → Review Cycle → Retire
```

Policies are mapped to:
- **Regulations** (GDPR, SOX, HIPAA, PCI-DSS)
- **Controls** (specific mechanisms that satisfy the policy)
- **Entities** (business units or locations the policy applies to)

### 3.5 Risk Appetite vs. Risk Tolerance

| Term | Definition | Example |
|------|-----------|---------|
| **Risk Appetite** | The overall level of risk the organization will accept | "We accept moderate operational risk to enable innovation" |
| **Risk Tolerance** | Specific measurable threshold that triggers action | "If residual risk score > 15, escalate to the board" |

### 3.6 Key Risk Indicator (KRI)
A **KRI** is a metric that gives early warning when a risk is increasing — before an incident actually occurs.

**Example:** 
- KRI: "Number of failed access attempts to financial systems"
- Threshold: If > 50/day → alert raised → investigate immediately

### 3.7 Test Once, Comply Many
One control mapped to multiple regulations. Test it once → evidence covers compliance for all mapped regulations.

**Example:**
- Control: "Quarterly access review of financial systems"
- Satisfies: SOX Section 404 + PCI-DSS Requirement 7 + ISO 27001 A.9 + GDPR Article 32
- Test once → 4 compliance requirements satisfied

---

## 4. Risk Management Lifecycle

```
1. IDENTIFY    → Define risk statements (what could go wrong?)
2. ANALYZE     → Score likelihood and impact
3. PRIORITIZE  → Rank by risk score; focus resources on top risks
4. CONTROL     → Link controls that reduce each risk
5. MONITOR     → Track via KRIs and control test results
6. RESPOND     → Accept, mitigate, transfer, or avoid each risk
7. REPORT      → Executive dashboards showing risk posture
```

---

## 5. Audit Management

**Risk-based audit planning:** Use risk scores to decide WHERE to audit first.

**Audit lifecycle:**
```
Risk Assessment → Audit Planning → Scoping → Evidence Collection → 
Finding Documentation → Remediation Tracking → Closure Verification
```

**Audit findings:**
- Linked to specific controls and policies
- Have assigned owners and remediation deadlines
- Tracked until closed with verified evidence

---

## 6. Vendor / Third-Party Risk Management

Organizations face risk from their suppliers (cloud providers, outsourced services, contractors).

**TPRM Process:**
1. Maintain vendor inventory
2. Send risk questionnaires (automated in IRM)
3. Score vendor risk responses
4. Monitor SLA performance
5. Manage remediation workflows when issues found
6. Link vendor risk to organizational impact

---

## 7. Operational Resilience

Ensures critical business processes can survive major disruptions.

- Identify critical processes and their dependencies
- Define **RTO** (Recovery Time Objective) and **RPO** (Recovery Point Objective)
- Create and test business continuity plans
- Capture gaps and monitor readiness
- Integrated with CMDB to understand infrastructure dependencies

---

## 8. Key Tables in IRM

| Table Name | Label | Purpose |
|-----------|-------|---------|
| `sn_risk_risk` | Risk | Risk statement records |
| `sn_compliance_policy` | Policy | Policy records |
| `sn_compliance_control` | Control | Control records |
| `sn_audit_engagement` | Audit Engagement | Planned audit records |
| `sn_audit_finding` | Audit Finding | Findings from audits |
| `sn_compliance_test_result` | Control Test Result | Evidence from control testing |
| `sn_risk_assessment` | Risk Assessment | Formal risk assessment records |
| `sn_vendor_risk_vendor` | Vendor Risk | Vendor risk profile records |
| `sn_risk_indicator` | Risk Indicator | KRI metric records |
| `sn_compliance_issue` | Issue | Compliance exceptions and issues |
| `sn_policy_attestation` | Policy Attestation | Policy acknowledgment records |
| `sn_resilience_plan` | Resilience Plan | Business continuity plan records |

---

## 9. Entity Concept

An **Entity** is an organizational partition that scopes governance data:
- Business unit, location, legal entity, vendor
- Policies, risks, and controls are mapped to entities
- Enables: "This control applies to the Finance department in the US"
- **Entity Mapping** = linking governance objects to specific org units

---

## 10. Roles in IRM

| Role | Responsibilities |
|------|----------------|
| **Risk Owner** | Accountable for specific risk areas; monitors and accepts/mitigates |
| **Control Owner** | Ensures controls are designed and operating effectively |
| **Policy Owner** | Manages policy lifecycle from creation to retirement |
| **Auditor** | Plans and executes audits; documents findings |
| **Vendor Manager** | Oversees third-party risk assessments and remediation |
| **Compliance Officer** | Tracks regulatory compliance; manages exceptions |
| **GRC Admin** | Configures IRM; manages entities, frameworks, scoping rules |

---

## 11. Regulatory Frameworks Commonly Managed in IRM

| Framework | Industry | Focus |
|-----------|---------|-------|
| **SOX** | Public companies | Financial reporting controls |
| **GDPR** | All (EU data) | Data privacy and protection |
| **HIPAA** | Healthcare | Patient data security |
| **PCI-DSS** | Payments | Card data security |
| **ISO 27001** | All | Information security management |
| **NIST CSF** | All | Cybersecurity framework |
| **DORA** | Financial (EU) | Digital operational resilience |
| **SOC 2** | Technology | Service organization controls |

---

## 12. CSDM Alignment in IRM

IRM uses CMDB/CSDM to:
- Link risks and controls to business services in the CMDB
- Understand cascading risk (if a business service fails, what regulations are impacted?)
- Map technical vulnerabilities to compliance obligations
- Enable business impact analysis for operational resilience planning

---

## 13. Glossary

| Term | Simple Explanation |
|------|-------------------|
| **GRC** | Governance, Risk & Compliance — the practice IRM supports |
| **IRM** | Integrated Risk Management — ServiceNow's GRC module |
| **Risk** | Potential event that could negatively impact the organization |
| **Control** | Mechanism that reduces a risk |
| **Policy** | Formal organizational rule or requirement |
| **Inherent Risk** | Risk level without controls |
| **Residual Risk** | Risk level after controls are applied |
| **Risk Appetite** | Overall level of risk org will accept |
| **Risk Tolerance** | Specific threshold that triggers action |
| **KRI** | Key Risk Indicator — early warning metric |
| **Attestation** | Formal acknowledgment that a policy has been read and agreed to |
| **Entity** | Organizational unit (business unit, location, vendor) |
| **Test Once, Comply Many** | One control test satisfies multiple regulatory requirements |
| **RTO** | Recovery Time Objective — max acceptable downtime |
| **RPO** | Recovery Point Objective — max acceptable data loss window |
| **TPRM** | Third-Party Risk Management — vendor risk assessment |
| **Control Deficiency** | A control that is not working as designed or not operating effectively |
| **Exception** | Approved deviation from a policy requirement |
