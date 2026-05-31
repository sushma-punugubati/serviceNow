# ServiceNow AI — Fundamentals & Complete Capability Guide

> Sources: ServiceNow Docs | Now Learning | ServiceNow Knowledge Conference | ServiceNow AI Product Documentation

---

## 1. The ServiceNow AI Portfolio — Overview

ServiceNow's AI strategy spans multiple capability layers, from ML-based automation to generative AI to autonomous agents:

```
LAYER 4: Autonomous AI Agents — multi-step, goal-directed workflows
LAYER 3: Generative AI (Now Assist) — LLM-powered summaries, drafts, code
LAYER 2: Conversational AI (Virtual Agent) — NLU chatbots for self-service
LAYER 1: Predictive/Analytical AI — ML classification, routing, anomaly detection
```

Each layer builds on the one below it — and all layers are governed by the **AI Control Tower**.

### The Unifying Principle

All ServiceNow AI capabilities share a common foundation:
- **Data:** CMDB, ITSM records, Knowledge Base, user profiles — ServiceNow's rich operational data
- **Context:** CSDM relationships, process context, organizational hierarchy
- **Governance:** AI Control Tower provides visibility, controls, and safety for all AI features

> The reason ServiceNow AI is uniquely powerful vs. generic AI tools: it has access to your structured operational data — not just documents. Now Assist can say "this incident is affecting the Payment Processing service (3 business stakeholders, SLA at 85%)" because it reads CSDM data, not just the ticket text.

---

## 2. Now Assist — Generative AI

### What is Now Assist?

**Now Assist** is ServiceNow's generative AI layer — powered by large language models (LLMs) — that adds AI-generated content to existing ServiceNow workflows. It does not replace agents or processes; it makes them faster by generating drafts, summaries, and suggestions that humans then review and act on.

### Now Assist Capabilities by Module

#### Now Assist for ITSM
| Feature | What it does |
|---------|-------------|
| **Incident Summarization** | Generates a concise natural-language summary of a long incident thread — who reported it, what happened, what was tried, current status |
| **Resolution Notes** | Drafts the resolution note based on work notes and actions taken — agent edits and approves |
| **Case Deflection** | Suggests KB articles and quick answers when an employee submits an incident |
| **Alert Summarization** | Summarizes a correlated alert group in Event Management — what is happening, affected CIs, business impact |
| **Change Summarization** | Summarizes a change record's implementation plan and risk for CAB reviewers |

#### Now Assist for HRSD
| Feature | What it does |
|---------|-------------|
| **Case Summarization** | Summarizes long HR case threads for agents picking up handed-off cases |
| **Draft Response** | Generates a first-draft response to the employee based on case context and KB |
| **KB Article Suggestions** | Suggests relevant KB articles to agents working a case |
| **Resolution Notes** | Drafts HR case closure notes |
| **Now Assist for Employee Center** | Generative responses in the Employee Center chat — employees ask HR policy questions in natural language and get AI-generated answers from the KB |

#### Now Assist for CSM
| Feature | What it does |
|---------|-------------|
| **Case Summarization** | Summarizes customer case history for agents |
| **Email Draft** | Drafts customer-facing response emails based on case context |
| **Sentiment Analysis** | Assesses customer sentiment from case text to flag at-risk customer relationships |

#### Now Assist for Creator
| Feature | What it does |
|---------|-------------|
| **Code Generation** | Generates JavaScript for Business Rules, Script Includes, Client Scripts from natural language description |
| **Flow Design** | Generates Flow Designer flows from a natural language description of the process |
| **Test Generation** | Creates ATF test cases from a description of what to test |
| **Documentation** | Generates inline code documentation and comments |

### How Now Assist Works — Technical Architecture

```
User action (open incident, click Summarize)
        ↓
Now Assist retrieves context (incident record, work notes, CSDM data, KB)
        ↓
Prompt constructed with context + task instruction
        ↓
LLM processes prompt (ServiceNow-hosted model or configured external LLM)
        ↓
Generated content returned to ServiceNow UI
        ↓
Human reviews, edits, and approves before any action
```

### LLM Options in ServiceNow

ServiceNow supports multiple LLM backends:
- **ServiceNow NowLLM:** ServiceNow's own hosted LLM — data stays within ServiceNow's tenant, no external data transmission
- **Azure OpenAI:** Connect to Azure-hosted GPT models via the AI Provider configuration
- **Other LLMs:** Via the AI Provider framework, organizations can configure connections to Anthropic Claude, Google Gemini, and other providers

**AI Provider configuration:** Set up in AI Administration → AI Providers. Defines which LLM to use, authentication, rate limits, and safety filters.

