# ServiceNow — Most Frequently Asked Interview Questions

> Covers platform fundamentals, scripting, ITSM, CMDB/CSDM, integrations, Flow Designer, security, deployment, AI, and scenario-based questions. Organized from most-asked to deeper technical depth.

---

## QUICK NAVIGATION

| Section | Topics |
|---------|--------|
| [1. Platform Fundamentals](#1-platform-fundamentals) | Architecture, tables, forms, upgrade |
| [2. Scripting & Development](#2-scripting--development) | Business Rules, Client Scripts, Script Includes, GlideRecord |
| [3. ITSM](#3-itsm) | Incident, Problem, Change, Catalog, SLA |
| [4. CMDB & CSDM](#4-cmdb--csdm) | Discovery, IRE, CSDM layers, Service Graph |
| [5. Integrations](#5-integrations) | REST, SOAP, IntegrationHub, MID Server |
| [6. Flow Designer & Automation](#6-flow-designer--automation) | Flows, subflows, actions, Orchestration |
| [7. Security & Access Control](#7-security--access-control) | ACLs, roles, domain separation |
| [8. Deployment & Administration](#8-deployment--administration) | Update sets, cloning, upgrades, ATF |
| [9. AI & Now Assist](#9-ai--now-assist) | Now Assist, Virtual Agent, Predictive Intelligence |
| [10. Scenario-Based Questions](#10-scenario-based-questions) | Real-world problem-solving |

---

## 1. Platform Fundamentals

**Q: What is ServiceNow and what does it do?**

ServiceNow is a cloud-based Platform-as-a-Service (PaaS) that provides IT Service Management, IT Operations Management, HR Service Delivery, Customer Service Management, and many other enterprise workflows on a single platform. Its core value is consolidating fragmented, email-based processes into structured, trackable, SLA-governed workflows — with a single system of record (the CMDB and the Now Platform database) used by all modules.

---

**Q: What is the ServiceNow architecture?**

ServiceNow is a multi-instance SaaS platform. Each customer gets their own dedicated database instance (not shared with other customers). The architecture has three layers:
- **Database layer:** All data stored in tables (MySQL-based)
- **Application layer:** Server-side Java application processing business logic
- **Presentation layer:** Browser-based UI (Next Experience / Classic UI)

Key platform components:
- **Tables:** All data lives in tables — `incident`, `cmdb_ci`, `sc_request`, etc. Everything extends from `task` (the parent of all work items)
- **ECC Queue:** Communication channel between ServiceNow and MID Servers
- **Scheduler:** Manages scheduled jobs, SLA calculations, notifications
- **Workflow/Flow Engine:** Processes automation logic

---

**Q: What is table inheritance in ServiceNow?**

ServiceNow uses **single-table inheritance** — child tables extend parent tables and inherit all their fields, business rules, and ACLs.

Example hierarchy:
```
task (base)
  └── incident
  └── change_request
  └── sc_request
       └── sc_req_item
```

A query on `task` returns records from ALL child tables. A query on `incident` returns only incidents. Fields added to `task` appear on all child tables. Business Rules on `task` fire for incidents, changes, and catalog items.

**Why it matters:** If a Business Rule fires on `task` when it should only fire on `incident`, you get unexpected behavior across all modules.

---

**Q: What is the difference between configuration and customization in ServiceNow?**

| Configuration | Customization |
|--------------|---------------|
| Using OOTB platform tools to achieve requirements (UI Policies, Workflows, Flow Designer, ACLs) | Modifying core platform code, extending base tables with scripts, overriding OOTB behavior |
| Upgrade-safe — platform changes do not break configuration | Creates upgrade risk — customizations may conflict with new releases |
| No coding required (or minimal) | Requires scripting (JavaScript, Business Rules, Client Scripts) |
| Always preferred first | Only when configuration cannot meet the requirement |

**ServiceNow best practice:** Configuration first, customization only when necessary and with documented upgrade risk.

---

**Q: What is a Scope in ServiceNow?**

A **Scope** (or application scope) is a namespace that isolates an application's artifacts from the global platform and other applications.

- **Global scope:** Platform-wide — scripts and configurations accessible by everything. Upgrade-risky — customizations in global scope can be overwritten by ServiceNow upgrades.
- **Scoped applications:** Custom namespace (e.g., `x_acme_myapp`) — scripts, tables, and records are isolated within the scope. Upgrade-safe — ServiceNow upgrades cannot touch scoped app code.

**Best practice:** Always develop in a scoped application. Never put customizations in global scope unless absolutely required.

---

**Q: What are the most important base tables to know?**

| Table | Label | Purpose |
|-------|-------|---------|
| `task` | Task | Parent of all work items — incidents, changes, requests, cases |
| `incident` | Incident | ITSM incidents |
| `change_request` | Change | Change requests |
| `sc_request` | Request | Service Catalog parent request (REQ) |
| `sc_req_item` | Request Item | Individual catalog item in a request (RITM) |
| `cmdb_ci` | Configuration Item | All CMDB CIs |
| `cmdb_rel_ci` | CI Relationship | Relationships between CIs |
| `sys_user` | User | All users |
| `sys_user_group` | Group | Assignment and approval groups |
| `kb_knowledge` | Knowledge | Knowledge Base articles |
| `contract_sla` | SLA Definition | SLA templates |
| `task_sla` | SLA (task record) | Active SLA instance on a ticket |

---

**Q: What is the difference between a Reference field and a List field?**

- **Reference field:** Points to a single record in another table. Example: `incident.assigned_to` references one record in `sys_user`. The field stores the `sys_id` of the referenced record.
- **List (glide_list):** Points to multiple records in another table. Example: `incident.watch_list` — multiple users watching an incident. Stored as a comma-separated list of sys_ids.

**In GlideRecord:** Reference fields are accessed via dot-walking (`current.assigned_to.email`). List fields are accessed via `getRefRecord()` or by splitting the comma-separated value.

---

## 2. Scripting & Development

**Q: What is a Business Rule and when should you use one?**

A **Business Rule** is a server-side script that runs automatically when a record in a specific table is created, read, updated, or deleted.

**When/Before:** `When = Before` — runs before the database write. Use for: field validation, field calculation before saving, preventing saves with `current.setAbortAction(true)`.

**When/After:** `When = After` — runs after the database write. Use for: sending notifications, creating related records, updating other tables.

**When/Async:** `When = Async` — runs in a background thread after the database write, without blocking the user. Use for: expensive operations (emails, third-party calls) that do not need to complete before the user gets their response.

**Common anti-patterns to avoid:**
- Unlimited GlideRecord queries inside a loop (N+1 query problem)
- Calling `update()` inside an after Business Rule (causes recursive firing)
- Using `current.setWorkflow(false)` without understanding what it suppresses

---

**Q: What is a Script Include and when would you use one?**

A **Script Include** is a reusable library of server-side JavaScript functions that can be called from Business Rules, Client Scripts (via GlideAjax), REST APIs, and other server-side contexts.

**Use cases:**
- Common utility functions used across multiple Business Rules
- Complex logic that needs to be called from multiple places
- Logic that needs to be unit-testable independently
- Logic called from Client Scripts via GlideAjax (must extend `AbstractAjaxProcessor`)

**Class-based vs. procedural:**
```javascript
// Class-based (recommended)
var MyUtil = Class.create();
MyUtil.prototype = {
    initialize: function() {},
    getAssignmentGroup: function(category) {
        var gr = new GlideRecord('sys_user_group');
        gr.addQuery('u_category', category);
        gr.setLimit(1);
        gr.query();
        return gr.next() ? gr.sys_id.toString() : '';
    },
    type: 'MyUtil'
};
```

---

**Q: What is the difference between a Client Script and a UI Policy?**

| UI Policy | Client Script |
|-----------|--------------|
| No-code — uses condition builder | Requires JavaScript |
| Controls: mandatory, read-only, visible | Can do anything — calculate, validate, call server |
| Declarative — easy to read and audit | Imperative — flexible but harder to audit |
| Reversal is automatic when condition changes | Must explicitly handle reverse condition |
| Best for: show/hide/mandatory field rules | Best for: complex validation, server lookups, dynamic field population |

**Best practice:** Always use UI Policy for simple show/hide/mandatory rules. Only use a Client Script when the logic cannot be expressed declaratively.

---

**Q: What is GlideRecord and what are the best practices?**

**GlideRecord** is the ServiceNow API for querying and modifying database records in server-side scripts.

```javascript
// Query pattern
var gr = new GlideRecord('incident');
gr.addQuery('state', 1);                    // New incidents
gr.addQuery('priority', 'IN', '1,2');       // P1 or P2
gr.orderByDesc('sys_created_on');
gr.setLimit(50);
gr.query();
while (gr.next()) {
    gs.log(gr.number + ': ' + gr.short_description);
}

// Update pattern
var gr = new GlideRecord('incident');
if (gr.get('INC0012345', true)) {           // get by number
    gr.state = 6;                            // Resolved
    gr.close_notes = 'Auto-resolved by script';
    gr.update();
}
```

**Best practices:**
1. Always use `addQuery()` — never string-concatenate conditions (injection risk)
2. Use `setLimit()` when you do not need all records
3. Check `gr.next()` before accessing fields — never assume the record exists
4. Use `GlideAggregate` for count/sum — never loop GlideRecord just to count
5. In After Business Rules, avoid calling `gr.update()` on `current` — causes recursion
6. Use `gs.nil()` to check if a reference field is empty — `current.assigned_to == ''` does not work reliably

---

**Q: What is GlideAjax and when do you use it?**

**GlideAjax** allows Client Scripts (running in the browser) to call server-side Script Include methods asynchronously — without refreshing the page.

**Use case:** A dropdown change on the form needs to look up a value from the database (which the client cannot access directly). GlideAjax calls the server, gets the result, and updates the form.

```javascript
// Client Script (onChange)
function onChange(control, oldValue, newValue, isLoading) {
    if (isLoading || newValue == '') return;
    var ga = new GlideAjax('MyUtil');        // Script Include name
    ga.addParam('sysparm_name', 'getManagerEmail');
    ga.addParam('sysparm_user_id', newValue);
    ga.getXMLAnswer(function(answer) {
        g_form.setValue('u_manager_email', answer);
    });
}
```

The Script Include must extend `AbstractAjaxProcessor` to be callable via GlideAjax.

---

**Q: What is the difference between `gs.getUser()` and `g_user`?**

- **`gs.getUser()`:** Server-side API. Returns a `GlideUser` object for the currently logged-in user. Available in Business Rules, Script Includes, and server-side scripts.
- **`g_user`:** Client-side object. Available in Client Scripts and UI Policies. Provides user information like `g_user.userID`, `g_user.hasRole('itil')`, `g_user.firstName`.

**Rule of thumb:** `gs.getUser()` is server-side. `g_user` is client-side. Never use `g_user` in a Business Rule and never use `gs.getUser()` in a Client Script.

---

**Q: What is a Scheduled Job and when do you use it?**

A **Scheduled Job** (sys_trigger) is a server-side script that runs automatically on a defined schedule — without any user action.

**Use cases:**
- Auto-close resolved incidents after 5 days
- Send weekly SLA compliance reports
- Clean up stale data or temporary records
- Recalculate metrics or aggregate values
- Trigger integrations that poll external systems on a schedule

**Important:** Scheduled Jobs run in a system context (admin-level). Be careful with `gr.update()` calls in scheduled jobs — they bypass ACLs.

---

## 3. ITSM

**Q: What are the Incident Management states and their meaning?**

| State # | State Label | Meaning |
|---------|-------------|---------|
| 1 | New | Created, not yet assigned |
| 2 | In Progress | Being worked by assigned team |
| 3 | On Hold | Waiting for external input (customer, vendor, change) |
| 6 | Resolved | Fix applied, waiting for confirmation |
| 7 | Closed | Confirmed resolved or auto-closed |
| 8 | Cancelled | Not a valid incident |

---

**Q: What is the difference between Impact and Urgency in Incident Management?**

- **Impact:** How many users, services, or processes are affected — the *breadth* of the problem. Set by the analyst based on who/what is affected.
- **Urgency:** How time-sensitive the fix is — the business tolerance for the disruption. Set based on business context.

**Priority** is calculated from the combination: `Impact × Urgency = Priority (P1-P4)`. ServiceNow calculates this automatically via a Business Rule using a priority lookup matrix.

**Interview tip:** P1 requires both high Impact AND high Urgency. A VIP's laptop being down might be high Urgency (they need it now) but low Impact (only one user) — making it P2, not P1.

---

**Q: What is the Service Catalog REQ → RITM → Task relationship?**

- **REQ (`sc_request`):** The parent record created when an employee submits a cart of catalog items. One REQ per submission session. Contains requester and billing information.
- **RITM (`sc_req_item`):** One per catalog item ordered. The RITM goes through approval and fulfillment. If you order 3 items, you get 1 REQ with 3 RITMs.
- **Catalog Task (`sc_task`):** Sub-tasks of an RITM, each assigned to a different team for fulfillment. One RITM can have multiple parallel tasks.

**Lifecycle:** REQ is open → RITMs complete → REQ auto-closes. RITM is approved → Tasks created → Tasks complete → RITM auto-closes.

---

**Q: How does an SLA pause condition work?**

An SLA pause condition is configured on the SLA Definition record. When the condition evaluates to `true`, the SLA timer stops. When the condition evaluates to `false` again, the timer resumes.

**Common pause conditions:**
- `state = 3` (On Hold) — timer pauses when incident is On Hold
- A custom wait-reason field is populated

**Retroactive pause:** If an On Hold state was set before the pause condition was configured, the pause can be applied retroactively back to when the condition was actually true — preventing false SLA breaches from configuration gaps.

**Troubleshooting SLA issues:** Check the SLA task record's business elapsed vs. actual elapsed time. Review the SLA Definition conditions. Use the SLA Debugger (available from the ticket's SLA related list) to see exactly why the SLA is in its current state.

---

**Q: What is a Record Producer and how is it different from a Catalog Item?**

- **Catalog Item** (`sc_cat_item`): Creates a `sc_req_item` (RITM) when submitted. Used for standard service requests.
- **Record Producer**: Creates a record in any specified table when submitted (e.g., `incident`, `change_request`, a custom table). Used when the self-service form should create something other than a catalog request.

**Example:** A "Report a Problem" form on the Service Portal that creates an Incident directly — this is a Record Producer. The user fills in a catalog-style form but an `incident` record is created, not an RITM.

---

## 4. CMDB & CSDM

**Q: What is the CMDB and why is it important?**

The **CMDB (Configuration Management Database)** is ServiceNow's central repository of all Configuration Items (CIs) and their relationships. It is the data backbone for almost every other platform module.

**Why it matters:**
- Incident Management: CI field shows what system is affected — who owns it, what services depend on it
- Change Management: impact analysis traverses CI relationships to show what else might break
- Event Management: alerts are bound to CIs for business context
- SecOps: vulnerabilities are prioritized by affected CI's business criticality

**Without an accurate CMDB:** Features like impact analysis, change risk, and service health dashboards produce empty or incorrect results.

---

**Q: What is the IRE and what problem does it solve?**

The **IRE (Identification and Reconciliation Engine)** solves two problems when data from multiple sources (Discovery, SCCM, manual entry, cloud APIs) lands in the CMDB:

1. **Identification:** Is this CI new, or does a record already exist for it? IRE uses configurable identification rules (serial number, VM UUID, hostname + domain) to match incoming data to existing CI records. Without good identification rules, every Discovery run creates duplicates.

2. **Reconciliation:** When two data sources provide different values for the same CI attribute, which wins? IRE applies reconciliation rules to define the authoritative source per attribute. Example: Discovery wins for IP address (most current), SCCM wins for installed software.

---

**Q: What is CSDM and why does ServiceNow recommend it?**

**CSDM (Common Service Data Model)** is ServiceNow's prescriptive framework for structuring CMDB data. It defines standard CI classes, relationship types, and data model layers.

**Key layers:**
- Business Application → Application Service → Technical Service → Infrastructure

**Why recommended:** All CMDB-dependent features (Service Mapping, Event Management, Change impact) work correctly only when data follows CSDM standards. Without CSDM: Event Management alerts cannot determine which business service is affected; Change impact analysis returns empty results; BSM maps render incorrectly.

**The most important layer:** Application Service — the bridge between the business view and the technical infrastructure view.

---

**Q: What is the difference between Discovery and Service Mapping?**

| Discovery | Service Mapping |
|-----------|-----------------|
| Finds infrastructure: what CIs exist | Traces application dependencies: how CIs are connected |
| Scans IP ranges with probes | Starts at an entry point and follows connections |
| Creates individual CI records | Creates CI records AND relationship records |
| "What servers are in this network?" | "What is the full dependency chain for the Payment service?" |
| Horizontal: infrastructure; Vertical: applications | Full service topology including business service layer |

**Together:** Discovery populates the CMDB with CI records. Service Mapping populates the *relationships* between those CIs that form a service topology map. Both are needed for a complete, useful CMDB.

---

## 5. Integrations

**Q: What are the main integration patterns in ServiceNow?**

| Pattern | When to Use |
|---------|-------------|
| **REST (outbound)** | Call an external REST API from ServiceNow — most modern systems |
| **REST (inbound)** | External system calls ServiceNow's REST API to create/update records |
| **SOAP** | Legacy systems that expose SOAP web services |
| **IntegrationHub** | Pre-built spokes for common systems (Jira, Salesforce, Slack); custom spoke builder for others |
| **MID Server** | For integrations that need access to on-premises systems behind firewalls |
| **Import Sets** | Batch data loads — external system dumps data to a staging table, transform map loads to target table |
| **Scripted REST API** | Build a custom REST endpoint in ServiceNow that external systems can call |

---

**Q: What is IntegrationHub and why is it preferred over custom REST messages?**

**IntegrationHub** is ServiceNow's integration platform — it provides:
- Pre-built **spokes** for 200+ common systems (Jira, ServiceNow, Salesforce, AWS, Azure, Microsoft Teams, etc.)
- **Custom spoke builder** for systems without a pre-built spoke
- Visual **Flow Designer integration** — drag-and-drop integration actions into flows
- Built-in **retry logic, error handling, and logging** — no need to build these manually
- **Connection aliases** — credentials stored securely and referenced by name, not hardcoded

**vs. custom REST messages (sys_web_service_definition):** Custom REST messages require manually scripting authentication, retry logic, error handling, and logging. IntegrationHub provides these as platform features. For new integrations, IntegrationHub is always preferred.

---

**Q: What is the MID Server and what does it do for integrations?**

The **MID (Management, Instrumentation, and Discovery) Server** is a Java application installed in the on-premises network that acts as a bridge between ServiceNow (cloud) and internal systems behind firewalls.

**Integration use cases for MID Server:**
- IntegrationHub actions that need to reach on-premises APIs not accessible from the internet
- JDBC connections to on-premises databases for import sets
- Discovery probes against internal infrastructure
- Event Management connectors for on-premises monitoring tools

**Architecture:** MID Server initiates an outbound HTTPS connection to ServiceNow — no inbound firewall rules needed. ServiceNow sends work items to the MID Server via the ECC Queue; the MID Server executes them and returns results.

---

**Q: How do you authenticate to an external REST API from ServiceNow?**

Authentication methods in ServiceNow IntegrationHub and REST messages:

| Method | Use Case |
|--------|---------|
| **Basic Auth** | Username + password — older APIs, internal systems |
| **OAuth 2.0** | Most modern APIs — Authorization Code, Client Credentials, or Password grant |
| **API Key** | Header-based key authentication — common for public APIs |
| **JWT** | JSON Web Token — used by some enterprise APIs |
| **Mutual TLS (mTLS)** | Certificate-based — high-security environments |

**Best practice:** Store all credentials in the ServiceNow **Credential Store** (Discovery → Credentials or Connection & Credential Aliases) — never hardcode credentials in scripts. Reference them by alias name.

---

## 6. Flow Designer & Automation

**Q: What is Flow Designer and how does it differ from the legacy Workflow Editor?**

**Flow Designer** is ServiceNow's modern, no-code/low-code automation builder. It replaced the legacy Workflow Editor for new development.

| Workflow Editor | Flow Designer |
|-----------------|---------------|
| Legacy (still works but deprecated direction) | Current standard |
| Activity-based with drag-and-drop canvas | Step-based with linear structure |
| Limited reusability | Reusable subflows and actions |
| No IntegrationHub integration | Native IntegrationHub integration |
| Complex to debug | Clear execution history and debugging |
| Harder for non-developers | More accessible to low-code practitioners |

**When to use Workflow Editor:** Migrating or maintaining existing workflows. All new automation should be built in Flow Designer.

---

**Q: What is the difference between a Flow, Subflow, and Action in Flow Designer?**

- **Flow:** The main automation — triggered by a specific event (record created, scheduled, manual trigger). Contains the overall process logic.
- **Subflow:** A reusable module of steps that can be called from multiple parent Flows. Like a function. Example: "Get Manager Approval" subflow used in both Change and Catalog approval flows.
- **Action:** A single, atomic operation — the smallest reusable unit. Examples: "Create Record," "Send Email," "Call REST Endpoint," "Run Script." Actions are the building blocks of Flows and Subflows.

**Best practice:** Extract repeated logic into Subflows. Build custom Actions for complex operations that need to be reused across many Flows.

---

**Q: What triggers are available in Flow Designer?**

| Trigger Type | When it fires |
|-------------|---------------|
| **Record Created** | When a new record is inserted in a table |
| **Record Updated** | When a specific field changes on a record |
| **Record Created or Updated** | Either insert or update |
| **Record Deleted** | When a record is deleted |
| **Scheduled** | On a defined schedule (hourly, daily, weekly) |
| **Service Catalog** | When a catalog item is ordered |
| **Inbound Email** | When an email matching criteria is received |
| **Manual** | Invoked by a UI Action button click |
| **SLA** | When an SLA reaches a defined percentage or breaches |
| **Subflow** | Invoked from a parent flow |

---

## 7. Security & Access Control

**Q: What are ACLs and how do they work?**

**ACLs (Access Control Lists)** define who can perform which operations on which data.

**ACL structure:**
- **Object:** What is being protected (table name, field name, or record)
- **Operation:** What is being controlled (read, write, create, delete)
- **Requires role / condition / script:** Who is allowed (role check, condition evaluation, or custom script)

**Evaluation order:** For a user to access data, ALL of the following must pass:
1. Table-level ACL (can the user read this table?)
2. Field-level ACL (can the user read this field?)
3. Record-level condition (if configured)

If any ACL returns false → access denied.

**Critical:** ACLs are bypassed by admin (`admin` role bypasses all ACLs). Always test ACLs logged in as a non-admin user persona, not as admin.

---

**Q: What is the difference between a Role and a Group in ServiceNow?**

| Role | Group |
|------|-------|
| Controls what a user can *do* — access, features, capabilities | Controls what a user is *assigned to* — routing, notifications, approvals |
| Used in ACLs, UI Policies, Business Rule conditions | Used in assignment rules, notification recipients, approval rules |
| Examples: `itil`, `admin`, `catalog_admin` | Examples: "Network Team", "HR Benefits", "Change Advisory Board" |
| Inherited — roles granted to groups are inherited by group members | Members can be users, other groups, or dynamic membership lists |

**A user can have roles directly OR inherit them from group membership.** Grant roles to groups, not individual users — easier to maintain.

---

**Q: What is Domain Separation and when is it used?**

**Domain Separation** is a ServiceNow feature that partitions data and configuration within a single instance so that different organizations (domains) cannot see each other's records.

**Use cases:**
- **Managed Service Providers (MSPs):** Multiple customers on one instance, completely isolated
- **Enterprise:** Subsidiaries or business units with strict data separation requirements

**How it works:** Every record has a `sys_domain` field. ACLs evaluate domain membership — users only see records in their domain (and parent domains). Configurations can be domain-specific or global.

**Complexity warning:** Domain Separation significantly increases implementation complexity. Everything — tables, workflows, catalog items, ACLs — must be domain-aware. Only use when true multi-tenancy is required.

---

## 8. Deployment & Administration

**Q: What is an Update Set and how does it work?**

An **Update Set** is a container that captures all configuration changes made in a ServiceNow instance and allows them to be transported to other instances (DEV → TEST → UAT → PROD).

**How it works:**
1. Create a named Update Set in DEV and set it as current
2. Make configuration changes (create Business Rules, modify catalog items, etc.) — all changes are captured in the Update Set
3. Complete and export the Update Set as an XML file
4. Import the XML into the target instance
5. Preview (check for conflicts) → Commit (apply changes)

**Best practices:**
- One Update Set per story/change — never mix unrelated changes
- Name consistently: `PROJ-1234-short-description`
- Preview before committing — review all conflicts
- Document what is in each set — other developers need to know

---

**Q: What is ATF and how is it used?**

**ATF (Automated Test Framework)** is ServiceNow's built-in test automation platform. It allows developers to write automated tests for platform configurations and customizations.

**Key concepts:**
- **Test:** A single test case with setup, steps, and assertions
- **Test Suite:** A collection of related tests run together
- **Quick Start Tests:** ServiceNow-provided OOTB tests for common scenarios

**Integration with CI/CD:** ATF tests can be integrated with external CI/CD pipelines (Jenkins, GitLab) — tests must pass before changes are promoted to the next environment.

**Best practice:** Write ATF tests for all critical paths before going live — incident creation, catalog item submission, approval workflows. Hook tests into every TEST environment deployment so regressions are caught immediately.

---

**Q: What is the process for upgrading a ServiceNow instance?**

1. **Plan:** Review the upgrade guide for the target release — identify skipped records (customizations that will be overwritten), deprecated features, and new plugin requirements

2. **Clone PROD to sub-prod:** Clone the production instance to a separate upgrade test environment so the upgrade is tested on real data

3. **Run upgrade:** Trigger the upgrade in the test environment. Review the **Upgrade Monitor** for progress and issues

4. **Review skipped records:** Every customization that conflicts with a new OOTB record appears as a "skipped record." Review each one: accept the new OOTB version, keep the customization, or merge both

5. **Regression testing:** Run the full ATF test suite. Manually test all critical business processes

6. **Promote to PROD:** Schedule a maintenance window, clone PROD again (to include recent changes since the test clone), run the upgrade on PROD

7. **Post-upgrade validation:** Confirm all integrations are working, MID Servers have upgraded, and key workflows are functioning

---

**Q: What is Instance Cloning in ServiceNow?**

**Cloning** copies a source instance's data and configuration to a target instance, making the target an exact replica of the source at the point in time of the clone.

**Common use cases:**
- Clone PROD to DEV/TEST for realistic development environments
- Clone PROD to an upgrade test environment before running an upgrade
- Refresh UAT with current PROD data before testing

**Important:** Cloning overwrites the target instance completely. Any customizations in the target that are not also in the source will be lost. Always complete and export Update Sets before allowing the target to be cloned over.

---

## 9. AI & Now Assist

**Q: What is Now Assist and how does it work?**

**Now Assist** is ServiceNow's generative AI layer — powered by large language models — that adds AI-generated content to existing workflows. It can summarize incidents, draft resolution notes, answer employee HR questions, and generate code for developers.

**How it works:**
1. User triggers an AI skill (opens an incident → Now Assist auto-summarizes)
2. Now Assist retrieves context from the ServiceNow record (work notes, CI data, CSDM relationships)
3. A prompt is constructed with context + task instruction
4. The LLM processes the prompt and returns generated content
5. Content is shown as a draft — the human reviews, edits, and approves

**Key principle:** Now Assist augments human agents — it generates drafts and summaries that humans review before acting. It does not make decisions or take actions autonomously (that is AI Agents).

---

**Q: What is the difference between Virtual Agent and Now Assist?**

| Virtual Agent | Now Assist |
|---------------|-----------|
| Structured chatbot — scripted conversation flows | Generative AI — open-ended natural language |
| NLU classifies intent → routes to topic flow | LLM generates a response from context |
| Best for: task completion (create ticket, check status, reset password) | Best for: answering complex questions, summarizing records, drafting content |
| Predictable, deterministic outputs | Variable, contextually tailored outputs |
| Available since early ServiceNow releases | Introduced in recent releases (Washington+) |

In modern implementations, they work together: Virtual Agent handles structured task completion; Now Assist handles the open-ended Q&A within the same chat interface.

---

**Q: What is Predictive Intelligence and what does it automate?**

**Predictive Intelligence** applies ML to historical ServiceNow records to predict categorical outcomes on new records.

**Common uses:**
- Auto-assign incident **category and subcategory** from short description
- Auto-route to the correct **assignment group**
- Predict **change risk** level
- Auto-categorize **CSM cases**
- Find **similar incidents** for suggested resolution

**How it works:** A model is trained on historical records (minimum 400 examples per category). When a new record is created, the model predicts the outcome. If confidence is above the auto-apply threshold (typically 85%), the prediction is applied automatically. Below that, it is shown as a suggestion.

**Data quality is everything:** Training on inconsistently labeled historical data produces a poor model. Clean, consistently labeled training data outperforms large volumes of messy data every time.

---

## 10. Scenario-Based Questions

**Q: A Business Rule is running when it should not be. How do you debug it?**

1. **Check the Business Rule conditions:** Open the Business Rule record and review the Condition field and When/Table settings. Is the condition too broad?
2. **Add logging:** Add `gs.log('BR fired: ' + current.number)` temporarily to confirm the rule is actually firing (vs. a different rule causing the behavior)
3. **Check execution order:** Other Business Rules may be setting a field that triggers this one. Review all active Business Rules on the table
4. **Check if it is a child table issue:** If the Business Rule is on `task`, it fires for incidents, changes, and all task sub-types. If only meant for incidents, add a condition: `current.sys_class_name == 'incident'`
5. **Use the Script Debugger:** ServiceNow's built-in Script Debugger can step through server-side code in real time. Enable it in System Diagnostics → Script Debugger

---

**Q: An SLA is breaching even though the incident was On Hold. How do you fix it?**

1. Open the SLA Definition record for the affected SLA
2. Review the **Pause Condition** field — confirm it includes the On Hold state (`state = 3`)
3. If the On Hold state was added after the SLA was created, the pause condition may have never been updated
4. Fix: update the pause condition to include state 3 (On Hold)
5. For already-breached incidents: use retroactive SLA recalculation to credit the pause time back
6. Check the `task_sla` record: compare Business Elapsed vs. Actual Elapsed — if they match exactly (no pause time credited), the pause condition is not working

---

**Q: Discovery is running but CIs are not being created. What do you investigate?**

1. **Check the ECC Queue:** Look for input records from the affected MID Server — are there errors in probe results?
2. **Check Discovery Status:** The Discovery Status record shows probe counts — how many succeeded vs. failed
3. **Check credentials:** If SSH/WMI probes are failing with authentication errors, the credential may be expired or the target service may not be running (SSH disabled, WMI firewall blocked)
4. **Check IRE:** Review the IRE log for identification errors — the record may be created but matched to an existing CI rather than creating a new one
5. **Check CI class:** The discovered device may not be matching any classifier — ServiceNow does not know what CI class to create
6. **Check MID Server status:** Confirm the MID Server is "Up" and its Work Queue is not backed up

---

**Q: A Flow Designer flow is not triggering. How do you troubleshoot?**

1. **Check the trigger condition:** Open the flow, review the trigger type and conditions. Is the condition matching what you expect? Add a simple test: change the condition to always true and trigger manually to confirm the flow itself works.
2. **Check flow activation:** Is the flow Active? Flows must be set to Active to fire.
3. **Check the execution context:** Flows created in a scoped app only fire for records in that scope. If the trigger is on a global table, the flow must have cross-scope access.
4. **Review Flow Execution History:** Navigate to the flow, click "Execution Details" — all past execution attempts are logged here. If the flow was triggered but failed, the error details appear here.
5. **Check for conflicting flows:** Multiple flows on the same trigger/table can interact. Review all active flows on the same table/trigger combination.

---

**Q: How would you approach a ServiceNow implementation for a new client?**

A high-level approach:
1. **Requirements gathering:** Understand current state (existing tools, pain points, processes), desired future state, and success metrics. Identify stakeholders and executive sponsor.
2. **Platform setup:** Instance provisioning, SSO/LDAP configuration, User/Group import, basic admin configuration.
3. **CMDB foundation:** Import existing asset data. Configure Discovery if ITOM is in scope. Establish CSDM baseline.
4. **Core ITSM:** Incident, Problem, Change configuration — category taxonomy, assignment rules, SLA definitions, Service Catalog baseline.
5. **Portal / Self-service:** Service Portal or Employee Center setup, Virtual Agent for top use cases, Knowledge Base population.
6. **Integrations:** Identify and implement required integrations (LDAP/AD, monitoring tools, SCCM, etc.).
7. **Testing:** UAT with business stakeholders, ATF regression suite, performance testing for key workflows.
8. **Training and go-live:** Agent training, admin training, hyper-care support for first 4 weeks.
9. **Continuous improvement:** Performance Analytics dashboards, monthly review of adoption metrics, backlog of enhancements.

---

**Q: What is your approach when a stakeholder asks for a customization that can be done with OOTB configuration?**

Always evaluate OOTB capabilities first. My approach:
1. **Understand the requirement:** What outcome does the stakeholder actually need? Often the stated requirement ("I need a custom table") is a solution, not the real need.
2. **Demonstrate OOTB:** Show the stakeholder how the platform already handles this — UI Policies, Flow Designer, existing catalog functionality.
3. **Explain the trade-off:** Customization creates upgrade risk and maintenance burden. Configuration is upgrade-safe and maintainable by admins without developer involvement.
4. **When customization IS needed:** Document the requirement clearly, assess upgrade risk, build in a scoped app, write ATF tests, and document the customization thoroughly for future maintainers.

---

*Last updated: 2026 | Covers platform through Xanadu release | 50 questions across all major topic areas*
