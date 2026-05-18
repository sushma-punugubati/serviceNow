# ServiceNow CSM — Study Materials Index

> Created: May 2026 | Sources: ServiceNow Official Docs, ServiceNow Community, SNReady, ProcessExam, industry case studies

---

## Files in This Folder

### 1. [CSM_Basics.md](CSM_Basics.md)
**Start here.** Comprehensive fundamentals guide covering:
- What CSM is and its business value
- Business models: B2B, B2C, B2B2C
- All core concepts explained in plain English (Account, Contact, Consumer, Case, Entitlement, Contract, etc.)
- Key tables reference with table names and prefixes
- Case lifecycle (New → Closed)
- Roles and responsibilities (internal + external)
- Omnichannel support channels
- CSM vs ITSM comparison
- Key modules: Workspace, Playbooks, Guided Decisions, AWA, Major Issue Management
- Data model relationships
- Important plugins and implementation methodology
- Complete glossary

### 2. [CSM_CustomerSuccessStories.md](CSM_CustomerSuccessStories.md)
**Learn by example.** 8 real-world stories that explain CSM features through practical scenarios:
| Story | Concept Illustrated |
|-------|-------------------|
| Stock Exchange (90K cases, 14 depts) | Account 360° view, omnichannel |
| Software Company (expired contracts) | Entitlements & Service Contracts |
| Telecom Outage (50K customers) | Major Issue Management + Proactive CSM |
| Bank with Advisor Partners | Partner Management |
| Electronics Manufacturer (1M devices) | Sold Products + Install Base Items |
| Industrial Equipment (8hr downtime) | CSM + Field Service + AWA |
| 401k Provider (80% simple calls) | Case Deflection, Virtual Agent |
| SaaS Company (new agents struggling) | Playbooks + Guided Decisions |

Includes ROI metrics table and feature-to-scenario mapping.

### 3. [CSM_InterviewQuestions.md](CSM_InterviewQuestions.md)
**Interview and exam prep.** 38 questions across 6 sections:
- **Part 1**: Conceptual fundamentals (10 questions)
- **Part 2**: Roles & configuration (8 questions)
- **Part 3**: Scenario-based (7 questions)
- **Part 4**: Technical/configuration (5 questions)
- **Part 5**: CIS-CSM exam quick facts table
- **Part 6**: Tricky "gotcha" questions (4 questions)

Includes a study checklist of the 13 most important facts to memorize.

---

## Quick Reference: Most Important Facts

```
Contact     = B2B (company employee)     table: customer_contact
Consumer    = B2C (individual customer)  table: csm_consumer
Case table  = sn_customerservice_case
Plugin ID   = com.sn_customerservice
Final state = Closed (not Resolved)
Methodology = Now Create (not Agile)
AWA channels = Case, Chat, Walk-up
Product table ≠ ITSM Asset table (CSM uses its own)
Entitlement = support type + channels + SLA tier
Contract → Entitlements → applied to → Cases
```

---

## Recommended Study Order

1. **Day 1-2**: Read `CSM_Basics.md` — build mental model of data relationships
2. **Day 3**: Read `CSM_CustomerSuccessStories.md` — anchor concepts to real scenarios
3. **Day 4**: Work through `CSM_InterviewQuestions.md` — test yourself, note gaps
4. **Day 5**: Re-read weak areas in Basics, re-attempt missed questions

---

## Additional Resources

- [ServiceNow Official CSM Docs (Zurich)](https://www.servicenow.com/docs/r/zurich/customer-service-management/c_CustomerServiceManagement.html)
- [CSM Fundamentals — ServiceNow Community](https://www.servicenow.com/community/csm-articles/customer-service-management-csm-fundamentals/ta-p/2296920)
- [ServiceNowWithRunjay CSM Guide](https://servicenowwithrunjay.com/customer-service-management/)
- [CIS-CSM Exam Guide 2026 — SNReady](https://snready.com/blog/servicenow-cis-csm-exam-guide-2026)
- [CIS-CSM Sample Questions — ProcessExam](https://www.processexam.com/servicenow/servicenow-certified-implementation-specialist-customer-service-management-cis-csm)
- [Now Learning — CSM Essentials (Free)](https://learning.servicenow.com)
