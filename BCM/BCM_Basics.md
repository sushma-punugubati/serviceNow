# ServiceNow BCM — Business Continuity Management: Basics

---

## 1. What Is Business Continuity Management?

**Business Continuity Management (BCM)** is the discipline of ensuring that critical business operations can continue during and recover from disruptions — whether caused by natural disasters, cyberattacks, system failures, pandemics, or supply chain failures.

**In ServiceNow:** BCM is a key capability within the **Operational Resilience** module of **IRM (Integrated Risk Management)**. It provides a structured, digital approach to:
- Identifying which business processes are critical
- Understanding how long those processes can tolerate disruption
- Creating and maintaining plans for recovery
- Testing those plans regularly
- Tracking gaps and improvements

**The core questions BCM answers:**
1. What would break if our data center went offline?
2. How quickly do we need to recover it?
3. Do we have a documented plan to recover it?
4. Have we tested that plan recently?
5. What gaps exist, and who is fixing them?

---

## 2. Key BCM Definitions

These definitions appear constantly in interviews:

| Term | Definition |
|------|-----------|
| **BCP** | Business Continuity Plan — the playbook for maintaining operations during a disruption |
| **DRP** | Disaster Recovery Plan — specifically focused on restoring IT systems and data |
| **IT DRP** | IT Disaster Recovery Plan — recovery of IT infrastructure, servers, databases, applications |
| **BIA** | Business Impact Analysis — assessment identifying critical processes and the impact of disruption over time |
| **RTO** | Recovery Time Objective — the maximum acceptable time to restore a process/system after disruption |
| **RPO** | Recovery Point Objective — the maximum acceptable amount of data loss measured in time |
| **MTPD** | Maximum Tolerable Period of Disruption — beyond this point, the business cannot recover (worse than RTO) |
| **CBF** | Critical Business Function — a process or function whose failure would significantly impact the organization |
| **Crisis Management Plan** | Plan for managing the organizational response to a crisis event (communications, leadership decisions) |
| **Tabletop Exercise** | Discussion-based testing of a BCP without actually activating systems |
| **Full Simulation** | Live test of the BCP — actually failing over to backup systems to prove recovery works |

---

## 3. RTO vs. RPO — The Most Important BCM Concepts

These two are the most tested concepts. Learn them cold:

```
DISRUPTION EVENT OCCURS
        ↓
[←————— RPO ————→] [←———— RTO ————→]
Data lost in this    Time allowed to
window is accepted   restore operations
```

**RTO (Recovery Time Objective):**
- "How long can we be down?"
- Example: "Our payment processing system must be restored within 4 hours of failure"
- Sets the clock for IT to restore systems

**RPO (Recovery Point Objective):**
- "How much data can we afford to lose?"
- Example: "We can lose at most 1 hour of transaction data"
- Drives backup frequency decisions (hourly backups if RPO = 1 hour)

**Real-world example:**
| System | RTO | RPO |
|--------|-----|-----|
| Online Banking | 2 hours | 15 minutes |
| Internal HR system | 48 hours | 24 hours |
| Trading platform | 30 minutes | 0 (no data loss) |
| Email | 4 hours | 1 hour |

**MTPD vs. RTO:**
- RTO = what we AIM for
- MTPD = the absolute maximum before we cannot recover (MTPD > RTO always)

---

## 4. Business Impact Analysis (BIA)

The **BIA** is the foundational step in BCM. Before you can plan recovery, you must understand what you're protecting.

### BIA Process in ServiceNow
```
1. Identify business processes/functions
2. Assess impact of disruption at different time intervals (1hr, 4hr, 8hr, 24hr, 1 week)
3. Set RTO and RPO for each process
4. Identify dependencies (people, technology, suppliers)
5. Document recovery strategies for each critical function
```

### Impact Categories in BIA
- **Financial:** Revenue loss per hour of downtime
- **Regulatory:** Fines or sanctions if the process is offline
- **Reputational:** Customer trust damage
- **Operational:** Inability to serve customers or fulfill obligations
- **Legal:** Contractual breach if SLAs to customers are missed

