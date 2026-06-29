# ServiceNow FSO — Financial Services Operations: Basics

> Sources: ServiceNow Docs, ServiceNow Community, Xceltrait, NewRocket, Accutive Fintech

---

## 1. What Is Financial Services Operations (FSO)?

**Financial Services Operations (FSO)** is ServiceNow's purpose-built industry solution for banks, insurance companies, and wealth management firms. It enables financial institutions to deliver connected service and operations experiences across customer-facing and back-office teams — powered by a unified workflow platform with Agentic AI.

**Core problem FSO solves:**
Financial institutions operate with fragmented systems — core banking platforms, CRMs, claims systems, lending platforms — where work moves between departments via email, spreadsheets, and manual handoffs. Customers can't see the status of their requests. Operations teams don't have full customer context. Compliance teams lack audit trails. FSO unifies all of this on one workflow platform.

**FSO is NOT a system of record.** Core banking systems (Temenos, FIS, Finacle), policy admin systems, and CRMs remain the source of truth. FSO is the **workflow and orchestration layer** — routing, tracking, automating, and connecting work across those systems.

### Key Principles
- **Workflow first, case-centric:** All work is structured as cases flowing through defined lifecycles
- **Rules-driven, event-based:** Business rules automate routing and assignment
- **Agentic AI-infused:** AI assists with classification, summarization, and repetitive automation
- **Built on CSM:** FSO extends Customer Service Management (CSM) with financial-services-specific data models and workflows

---

## 2. FSO Vertical Modules

FSO has three primary vertical solutions:

### 2a. Banking Operations (FSO for Banking)
Covers front-to-back banking operations:

| Banking Module | What It Does |
|---------------|-------------|
| **Card Operations** | Credit/debit card applications, transaction dispute processing, card lifecycle management |
| **Payment Operations** | Personal and business payment inquiries, payment claims, debit processing, wire transfers |
| **Loan Operations** | Personal and business loan origination, servicing, exception handling |
| **Deposit Operations** | Deposit account origination, servicing, and closure |
| **Treasury Operations** | End-to-end onboarding and management of treasury products |
| **Complaint Management** | Customer complaint intake, routing, tracking, and regulatory reporting |
| **Customer Lifecycle Operations** | KYC, identity verification, account onboarding/offboarding |

### 2b. Insurance Operations (FSO for Insurance)
Covers policyholder servicing and claims:

| Insurance Module | What It Does |
|-----------------|-------------|
| **Commercial Lines Claims** | Full claims lifecycle: first notice of loss (FNOL) → investigation → settlement |
| **Insurance Policy Operations** | Policy servicing, underwriting support, endorsements, renewals |
| **Policy Complaint Management** | Regulatory-compliant complaint tracking with audit trails |

### 2c. Wealth Management (FSO for Wealth)
Covers advisor-client operations:
- Client onboarding and KYC
- Account servicing and maintenance
- Advisor workspace with 360° client view
- Managed account operations

---

## 3. FSO Architecture — How It Works

```
CUSTOMER / ADVISOR / EMPLOYEE
         ↓
INTAKE CHANNELS
(Portal, Chat, Phone, Email, Core System Events, IoT)
         ↓
INTELLIGENT INTAKE
(Classify → Route → Create Case)
         ↓
FINANCIAL SERVICES WORKSPACE
(Agent/Banker/Adjuster processes the case)
         ↓
WORKFLOW ORCHESTRATION
(Automated steps, approvals, checks, integrations)
         ↓
BACK-OFFICE SYSTEMS
(Core Banking, Policy Admin, CRM, AML/KYC tools)
         ↓
RESOLUTION + CUSTOMER NOTIFICATION
```

### Workflow First, Case Centric
Every piece of work in FSO is a **Case**. Cases:
- Have a defined lifecycle (New → In Progress → Resolved → Closed)
- Group related tasks, documents, and decisions
- Track SLA compliance
- Provide full audit trail
- Can be viewed by customers through self-service portal

---

## 4. FSO Data Model

FSO extends the CSM data model with financial-services-specific layers:

### The "Who" — Customer Data Foundation (CDF)

| Entity | Description |
|--------|-------------|
| **Account** | B2B customer (a business, company, or institution) |
| **Contact** | Individual linked to a business account |
| **Consumer** | Individual B2C customer |
| **Household** | Group of related consumers (family unit) |
| **Party** | Flexible entity covering individuals, businesses, trusts |

### The "What" — Financial Product Model

| Entity | Description |
|--------|-------------|
| **Financial Product** | Template defining a banking offering (e.g., "Checking Account" product) |
| **Financial Account** | Specific customer instance of a product (e.g., "John's Checking #12345") |
| **Financial Service** | Service enabled on an account (e.g., wire transfer enabled on the account) |
| **Financial Transaction** | Individual transaction record (used for disputes, inquiries) |

### The "How" — Case Management

| Entity | Description |
|--------|-------------|
| **Case Type** | Specific category of customer request (extends CSM Case table) |
| **Service Definition** | Connects a customer need to the workflow/process that resolves it |
| **Task** | Individual action item within a case |
| **Approval** | Structured sign-off embedded in workflow |

