# ServiceNow CMDB — Study Materials Index

> Configuration Management Database: The backbone of the ServiceNow platform

## Files in This Folder

### 1. [CMDB_Basics.md](CMDB_Basics.md)
Core concepts, CI classes hierarchy, key tables, Discovery, identification/reconciliation rules, health KPIs, glossary.

### 2. [CMDB_CustomerSuccessStories.md](CMDB_CustomerSuccessStories.md)
Real-world CMDB scenarios: outage prevention, change impact, cloud visibility, audit compliance.

### 3. [CMDB_InterviewQuestions.md](CMDB_InterviewQuestions.md)
35+ Q&As on CMDB, Discovery, MID Server, CI relationships, reconciliation, and health management.

### 4. [CMDB_CommonIssues.md](CMDB_CommonIssues.md)
35+ documented CMDB implementation problems with causes and solutions.

## Quick Reference

```
Core tables:
  cmdb          = Base table (abstract)
  cmdb_ci       = Configuration Item (all CIs live here or child tables)
  cmdb_rel_ci   = CI Relationships
  cmdb_ci_hardware  = Hardware CIs
  cmdb_ci_computer  = Computers
  cmdb_ci_server    = Servers
  cmdb_ci_appl      = Application CIs

Discovery:
  MID Server    = Secure bridge between ServiceNow and target infrastructure
  Probe         = Gathers raw data from targets
  Sensor        = Processes probe data → writes CI records
  Horizontal    = Finds infrastructure (servers, network)
  Vertical      = Maps applications and dependencies

Health KPIs = Completeness + Correctness + Compliance
CSDM        = Common Service Data Model (foundational layer)
```

## Sources
- [ServiceNow CMDB Community Interview Questions](https://www.servicenow.com/community/cmdb-forum/top-cmdb-amp-discovery-interview-questions/m-p/3306831)
- [ServiceNow Docs — CMDB Tables](https://www.servicenow.com/docs/bundle/zurich-servicenow-platform/page/product/configuration-management/reference/cmdb-tables-details.html)
- [CMDB Implementation Issues 2026](https://servicenowspectaculars.com/servicenow-configuration-management-database-cmdb-implementation-issues-2026/)
