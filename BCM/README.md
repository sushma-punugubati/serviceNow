# ServiceNow BCM — Business Continuity Management

## Study Files in This Folder

| File | What It Covers |
|------|----------------|
| `BCM_Basics.md` | Core concepts, BIA, recovery plans, RTO/RPO, regulatory requirements |
| `BCM_InterviewQuestions.md` | 20 Q&As across conceptual, configuration, and scenario topics |
| `BCM_CustomerSuccessStories.md` | Real-world BCM implementation stories with metrics and outcomes |
| `BCM_CommonIssues.md` | Common pitfalls, root causes, and fixes |

---

## Quick Reference

| Concept | Answer |
|---------|--------|
| What is BCM? | Managing plans to ensure critical business processes survive and recover from disruptions |
| BIA | Business Impact Analysis — identifies critical functions and the impact of disruption over time |
| BCP | Business Continuity Plan — steps to continue operations during disruption |
| DRP | Disaster Recovery Plan — steps to restore IT systems after disruption |
| RTO | Recovery Time Objective — maximum acceptable downtime |
| RPO | Recovery Point Objective — maximum acceptable data loss (in time) |
| MTPD | Maximum Tolerable Period of Disruption — beyond this, the business cannot recover |
| Key BCM table | `sn_bcm_plan` (BCM Plan records) |
| BIA table | `sn_bcm_bia` |
| ISO standard | ISO 22301 — Business Continuity Management Systems |
| Regulatory drivers | DORA (EU financial), FFIEC (US banks), APRA (Australia), MAS (Singapore) |
| BCM in IRM | BCM is part of ServiceNow's Operational Resilience module within IRM |

---

## Module Overview

**Business Continuity Management (BCM)** in ServiceNow is the capability — part of the IRM Operational Resilience module — that ensures organizations can continue operations and recover from disruptions:

- **Business Impact Analysis (BIA):** Identify which processes are critical and how long they can tolerate downtime
- **Plan Management:** Create, maintain, and test Business Continuity Plans and Disaster Recovery Plans
- **Recovery Objectives:** Define RTO and RPO per process/service
- **Testing:** Plan tabletop exercises, full simulations, and track gaps
- **CMDB Integration:** Map critical processes to the CIs and services they depend on
- **IRM Integration:** BCM findings link to risk register; control gaps tracked

---

## Related Modules

- **IRM/GRC** — BCM is a sub-module of Operational Resilience within IRM
- **CMDB** — Critical process dependencies mapped to CIs and business services
- **SecOps** — Cyber incidents can trigger BCP activation
- **ITSM** — IT DRP links to ITSM incident and problem management
- **SPM** — BCM gap remediation projects tracked in SPM