---

## 3. Virtual Agent — Conversational AI

### What is Virtual Agent?

**Virtual Agent** is ServiceNow's chatbot platform that enables employees and customers to resolve requests through a conversational interface — without contacting a human agent. It uses NLU (Natural Language Understanding) to interpret what users are asking and routes them to the correct resolution path.

### Virtual Agent Architecture

```
User types a message
        ↓
NLU Engine processes input → Intent classification → Entity extraction
        ↓
Topic matched (or fallback triggered)
        ↓
Topic flow executes (asks questions, looks up data, creates records)
        ↓
Resolution provided OR handoff to human agent with transcript
```

### NLU — Natural Language Understanding

**NLU** is the AI layer that converts free-text user input into structured intent + entity data:
- **Intent:** What the user wants to do ("reset password," "check ticket status," "request laptop")
- **Entity:** The specific details ("my ticket INC0012345," "a Dell XPS 15")

NLU is trained on example utterances — the more varied, high-quality training data, the better the model understands user input.

### Topic Design

**Topics** are the conversation flows that handle specific intents. Each topic has:
- **Trigger phrases:** Example utterances that activate this topic
- **Dialog steps:** Questions the bot asks, lookups it performs, actions it takes
- **Resolution:** What happens at the end (creates ticket, shows answer, escalates)

**Topic types:**
| Type | Purpose |
|------|---------|
| **Informational** | Answers a question without creating a record ("what is the VPN policy?") |
| **Fulfillment** | Creates a ticket, request, or case on behalf of the user |
| **Diagnostic** | Walks user through troubleshooting steps before escalating |
| **Status Check** | Looks up an existing ticket or request and reports status |

### Virtual Agent Integration with Now Assist

In recent releases, Virtual Agent topics can use **Now Assist** to generate responses for open-ended questions — instead of scripted topic flows, the VA can answer free-text questions using generative AI grounded in the knowledge base.

### Human Handoff

When Virtual Agent cannot resolve a request:
1. VA presents the option to "talk to an agent"
2. Conversation transcript is attached to the created case/incident
3. Case is routed to the appropriate human queue
4. The human agent sees the full VA conversation history — no need to ask the user to repeat themselves

---

## 4. Predictive Intelligence — Machine Learning

### What is Predictive Intelligence?

**Predictive Intelligence** applies machine learning to historical ServiceNow data to predict categorical outcomes — automatically classifying, routing, and suggesting actions on new records based on patterns in past records.

### Predictive Intelligence Use Cases

| Use Case | What it predicts | Training data |
|----------|-----------------|---------------|
| **Incident Category** | Auto-assign category/subcategory based on short description | Historical incidents with manually set categories |
| **Assignment Group** | Route to the correct team automatically | Historical incidents with their assignment groups |
| **Priority** | Suggest incident priority based on description and CI | Historical incidents with priority |
| **Change Risk** | Predict change risk level | Historical changes with their actual outcomes |
| **Case Category (CSM)** | Auto-categorize customer cases | Historical CSM cases |
| **Similarity** | Find similar past incidents to suggest resolution notes | All closed incidents |

### How Predictive Intelligence Works

1. **Data Collection:** ServiceNow accumulates historical records (minimum 400+ examples per category recommended)
2. **Model Training:** ML model trained on historical data — learns patterns correlating text/fields to outcomes
3. **Prediction:** When new record created, model predicts the outcome (category, routing, priority)
4. **Confidence Score:** Each prediction has a confidence score (0-100%) — high confidence predictions are auto-applied, low confidence shown as suggestions
5. **Feedback Loop:** When agents correct a prediction, the corrected data improves the model over time

### Confidence Thresholds

| Threshold | Behavior |
|-----------|---------|
| **Auto-apply:** 85%+ confidence | System automatically sets the predicted value without human review |
| **Suggest:** 60-84% confidence | Prediction shown as a suggestion — agent can accept or override |
| **No action:** Below 60% | Prediction not shown — too uncertain to be useful |

Thresholds are configurable based on the use case and risk tolerance.

---

## 5. AI Agents — Agentic AI

### What are AI Agents?

**AI Agents** in ServiceNow are autonomous AI-driven processes that can execute multi-step workflows, make decisions, and interact with multiple systems to complete a goal — without human intervention at each step.

Unlike traditional automation (which follows a fixed script), AI Agents:
- Interpret goals in natural language
- Break goals into steps dynamically
- Make decisions based on context
- Interact with external systems and ServiceNow APIs
- Escalate to humans when they encounter uncertainty

### AI Agent Components

