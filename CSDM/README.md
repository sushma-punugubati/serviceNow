# ServiceNow CSDM — Common Service Data Model

CSDM (Common Service Data Model) is ServiceNow's prescriptive framework for structuring CMDB data consistently across the entire platform. It defines *how* CIs should be classified, related, and organized — so that ITSM, ITOM, SecOps, IRM, and all other modules speak the same data language.

Without CSDM, every team models their CI data differently. CSDM gives a single, platform-wide standard that makes impact analysis, change risk, event correlation, and service topology all work correctly — because they all read from the same consistently structured CMDB.

## Files in this folder

| File | Description |
|------|-------------|
| `CSDM_Basics.md` | Framework overview, layers, maturity model, Service Graph Connectors |
| `CSDM_InterviewQuestions.md` | 20+ Q&A covering concepts, implementation, and scenarios |
| `CSDM_CommonIssues.md` | 10 real-world CSDM implementation issues with root causes and fixes |
| `CSDM_CustomerSuccessStories.md` | 5 case studies showing CSDM transformation outcomes |
| `CSDM_ProjectExperience_Challenges.md` | Personal CSDM project work and challenges faced |

## Key topics covered

- CSDM layers: Business Application, Application Service, Technical Service, Business Service
- CSDM maturity model: Foundation → Crawl → Walk → Run → Fly
- Service Graph Connectors vs. raw import sets
- CSDM relationship types (Runs on, Depends on, Hosted on)
- Service Offerings and Service Portfolio
- CSDM Health Score and data quality governance
- CSDM alignment with Discovery, Service Mapping, and ITOM
- Impact of CSDM on ITSM, Change Management, Event Management
