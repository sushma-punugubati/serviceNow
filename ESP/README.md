# ServiceNow ESP — Enterprise Services Portfolio

## Study Files in This Folder

| File | What It Covers |
|------|----------------|
| `ESP_Basics.md` | Core concepts, key tables, service lifecycle, ITSM integration |
| `ESP_InterviewQuestions.md` | 20 Q&As across conceptual, configuration, and scenario topics |
| `ESP_CustomerSuccessStories.md` | Real-world implementation stories with metrics and outcomes |
| `ESP_CommonIssues.md` | Common pitfalls, root causes, and fixes |

---

## Quick Reference

| Concept | Answer |
|---------|--------|
| What is ESP? | Managing the full lifecycle of enterprise services (pipeline → catalog → retired) across IT and business |
| Service Portfolio vs Catalog | Portfolio = all services (pipeline + active + retired). Catalog = what's currently available to request. |
| Service lifecycle stages | Pipeline → Active (Catalog) → Retired |
| Key ESP table | `service_offering` (Service Offerings) |
| Service CI table | `cmdb_ci_service` |
| Catalog item table | `sc_cat_item` |
| Service owner's role | Accountable for service quality, cost, and lifecycle decisions |
| CSDM alignment | ESP requires proper CSDM hierarchy: Business App → Application Service → Technical Service |
| ESP vs SPM | ESP = managing what services we OFFER. SPM = managing projects/programs to DELIVER change. |
| ESM | Enterprise Service Management — extending ServiceNow beyond IT to HR, Finance, Legal, Facilities |

---

## Module Overview

**Enterprise Services Portfolio (ESP)** is ServiceNow's discipline and capability for managing the full lifecycle of services offered across an enterprise. It provides:

- A **single source of truth** for all enterprise services
- Tools for **service owners** to manage service quality, cost, and lifecycle
- **Service Portfolio** view: pipeline (future services), catalog (active), retired (decommissioned)
- Integration with CMDB, ITSM, SPM, and ITAM for full service context
- Foundation for **Enterprise Service Management (ESM)** — extending service management beyond IT

---

## Related Modules

- **ITSM** — Service fulfillment (requests fulfilled from the catalog)
- **CMDB/CSDM** — Services mapped to technical CIs and infrastructure
- **SPM** — Projects that build or change services in the portfolio
- **ITAM** — Assets and licenses that support services
- **IRM** — Risk and compliance posture of services in the portfolio