---

## 5. Key Tables in ServiceNow FSO

| Table | What It Stores | Notes |
|-------|---------------|-------|
| `sn_fsc_account` | Financial Account records | Customer product instances |
| `sn_fsc_financial_product` | Financial Product catalog | Banking product templates |
| `sn_fsc_transaction` | Financial Transaction records | Used for disputes |
| `sn_fsc_card` | Card records | Credit/debit card details |
| `sn_fsc_loan` | Loan records | Loan-specific details |
| `sn_fsc_complaint` | Complaint records | Regulatory complaint tracking |
| `sn_fsc_claim` | Insurance Claim records | Claims lifecycle |
| `sn_fsc_policy` | Insurance Policy records | Policy details |
| `sn_customerservice_case` | Base CSM Case table | All FSO cases extend this |
| `customer_account` | CSM Account (B2B customer) | Inherited from CSM |
| `customer_contact` | CSM Contact | Individuals linked to accounts |
| `csm_consumer` | CSM Consumer | Individual B2C customers |

> **Key rule:** FSO Case Types are EXTENDED tables that inherit from `sn_customerservice_case`. Each banking/insurance application ships pre-built case types with domain-specific fields, workflows, and business rules.

---

## 6. Financial Services Workspace

The **Financial Services Workspace** is the primary interface for bankers, advisors, claims adjusters, and operations staff. It is a **single, persona-aware workspace** that serves multiple lines of business.

### Key Workspace Features
- **Landing Page/Dashboard:** Role-specific view — a banker sees their cases; a claims adjuster sees claims
- **360° Customer View:** All of a customer's accounts, cases, interactions, and history in one screen
- **Case Queue:** Prioritized work list filtered by role, LOB, and SLA urgency
- **Left Navigation:** Conditionally shown menu items based on persona (line of business)
- **Case Record Page:** Case details, related tasks, approvals, documents, and activity timeline
- **Page Variants:** Different case layouts per case type / persona — no need for multiple workspaces

### Workspace Architecture — "One Workspace, Every Line of Business"
Instead of separate workspaces per department, FSO uses **persona-aware configuration**:
- Dashboards displayed based on role/group membership
- Navigation menus show only relevant work types per LOB
- Record page variants show different layouts for different case types
- Same URL, different experience — controlled by user attributes

---

## 7. Case Management in FSO

Cases are the core unit of work in FSO. Unlike generic ITSM tickets, FSO cases carry rich financial context:

### Pre-Built Banking Case Types
- **Card Dispute** — fields: dispute amount, transaction date, card number, merchant
- **Payment Inquiry** — fields: payment date, amount, recipient account
- **Loan Servicing Request** — fields: loan number, request type, current balance
- **Account Maintenance** — fields: account number, change type, verification status
- **Complaint** — fields: complaint category, regulatory obligation, response deadline

### Case Lifecycle
```
NEW → ASSIGNED → IN PROGRESS → PENDING (awaiting info/approval) → RESOLVED → CLOSED
```

### SLA Management
- SLA clocks defined per case type (e.g., card disputes must resolve within regulatory deadlines — 5 business days for most)
- Regulatory SLA: CFPB requires card dispute resolution within 5 business days
- Breach notifications and escalations automated

---

## 8. Omnichannel Intake

FSO handles customer interactions from multiple channels — all routed into the same case management engine:

| Channel | How It Works |
|---------|-------------|
| **Self-Service Portal** | Customer logs into portal; submits case; tracks status ("Pizza Tracker" style) |
| **Chat / Virtual Agent** | AI-powered chat; intent recognition; auto-creates cases or resolves via self-service |
| **Phone / CTI** | Call center integration; screen pops customer record when call arrives |
| **Email** | Inbound email creates cases automatically via email flow |
| **Mobile App** | Customer mobile app for case submission and tracking |
| **Core System Events** | Automated: core banking event (failed payment) triggers case automatically |
| **Agentic AI** | Unstructured interactions converted to governed workflows by AI agents |

---

## 9. Agentic AI in FSO

FSO embeds AI capabilities at multiple levels:

| AI Feature | What It Does |
|-----------|-------------|
| **Case Summarization** | Auto-generates a summary of case history for agents picking up a case |
| **Intelligent Classification** | Routes incoming cases to correct team/queue based on content |
| **Recommended Actions** | Suggests next steps to agents based on case type and history |
| **Automated Steps** | Handles repetitive tasks (data lookups, form pre-fill) without human involvement |
| **Dispute Summary (GenAI)** | Uses generative AI to summarize dispute details and auto-populate resolution fields |
| **Agentic Contact Center** | AI agents handle common banking inquiries autonomously (balance inquiries, payment status) |

---

## 10. Integration with Core Banking Systems

FSO is the workflow layer — it integrates with, not replaces, existing systems:

