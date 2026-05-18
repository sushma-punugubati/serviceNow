# ServiceNow CSM — Fundamentals & Study Guide

> Source: [ServiceNow Official Docs (Zurich)](https://www.servicenow.com/docs/r/zurich/customer-service-management/c_CustomerServiceManagement.html) | [CSM Fundamentals Community](https://www.servicenow.com/community/csm-articles/customer-service-management-csm-fundamentals/ta-p/2296920)

---

## 1. What is ServiceNow CSM?

**Customer Service Management (CSM)** is a ServiceNow application that helps organizations manage customer relationships and resolve issues end-to-end — not just at the service desk but across departments (IT, field service, operations, finance).

**Think of it this way:** In a traditional call center, a customer calls about a broken product. The agent logs the issue and closes the ticket. In ServiceNow CSM, that same issue can automatically trigger a field service work order, a change request in IT, and a replacement order — all connected, all trackable, all visible to the customer.

**Core value proposition:**
- Move from reactive ("fix problems when customers call") to **proactive** ("detect and fix before customers notice")
- Connect front-office (agents) with back-office (IT, operations, logistics)
- Deliver consistent service across email, phone, chat, and portal

---

## 2. Business Models Supported

| Model | Description | Example |
|-------|-------------|---------|
| **B2B** (Business-to-Business) | Service company supports corporate clients | Software company supporting enterprise accounts |
| **B2C** (Business-to-Consumer) | Service company supports individual consumers | Telecom supporting individual subscribers |
| **B2B2C** | Corporate client's end-customers | Bank servicing both the bank (B2B) and bank's retail customers (B2C) |
| **Internal** | Company's own employees | IT helpdesk for employees |

---

## 3. Core Concepts Explained Simply

### 3.1 Account
An **Account** is a company/organization that buys your services. Think of it as the **customer company record**.

- Stored in table: `customer_account`
- Extends: `core_company`
- Has parent/child hierarchy (e.g., a parent company with subsidiaries)
- Can be a **customer account** or a **partner account**

**Example:** Autovity InfoTech is an Account. They have 500 employees using your software.

### 3.2 Contact
A **Contact** is a specific person who works at an Account — the human you actually talk to.

- Stored in table: `customer_contact`
- Extends: `sys_user` (they get a login!)
- One account can have **many contacts**
- One contact belongs to **one account** (but can be associated across account hierarchy)

**Example:** John Smith is a Contact at Autovity InfoTech. He's the IT Manager who calls when systems are down.

### 3.3 Consumer (B2C)
A **Consumer** is an individual customer (not tied to a company).

- Stored in table: `csm_consumer`
- Used in B2C or B2B2C models
- Has its own portal login

**Example:** Sarah is a retail bank customer with a savings account. She's a Consumer.

> **Key Exam Tip:** Contacts = B2B (company employees). Consumers = B2C (individuals). This distinction is heavily tested.

### 3.4 Partner
A **Partner** is a third party that sells/supports YOUR products on your behalf to their own customers.

- Partners can **create and manage cases for their customer accounts**
- Partners have their own users (partner contacts)

**Example:** A reseller company that sells your software to small businesses and handles Tier-1 support on your behalf.

### 3.5 Case
A **Case** is the primary unit of work in CSM — it's how customer issues are tracked from start to finish.

- Table: `sn_customerservice_case`
- Linked to: Account/Consumer, Contact, Product, Entitlement, Contract
- Has a lifecycle: **New → Open → In Progress → Pending → Resolved → Closed**

**Example:** John at Autovity calls because their software license expired. A Case is created, linked to their account, contract, and the specific product. The agent works it, escalates if needed, and closes it.

### 3.6 Case Task
A **Case Task** is a sub-task created within a case for specific actions that need to be completed.

- Table: `sn_customerservice_task`
- Multiple tasks can exist per case
- Can be assigned to different teams/individuals

**Example:** A case for a broken server might have three tasks: "Diagnose hardware," "Replace component," "Test and verify."

### 3.7 Product & Product Model
- **Product Model**: The template/catalog of what you sell (e.g., "Enterprise License v2")
- **Sold Product**: A specific instance sold to a customer (e.g., "Autovity bought 50 Enterprise Licenses on Jan 2024")
- **Install Base Item**: The configured, deployed instance of a product at a customer site

> **Key Difference from ITSM:** CSM uses the **Product table** (not the ITSM Asset table) and the **Product Model table** (not the ITSM Asset Model table).

### 3.8 Entitlement
An **Entitlement** defines what level of support a customer is entitled to receive — based on what they bought.

- Defines: support type, communication channels allowed, SLA tier
- Tracked by: cases (number of cases allowed) or hours (support hours remaining)
- Linked to: sold products, install base items, accounts, or contracts

**Example:** "Gold Support — unlimited cases, phone + email + chat, 4-hour response SLA"

### 3.9 Service Contract
A **Service Contract** is the formal agreement between your company and the customer that specifies the entitlements, coverage period, and SLAs.

- A contract contains **multiple entitlements**
- Linked to specific accounts and their products

**Example:** Autovity signed a 3-year contract covering their 50 Enterprise Licenses with Gold Support entitlement.

### 3.10 Interaction
An **Interaction** represents a single customer communication event (a phone call, a chat session, an email exchange).

- Interactions can create cases or link to existing cases
- Variants: Email Interaction, Chat Interaction, Voice Interaction

**Example:** John calls support. That phone call is an Interaction. During the call, the agent creates a Case linked to this Interaction.

---

## 4. Key Tables Reference

| Table Name | Table Label | Purpose |
|-----------|-------------|---------|
| `customer_account` | Account | Customer companies |
| `customer_contact` | Contact | People at customer companies |
| `csm_consumer` | Consumer | Individual B2C customers |
| `sn_customerservice_case` | Case | Core case/issue record |
| `sn_customerservice_task` | Case Task | Sub-tasks within a case |
| `sn_customerservice_product` | Product | Products/services sold |
| `sn_customerservice_productmodel` | Product Model | Product catalog templates |
| `sn_customerservice_soldproduct` | Sold Product | Products sold to specific customers |
| `sn_customerservice_install_base` | Install Base Item | Deployed product instances |
| `sn_customerservice_contract` | Service Contract | Customer support agreements |
| `sn_customerservice_entitlement` | Entitlement | Support coverage definitions |
| `interaction` | Interaction | Customer communication records |
| `csm_consumer_user` | Consumer User | Portal-registered consumers |

---

## 5. Case Lifecycle

```
NEW → OPEN → IN PROGRESS → PENDING → RESOLVED → CLOSED
```

| State | Description |
|-------|-------------|
| **New** | Case just created, not yet assigned |
| **Open** | Assigned to an agent, work started |
| **In Progress** | Actively being worked |
| **Pending** | Waiting on customer or third party |
| **Resolved** | Solution provided, awaiting customer confirmation |
| **Closed** | Final state — case officially done |

> **Exam Tip:** "Closed" is the **final/terminal state**, not "Resolved."

---

## 6. Roles & Responsibilities

### Internal (Agent-Side) Roles

| Role Name | Role ID | What They Can Do |
|-----------|---------|-----------------|
| CSM Agent | `sn_customerservice_agent` | Create, view, edit cases; support customers via channels |
| CSM Manager | `sn_customerservice_manager` | Manage agents/groups, override actions, admin skills |
| CSM Admin | `sn_customerservice_admin` | Full configuration access |

### External (Customer-Side) Roles

| Role Name | Role ID | What They Can Do |
|-----------|---------|-----------------|
| Customer | `sn_customerservice.customer` | Create and view own cases, see their assets |
| Customer Case Manager | `sn_customerservice.customer_case_manager` | Manage all cases for their account |
| Customer Admin | `sn_customerservice.customer_admin` | Full account data access |
| Partner | `sn_customerservice.partner` | Create cases for their accounts or customer accounts |
| Partner Admin | `sn_customerservice.partner_admin` | Manage users across partner + customer accounts |
| Consumer | `sn_customerservice.consumer` | B2C equivalent of Customer role |

---

## 7. CSM Channels (Omnichannel)

Customers can reach support through multiple channels, all funneling into the same case management system:

| Channel | Description |
|---------|-------------|
| **Email** | Email creates/updates cases automatically |
| **Phone/Voice** | Agent logs case during call; CTI integration possible |
| **Chat** | Live chat via Customer Portal or Virtual Agent |
| **Virtual Agent** | AI-powered chatbot for self-service |
| **Customer Portal** | Self-service web portal — log cases, track status, search KB |
| **Walk-up** | In-person service at service desk kiosks |
| **Communities** | Peer-to-peer forums for collaborative support |

---

## 8. CSM vs ITSM — Key Differences

| Aspect | ITSM | CSM |
|--------|------|-----|
| **Who it serves** | Internal employees | External customers |
| **Core record** | Incident | Case |
| **Customer entity** | User (sys_user) | Account + Contact / Consumer |
| **Product tracking** | Asset / Asset Model | Sold Product / Product Model |
| **Agreements** | OLA/SLA | Service Contracts + Entitlements |
| **Portal** | Service Portal | Customer Service Portal |
| **Self-service** | Knowledge + Catalog | Knowledge + Communities + Portal |

---

## 9. Key Modules & Features

### 9.1 CSM Configurable Workspace (Agent Workspace)
The modern agent interface replacing classic UI. Provides:
- **Form Ribbon**: Quick access to case actions (calls, emails, SLA details, timeline)
- **Contextual Side Panel**: Lookup and verify contacts/consumers, related records
- **Now Assist**: AI-suggested next actions, case summaries
- **Playbooks**: Step-by-step guided resolution paths for agents

### 9.2 Customer Service Portal
Self-service portal for customers/consumers:
- Submit and track cases
- Search knowledge base articles
- Participate in community forums
- Manage their profile and products

### 9.3 Major Issue Management
For widespread issues affecting multiple customers simultaneously:
- Creates a **Major Case** that links to many individual cases
- Sends bulk notifications to affected customers
- Allows mass case closure when resolved
- Tracks impact and escalation severity

**Example:** A cloud provider's data center goes down, affecting 500 customers. One Major Case is created, 500 customer cases are linked, and automated updates go to all affected accounts.

### 9.4 Guided Decisions (Decision Trees)
Interactive troubleshooting guides that help agents diagnose issues step-by-step, like a flowchart:
- Agent answers yes/no questions
- System guides to the most likely resolution
- Reduces training time for new agents

### 9.5 Advanced Work Assignment (AWA)
Automatically routes incoming work items to the best available agent based on:
- Agent skills and capacity
- Channel type (chat, case, walk-up)
- Queue configuration

**Work Item**: A single piece of work assigned to an agent in AWA.

**AWA Service Channels for CSM:** Walk-up, Chat, Case (key exam fact: NOT email or telephony directly)

### 9.6 Escalation Management
Cases and accounts can be escalated with:
- Configurable severity levels
- Escalation templates
- Automatic notifications to senior agents/managers
- De-escalation workflows

### 9.7 Proactive Customer Service (ITOM Integration)
CSM + ITOM integration enables:
- Monitoring services/products automatically
- Creating cases when alerts trigger (before customer calls)
- Notifying customers proactively about known issues
- Closing cases automatically when the issue resolves

---

## 10. CSM Data Model Relationships

```
Account
  ├── Contacts (many)
  ├── Service Contracts
  │     └── Entitlements (many)
  ├── Sold Products
  │     └── Install Base Items
  └── Cases
        ├── Case Tasks
        ├── Interactions (email, chat, voice)
        └── Knowledge Articles (linked)

Consumer (B2C)
  ├── Consumer Profile
  ├── Cases
  └── Sold Products
```

---

## 11. Implementation Methodology

ServiceNow uses the **Now Create** methodology for CSM implementations (not Agile, SIM, or OCM — this is a tested exam fact).

**Typical Implementation Steps:**
1. Create agent groups
2. Assign agents to groups
3. Configure products and product models
4. Create accounts and partners
5. Add contacts for accounts
6. Create service contracts linked to accounts
7. Create and associate entitlements
8. Enable contact portal login
9. Test case creation through customer portal
10. Route cases to agents for resolution

---

## 12. Important Plugins

| Plugin | Purpose |
|--------|---------|
| `com.sn_customerservice` | Core CSM plugin — requires admin to activate |
| Customer Service Portal | Self-service portal for external customers |
| Virtual Agent | AI chatbot for automated case deflection |
| Advanced Work Assignment | Intelligent case/interaction routing |
| Performance Analytics | CSM dashboards and KPI tracking |
| Field Service Management | For cases requiring on-site work |

---

## 13. SLA & Entitlement Management

- SLAs in CSM are configured per **entitlement type** and **case priority**
- **Entitlement verification** checks if a customer is eligible for support before a case is created
- **Breach alerts** automatically notify managers when SLA thresholds are approaching
- Two tracking methods: **case-based** (X cases per contract period) or **hour-based** (X support hours)

---

## 14. Knowledge Management Integration

- Agents can search and link KB articles directly from a case
- Portal customers search KB before/instead of creating a case (**case deflection**)
- AI can suggest relevant articles based on case description
- Agents can create/publish articles from resolved cases

---

## 15. Glossary of Key Terms

| Term | Simple Explanation |
|------|-------------------|
| **Case** | A customer issue or request being tracked |
| **Account** | A corporate customer (the company) |
| **Contact** | A person at that company |
| **Consumer** | An individual customer (B2C) |
| **Entitlement** | What support the customer is entitled to |
| **Service Contract** | The legal agreement defining support terms |
| **Sold Product** | A specific product a customer purchased |
| **Install Base Item** | The deployed/configured product at customer site |
| **Interaction** | A single communication event (call, chat, email) |
| **Playbook** | Step-by-step agent guide for resolving a case type |
| **Major Case** | A case covering a widespread issue affecting many customers |
| **AWA** | Advanced Work Assignment — smart routing engine |
| **Case Deflection** | Customer resolves issue via self-service (no case created) |
| **Omnichannel** | Supporting customers across all channels in one system |
| **CTI** | Computer Telephony Integration — links phone system to ServiceNow |
| **Now Create** | ServiceNow's implementation methodology |
