# ServiceNow FSO — Field Service Operations (Field Service Management)

## Study Files in This Folder

| File | What It Covers |
|------|----------------|
| `FSO_Basics.md` | Core concepts, FSO verticals (Banking/Insurance/Wealth), data model, key tables, workspace, AI, integrations, regulations |
| `FSO_InterviewQuestions.md` | 20+ Q&As across conceptual, technical, and scenario topics + study checklist |
| `FSO_CustomerSuccessStories.md` | 6 real-world stories (retail bank, card disputes, insurance, wealth, credit union, GDPR) with metrics |
| `FSO_CommonIssues_And_Pitfalls.md` | 10 common pitfalls with root causes, fixes, and quick troubleshooting reference |
| `FSO_ProjectExperience_Challenges.md` | Real project lessons: data modeling, workflow design, integration, change management, go-live |

---

## Quick Reference

| Concept | Answer |
|---------|--------|
| What is FSO/FSM? | Managing field technicians — dispatching, work orders, scheduling, mobile execution |
| Work Order table | `fsm_work_order` |
| Work Order Task table | `fsm_work_order_task` |
| Field Agent table | `fsm_agent` |
| Territory table | `fsm_territory` |
| Dispatcher Workspace | Central screen for dispatchers to view, assign, and manage work orders on a map |
| Scheduling types | Manual, Automatic (rule-based), Optimized (AI-driven) |
| Mobile app | ServiceNow Field Service app — technicians manage tasks on mobile |
| FSO + CSM | CSM case → Work Order → field technician dispatched |
| Planned Maintenance | Recurring scheduled work orders for preventive maintenance |
| Connected Operations | IoT integration — sensors trigger automatic Work Orders |
| Key metric | FTFR — First Time Fix Rate |

---

## Module Overview

**Field Service Operations (FSO)** — also called **Field Service Management (FSM)** — is ServiceNow's module for managing the complete lifecycle of work that happens in the field:

- **Work Order management:** Track every field job from creation to completion
- **Dispatcher Workspace:** Visual map-based scheduling and dispatch console
- **Scheduling Engine:** Auto-assign technicians based on skills, location, availability
- **Mobile Execution:** Field agents complete tasks on mobile with offline capability
- **Parts & Inventory:** Track parts used on jobs; manage van stock
- **Planned Maintenance:** Recurring preventive maintenance scheduling
- **Connected Operations:** IoT sensor triggers automatic work order creation

---

## Related Modules

- **CSM** — Customer cases escalate to Work Orders when field dispatch is needed
- **ITSM** — IT incidents can generate field work orders for hardware repair
- **ITAM** — Asset records (equipment being serviced) linked to Work Orders
- **CMDB** — CIs representing field equipment linked to Work Orders
- **IRM** — Safety and regulatory compliance for field operations