### BIA Output
After BIA, each process is classified:
- **Tier 1 (Mission Critical):** RTO < 4 hours — financial trading, 911 dispatch, payment processing
- **Tier 2 (Business Critical):** RTO 4-24 hours — customer service, order management
- **Tier 3 (Important):** RTO 24-72 hours — HR, internal reporting
- **Tier 4 (Normal):** RTO > 72 hours — marketing, analytics, non-time-sensitive functions

---

## 5. BCM Plan Types

### Business Continuity Plan (BCP)
Focuses on maintaining BUSINESS operations during disruption:
- Alternate work locations (if office is unavailable)
- Manual workaround procedures (if systems are down)
- Staffing and communication protocols
- Customer communication plan
- Supplier contingency arrangements

### Disaster Recovery Plan (DRP) / IT DRP
Focuses on restoring IT SYSTEMS after disruption:
- System recovery sequence (what gets restored first?)
- Backup restoration procedures
- Failover to DR site procedures
- Network and connectivity restoration
- Application restart sequences and verification steps

### Crisis Management Plan
Focuses on LEADERSHIP decision-making and communication during a crisis:
- Crisis Management Team composition
- Decision authority matrix
- Internal and external communication templates
- Media/PR response protocols
- Escalation and declaration criteria

### Pandemic / Human Resources Continuity Plan
- Remote work enablement
- Staff replacement / cross-training coverage
- Supply chain alternative contacts

---

## 6. BCM Lifecycle in ServiceNow

```
PLAN → ASSESS → ANALYZE → DESIGN → IMPLEMENT → TEST → IMPROVE → (repeat)
```

| Stage | ServiceNow Activity |
|-------|-------------------|
| **Plan** | Define BCM scope, roles, regulatory requirements |
| **Assess** | Business Impact Analysis (BIA) — identify critical functions |
| **Analyze** | Identify gaps in current recovery capabilities vs. RTO/RPO targets |
| **Design** | Create recovery strategies; document BCPs and DRPs |
| **Implement** | Deploy controls, backup systems, alternate work sites |
| **Test** | Tabletop exercises, functional tests, full simulations |
| **Improve** | Track findings, remediate gaps, update plans |

---

## 7. Key Tables in ServiceNow BCM

| Table | What It Stores |
|-------|---------------|
| `sn_bcm_plan` | BCM Plan records (BCP, DRP, Crisis Management) |
| `sn_bcm_bia` | Business Impact Analysis records |
| `sn_bcm_critical_function` | Critical Business Function records |
| `sn_bcm_exercise` | Plan testing exercises (tabletop, simulation) |
| `sn_bcm_exercise_finding` | Findings and gaps discovered during exercises |
| `sn_risk_risk` | Risk records linked to BCM gaps (from IRM) |
| `cmdb_ci_service` | Business Services mapped to critical functions (from CMDB) |

---

## 8. Plan Testing in BCM

Plans that have never been tested are not reliable plans. ServiceNow BCM manages the full testing lifecycle:

### Types of Tests
| Test Type | Description | Frequency |
|-----------|------------|-----------|
| **Tabletop Exercise** | Discussion only — walk through the scenario, no systems touched | Quarterly |
| **Walkthrough Test** | Team walks through plan step-by-step, validates procedures are current | Semi-annually |
| **Functional Test** | Specific recovery procedures actually executed (e.g., restore from backup) | Annually |
| **Full Simulation** | Complete failover to DR site; actual systems used | Annually or per regulation |
| **Surprise Test** | Unannounced exercise to test real-world readiness | As required |

### Testing Process in ServiceNow
1. Create Exercise record in BCM module
2. Assign participants and set date
3. Execute exercise — record what worked and what didn't
4. Document findings (gaps, process failures, timing misses)
5. Create remediation tasks for each finding
6. Re-test findings in next exercise cycle

---

## 9. BCM and CMDB Integration

BCM without CMDB is blind — you can document plans but can't prove the technical dependencies are understood.

**How CMDB supports BCM:**
- Critical Business Functions linked to **Application Services** in CMDB
- Application Services linked to infrastructure CIs (servers, databases)
- **Dependency maps:** show exactly what technology must be recovered for each critical function
- **RTO validation:** if the infrastructure recovery time exceeds the application's RTO, the gap is visible
- **Change Management:** changes to CIs in critical service paths require BCM impact review