| Component | Purpose |
|-----------|---------|
| **Agent Studio** | Build environment for creating AI Agents — define capabilities, knowledge, and decision logic |
| **AI Agent Fabric** | The runtime platform that orchestrates multi-agent workflows — multiple agents can collaborate on a complex task |
| **AI Control Tower** | Governance layer — monitor agent activity, set safety boundaries, audit decisions |
| **Skills** | Specific capabilities an agent has (e.g., "look up CMDB CI," "create incident," "send email") |
| **Playbooks** | Guided multi-step action sequences an agent follows for known scenarios |

### AI Agent Use Cases in ServiceNow

| Use Case | What the Agent does |
|----------|---------------------|
| **Incident Triage Agent** | Analyzes new incidents, looks up CMDB context, queries knowledge base, recommends or auto-applies category and assignment — all within seconds of incident creation |
| **Change Risk Agent** | Reviews a new Change Request, analyzes the affected CIs' history of incidents, evaluates the change plan against known failure patterns, recommends risk level and mitigation steps |
| **HR Onboarding Agent** | Monitors new hire Lifecycle Events, proactively checks task completion, escalates stalled tasks, answers employee questions about onboarding progress |
| **Procurement Agent** | Reviews purchase requests against policy, checks budget availability, routes for appropriate approval, creates PO records — all without human process management |
| **SecOps Triage Agent** | Analyzes security alerts, looks up threat intelligence, checks affected CI criticality, determines if an incident should be created and at what priority |

### The Difference Between Automation and AI Agents

| Traditional Automation (Flow Designer) | AI Agents |
|----------------------------------------|-----------|
| Fixed script — every step pre-defined | Dynamic — decides steps based on context |
| Cannot handle unexpected inputs | Can adapt to novel situations |
| Breaks on edge cases | Can escalate when uncertain |
| Requires human to design every scenario | Can be given a goal and figure out the steps |
| Fast and reliable for known scenarios | Better for open-ended, variable scenarios |

---

## 6. AI Control Tower — Governance

### What is AI Control Tower?

**AI Control Tower** (also called AI Control Center in some releases) is ServiceNow's governance platform for all AI features. It provides visibility into what AI is doing, controls for how AI can behave, and safety mechanisms to prevent harmful or inaccurate AI outputs.

### AI Control Tower Capabilities

| Capability | What it does |
|-----------|-------------|
| **AI Skill Management** | Enable/disable specific Now Assist and AI Agent skills per user group or persona |
| **Usage Monitoring** | Track which AI features are being used, by whom, how often |
| **Quality Monitoring** | Track AI output quality — thumbs up/down feedback, correction rates, escalation rates |
| **Safety Filters** | Content moderation — prevent AI from generating harmful, inappropriate, or sensitive content |
| **Model Configuration** | Configure which LLM backend to use for each AI skill |
| **Audit Trail** | Full log of all AI interactions — prompts, responses, feedback — for compliance |
| **Data Residency Controls** | Ensure AI processing stays within required geographic boundaries |

### AI Skills Management

Different teams get different AI capabilities based on their role:
- **ITSM agents:** Incident summarization, resolution note drafting, KB suggestions
- **HR agents:** Case summarization, draft response, Now Assist for Employee Center
- **Developers:** Now Assist for Creator (code generation, flow design)
- **Executives:** AI-generated reports and dashboards

Unused skills can be disabled to reduce cost and limit surface area.

### Responsible AI in ServiceNow

ServiceNow's responsible AI framework includes:
- **Transparency:** AI-generated content is labeled as AI-generated — users always know when AI produced content
- **Human-in-the-loop:** AI generates drafts and suggestions; humans review and approve before any action
- **Bias monitoring:** Predictive Intelligence models are monitored for demographic bias in routing and categorization
- **Explainability:** Predictive Intelligence shows which factors drove a prediction (why was this incident routed to Network team?)

---

## 7. Now Assist for Creator — AI-Assisted Development

### What is Now Assist for Creator?

**Now Assist for Creator** brings generative AI into the ServiceNow development experience. It allows developers and low-code practitioners to describe what they want in natural language and receive generated code, flows, or test cases.

### Creator AI Capabilities

| Feature | Input | Output |
|---------|-------|--------|
| **Script Generation** | "Write a Business Rule that fires on Incident insert and sets priority based on Impact × Urgency matrix" | Generated JavaScript Business Rule |
| **Flow Generation** | "Create a flow that gets triggered when a P1 Incident is created, sends notification to the on-call manager, and creates a Major Incident record" | Generated Flow Designer flow |
| **ATF Test Generation** | "Create a test for the catalog item submission flow that verifies approval is triggered for items over $500" | Generated ATF test case |
| **Code Explanation** | Select existing code → "Explain what this does" | Natural language explanation |
| **Code Refactoring** | "Refactor this Script Include to use GlideAggregate instead of GlideRecord for the count query" | Refactored code |