| Integration Type | Examples |
|-----------------|---------|
| **Core Banking** | FIS, Temenos, Jack Henry, Finastra — pull account/transaction data |
| **Card Processing** | Visa DPS, TSYS, Mastercard — card status, transaction history |
| **AML/KYC Vendors** | LexisNexis, Refinitiv, Acuris — run checks, return results to FSO |
| **Payment Networks** | SWIFT, ACH, FedWire — payment status lookups |
| **CRM** | Salesforce — bidirectional customer data sync |
| **Document Management** | DocuSign, Adobe Sign — e-signatures embedded in workflows |
| **Policy Admin Systems** (Insurance) | Guidewire, Duck Creek, Applied Epic — policy/claims data |
| **ServiceNow Store** | Hundreds of pre-built connectors for financial services integrations |

---

## 11. Compliance and Regulatory Support

FSO embeds compliance requirements directly into workflows:

### Built-in Compliance Features
- **Audit trails:** Every case action, decision, and approval logged with timestamp and actor
- **Regulatory SLA tracking:** CFPB, GDPR, FCA, FINRA deadlines enforced as SLAs
- **Complaint regulatory reporting:** Complaint Management module generates regulatory-format reports
- **KYC/AML orchestration:** FSO orchestrates identity verification steps with third-party tools
- **Data privacy controls:** Role-based access enforced on sensitive financial data

### Key Regulations FSO Supports
| Regulation | Relevance |
|-----------|----------|
| **CFPB (Regulation E/Z)** | Card and payment dispute resolution timelines |
| **GDPR / CCPA** | Customer data privacy and right to access/deletion |
| **FCA / PRA** | UK financial services conduct and prudential standards |
| **FINRA** | Wealth management compliance and reporting |
| **DORA** | Digital operational resilience for EU financial firms |
| **BSA/AML** | Anti-money laundering workflow support |

---

## 12. FSO vs. CSM — Key Distinction

| Aspect | CSM (Customer Service Management) | FSO (Financial Services Operations) |
|--------|----------------------------------|-------------------------------------|
| **Industry** | Any industry | Financial services (banking, insurance, wealth) |
| **Data model** | Generic accounts, contacts, consumers | Financial accounts, products, transactions, policies, claims |
| **Case types** | Generic | Pre-built: card disputes, loan servicing, claims, complaints |
| **Compliance** | Generic SLAs | Regulatory SLAs (CFPB deadlines, FCA timelines) |
| **Integration** | General | Pre-built for core banking, card processors, AML tools |
| **Workspace** | CSM Agent Workspace | Financial Services Workspace |

**FSO builds on top of CSM.** You need CSM knowledge to work with FSO — FSO adds the financial-services-specific layer on top.

---

## 13. FSO Certification Paths

ServiceNow offers two FSO certification tracks:

1. **Banking and Wealth Management — FSO with CSM Professional Suite**
   - Covers: CSM core + Banking Service Management + Wealth Management
   - Prerequisite: CSM Implementer knowledge

2. **Insurance — FSO with CSM Professional Suite**
   - Covers: CSM core + Insurance Claims + Policy Operations
   - Prerequisite: CSM Implementer knowledge

**Training resources:**
- FSO Essentials course
- Banking & Wealth Essentials course
- Insurance Essentials course
- Now Learning platform (learning.servicenow.com)

---

## 14. Key FSO Metrics

| Metric | Definition |
|--------|-----------|
| **Self-Service Adoption Rate** | % of cases created via portal/chat vs. phone/email |
| **Case Deflection Rate** | % of customer inquiries resolved via self-service without creating a case |
| **CSAT** | Customer satisfaction scores post-case resolution |
| **SLA Adherence** | % of cases resolved within regulatory/contractual time |
| **Average Handling Time (AHT)** | Average time agent spends on a case |
| **First Contact Resolution (FCR)** | % of cases resolved on first contact |
| **Cost per Interaction** | Total operations cost divided by case volume |
| **Claims Handling Time** (Insurance) | Duration from FNOL to settlement |
| **Dispute Write-Off Reduction** (Banking) | Reduction in amounts written off through better dispute processing |
| **Net Promoter Score (NPS)** | Customer loyalty measurement |

---

## 15. Glossary

| Term | Definition |
|------|-----------|
| **FSO** | Financial Services Operations — ServiceNow's financial industry solution |
| **CSM** | Customer Service Management — the base platform FSO extends |
| **CDF** | Customer Data Foundation — the "Who" layer (accounts, contacts, consumers) |
| **Financial Account** | Customer's specific instance of a financial product |
| **Financial Product** | Template defining a banking/insurance offering |
| **Case Type** | FSO-specific extended case table (e.g., Card Dispute, Loan Servicing) |
| **Service Definition** | Links a customer need to the workflow that resolves it |
| **FNOL** | First Notice of Loss — the initial claim report in insurance |
| **KYC** | Know Your Customer — identity verification process |
| **AML** | Anti-Money Laundering — regulatory compliance process |
| **Household** | Group of related consumers treated as a unit |
| **Party** | Flexible entity covering any participant (individual, business, trust) |
| **Financial Services Workspace** | Primary FSO agent/banker/adjuster interface |
| **Agentic AI** | AI that autonomously performs tasks within defined guardrails |
| **CFPB** | Consumer Financial Protection Bureau — US regulator for consumer financial products |
| **Regulation E** | US federal regulation covering electronic fund transfer disputes |