**Example:**
- BCM says: "Online Banking has RTO = 2 hours"
- CMDB says: "Online Banking App Service depends on DB cluster. DB cluster failover takes 45 minutes."
- BCM Gap: the 2-hour RTO is achievable IF the DB failover works as designed

---

## 10. Regulatory Requirements Driving BCM

BCM implementations are almost always regulatory-driven:

| Regulation | Who It Applies To |
|------------|-----------------|
| **ISO 22301** | International BCM standard — applicable to any organization |
| **DORA** | Digital Operational Resilience Act — EU financial services firms (banks, insurers, investment firms) |
| **FFIEC BCP Handbook** | US banks and financial institutions |
| **APRA CPS 232** | Australian banks and insurers |
| **MAS TRM** | Singapore financial institutions |
| **HIPAA Contingency Plan** | US healthcare organizations |
| **PCI-DSS Req 12** | Any organization processing payment cards |
| **NERC CIP** | North American electric utility operators |

**Key regulatory requirements:**
- Documented BCP and DRP for all critical systems
- Regular testing (at minimum annually for major regulators)
- Evidence of findings remediation
- Board/executive sign-off on BCM program

---

## 11. BCM Integration with Other ServiceNow Modules

| Integration | How It Works |
|------------|-------------|
| **IRM** | BCM plans linked to Risk Register; BCM gaps create risk records |
| **CMDB** | Critical functions mapped to CMDB service CIs and infrastructure |
| **SecOps** | Cyber incidents can trigger BCM plan activation workflows |
| **ITSM** | Major incidents can escalate to BCM crisis invocation |
| **SPM** | BCM gaps generate remediation projects tracked in SPM |
| **ITAM** | Assets supporting critical functions identified for priority recovery |
| **Change Management** | Changes to critical CI paths require BCM impact review |

---

## 12. BCM Roles in ServiceNow

| Role | Responsibilities |
|------|----------------|
| **BCM Manager** | Owns the BCM program; ensures plans are maintained and tested |
| **Business Continuity Planner** | Creates and maintains BCPs and DRPs |
| **Critical Function Owner** | Business leader responsible for a critical process; approves BIA data |
| **IT Recovery Owner** | Responsible for DRP execution; owns the technical recovery procedures |
| **Crisis Management Team** | Senior leadership who invoke and manage during a real crisis |
| **Exercise Facilitator** | Runs tabletop exercises; captures findings |

---

## 13. BCM vs. IRM — How They Work Together

BCM is a **sub-capability** of Operational Resilience within IRM:

```
IRM (Integrated Risk Management)
├── Policy & Compliance Management
├── Risk Management
├── Audit Management
├── Third-Party Risk Management
└── Operational Resilience
    ├── Business Continuity Management  ← BCM lives here
    ├── IT Disaster Recovery
    └── Crisis Management
```

**Key connections:**
- **BCM testing findings → Risk Register:** If a test reveals a BCP gap, it becomes a Risk in IRM
- **Control testing:** BCM controls (e.g., "Annual DRP test") are tested and evidenced in IRM
- **Compliance:** BCM compliance with ISO 22301 / DORA is tracked alongside other regulations in IRM

---

## 14. Glossary

| Term | Definition |
|------|-----------|
| **BIA** | Business Impact Analysis |
| **BCP** | Business Continuity Plan |
| **DRP** | Disaster Recovery Plan |
| **RTO** | Recovery Time Objective |
| **RPO** | Recovery Point Objective |
| **MTPD** | Maximum Tolerable Period of Disruption |
| **CBF** | Critical Business Function |
| **Tabletop** | Discussion-based plan test |
| **Failover** | Switching operations to backup system |
| **Warm Site** | Backup facility with systems pre-installed but not running |
| **Hot Site** | Fully operational backup facility ready for immediate switchover |
| **Cold Site** | Backup facility with infrastructure but no systems pre-installed |
| **ISO 22301** | International standard for BCM management systems |
| **DORA** | Digital Operational Resilience Act (EU) |
| **Invocation** | Formally activating a BCP or crisis management plan |