### How to Use Now Assist for Creator

- Available in **ServiceNow Studio** and **App Engine Studio**
- Invoked via the AI assistant panel or keyboard shortcut
- Generated code is inserted as a draft — developer reviews, tests, and commits
- All generated code goes through the same review and testing process as manually written code

---

## 8. Document Intelligence — AI for Documents

### What is Document Intelligence?

**Document Intelligence** uses AI to extract structured data from unstructured documents (PDFs, Word files, images, forms) and populate ServiceNow records automatically.

### Document Intelligence Use Cases

| Use Case | Document Type | Extracted Data |
|----------|---------------|----------------|
| **HR Document Processing** | Employment agreements, I-9 forms | Employee name, start date, role, signatures |
| **Contract Management** | Vendor contracts | Parties, terms, renewal dates, obligations |
| **Claims Processing (CSM)** | Insurance claim forms | Claim number, claimant, date of loss, amount |
| **Procurement** | Vendor invoices | Invoice number, line items, amounts, due dates |

### Document Intelligence Configuration

1. Upload sample documents to train the model (minimum 5-10 per document type)
2. Define the fields to extract (field name, data type, location hint)
3. Model learns extraction patterns from training documents
4. Deployed model automatically extracts data from new documents and populates ServiceNow fields

---

## 9. Health Log Analytics — AIOps

### What is Health Log Analytics (HLA)?

**Health Log Analytics** applies ML to log and metric data from IT infrastructure to detect anomalies before they cause outages. It is ServiceNow's AIOps capability.

### HLA Capabilities

| Capability | Description |
|-----------|-------------|
| **Log Anomaly Detection** | ML model learns normal log patterns; alerts when unusual patterns appear (error rate spike, new error type, unusual volume) |
| **Metric Anomaly Detection** | Detects deviations from normal metric baselines (CPU, memory, response time, error rate) without static thresholds |
| **Alert Clustering** | Groups related anomalies automatically — thousands of alerts collapsed into actionable clusters |
| **Probable Cause** | ML identifies the most likely root cause CI in a correlated alert cluster |
| **Noise Reduction** | Filters known-normal alert patterns — reduces alert fatigue significantly |

---

## 10. ServiceNow MCP — Model Context Protocol

### What is ServiceNow MCP?

**ServiceNow MCP (Model Context Protocol)** is an emerging capability that exposes ServiceNow platform actions and data to external AI tools and agents through a standardized protocol. It allows AI assistants (Claude, Copilot, etc.) to interact with ServiceNow — creating tickets, looking up records, triggering workflows — through natural language.

### MCP Use Cases

- An enterprise Copilot integration can create or update ServiceNow incidents via natural language: "Create a P2 incident for the network team about the VPN connection issue on the East Coast"
- External AI agents can query CMDB context: "What services depend on this server? What is the business impact of taking it offline?"
- Developer AI assistants can interact with ServiceNow development APIs: "Deploy this update set to the TEST instance"

### How MCP Works in ServiceNow

1. ServiceNow exposes a set of "tools" (actions and data queries) via the MCP server
2. External AI clients (Claude Desktop, Copilot Studio, custom agents) connect to the MCP server
3. The AI client calls ServiceNow tools as part of its reasoning and action execution
4. ServiceNow enforces authorization — the MCP connection uses an authenticated service account with scoped permissions

---

## 11. AI Implementation Roadmap — Typical Rollout Sequence

| Phase | Capabilities | Prerequisites |
|-------|-------------|---------------|
| **Phase 1 — Foundation** | Virtual Agent (scripted topics), Predictive Intelligence for categorization | Clean CMDB, good historical data (400+ records per category) |
| **Phase 2 — Generative AI Basics** | Now Assist summarization, resolution notes, KB suggestions | Phase 1 complete, AI Provider configured |
| **Phase 3 — Employee Self-Service** | Now Assist for Employee Center, advanced VA with NLU, search assist | KB well-populated and reviewed |
| **Phase 4 — Creator AI** | Now Assist for Creator, AI-assisted development | Developer team training |
| **Phase 5 — Autonomous Agents** | AI Agents for triage, routing, monitoring | CSDM aligned, strong data quality, AI Control Tower governance established |
| **Phase 6 — AI-Native** | MCP integrations, custom LLM connections, multi-agent workflows | Full platform maturity |
