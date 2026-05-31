# ServiceNow AI — Interview Questions & Answers

---

## PART 1: Now Assist and Generative AI

**Q1: What is Now Assist and how does it differ from traditional ServiceNow automation?**
**A:** Now Assist is ServiceNow's generative AI layer — powered by large language models — that generates natural language content (summaries, drafts, suggestions, code) to augment existing ServiceNow workflows.

**The key difference from traditional automation:**
- **Traditional automation:** Deterministic — same input always produces the same output. Flow Designer executes the exact steps you define.
- **Now Assist:** Probabilistic — generates contextually appropriate content from unstructured data. Can summarize a 50-comment incident thread, draft a response email, or generate code from a description — tasks that rule-based automation cannot do.

**What Now Assist does NOT do:** Make decisions, take actions, or modify records autonomously. It generates content that humans review and approve. The human stays in the loop.

---

**Q2: What are the different Now Assist skills available in ServiceNow?**
**A:**

| Module | Skills |
|--------|--------|
| **ITSM** | Incident summarization, resolution note drafting, KB article suggestions, alert summarization, change summarization |
| **HRSD** | Case summarization, draft response, resolution notes, Now Assist for Employee Center chat |
| **CSM** | Case summarization, email drafting, sentiment analysis |
| **Creator** | Code generation, flow generation, ATF test generation, code explanation, refactoring |
| **Search** | Semantic search — finds records by meaning, not just keyword match |
| **ITOM** | Alert summarization with business service impact context |

Skills are enabled/disabled per user group via AI Control Tower — different teams get different AI capabilities based on their role.

---

**Q3: How is Now Assist configured in a ServiceNow instance?**
**A:**
1. **Configure AI Provider:** In AI Administration → AI Providers, define the LLM connection (ServiceNow NowLLM, Azure OpenAI, or other). Enter authentication details, configure rate limits and safety filters.

2. **Enable AI Skills:** In AI Control Tower → Skill Management, enable the specific Now Assist skills for the relevant user groups (ITSM agents get incident summarization, developers get Creator skills, etc.)

3. **Configure prompts (optional):** Each Now Assist skill has a default prompt template. These can be customized — for example, add organization-specific instructions to the incident summary prompt ("always include the SLA status and business service in the summary")

4. **Test and validate:** Use the built-in AI testing tools to review sample outputs before rollout. Share with pilot users for feedback.

5. **Monitor via AI Control Tower:** After rollout, monitor usage, thumbs-up/down feedback, and correction rates to measure quality.

---

**Q4: What is the difference between Virtual Agent and Now Assist?**
**A:**
| Virtual Agent | Now Assist |
|---------------|-----------|
| Conversational chatbot — structured dialog flow | Generative AI — produces free-form text |
| Handles specific intents via topics (scripted paths) | Handles open-ended questions with generated responses |
| NLU classifies intent and routes to the right topic | LLM generates a response directly from context |
| Best for: task completion (create ticket, check status) | Best for: answering complex questions, summarizing content |
| Predictable, deterministic outputs | Variable outputs — probabilistic |
| Available since early ServiceNow releases | Introduced in Washington/Xanadu+ releases |

**In practice:** They are increasingly combined — Virtual Agent handles the structured task completion (creating tickets, looking up records), while Now Assist handles the open-ended Q&A within the same chat interface.

---

**Q5: What is the AI Control Tower and why is it important?**
**A:** AI Control Tower is ServiceNow's governance platform for all AI features — the single pane of glass for managing, monitoring, and controlling AI across the instance.

**Key capabilities:**
- **Skill Management:** Enable/disable AI skills per user group — control who has access to what AI feature
- **Usage Monitoring:** See which AI features are used, by whom, how often
- **Quality Tracking:** Track thumbs-up/down feedback and correction rates — measure AI output quality over time
- **Safety Filters:** Content moderation — prevent AI from generating harmful content or exposing sensitive data
- **Model Configuration:** Choose which LLM backend powers each skill
- **Audit Trail:** Log all AI interactions for compliance

**Why it matters:** Without governance, AI adoption becomes ungoverned and trust erodes quickly. AI Control Tower lets organizations deploy AI confidently — they can measure quality, detect problems early, and maintain compliance with data policies.

---

**Q6: What is Predictive Intelligence and how is it configured?**
**A:** Predictive Intelligence applies ML to historical ServiceNow data to predict categorical outcomes on new records — auto-classifying incidents, routing cases, suggesting priorities.

**Configuration steps:**
1. **Define the Solution:** What do you want to predict? (e.g., "Incident assignment group based on short description and category")
2. **Identify training data:** Historical records with the outcome field populated — minimum 400 records per class recommended
3. **Train the model:** ServiceNow trains the ML model on historical data — this runs as a scheduled job
4. **Evaluate accuracy:** Review precision/recall per class. If accuracy is below 60%, investigate data quality issues
5. **Configure thresholds:** Set auto-apply threshold (e.g., 85% confidence → auto-set) and suggest threshold (60-84% → show as suggestion)
6. **Enable the solution:** Activate the Predictive Intelligence solution — it begins scoring new records on creation/update

**Common pitfall:** Training data quality is more important than quantity. 400 records where "Network" incidents are correctly labeled as "Network" outperforms 4,000 records with inconsistent labeling.

---

**Q7: How does ServiceNow's NLU differ from traditional keyword matching in Virtual Agent?**
**A:**
| Keyword Matching | NLU (Natural Language Understanding) |
|-----------------|--------------------------------------|
| Matches exact keywords or phrases | Understands meaning and intent |
| "VPN not working" matches "VPN" | "I cannot connect to the company network remotely" still matches VPN intent |
| Rigid — misses variations | Flexible — handles paraphrasing and synonyms |
| No entity extraction | Extracts entities: "my ticket from last Tuesday" → ticket lookup + date |
| Simple string matching | ML model trained on language |

**Why NLU matters:** Employees do not phrase requests consistently. NLU handles the natural variation in how people ask for the same thing — increasing Virtual Agent resolution rates significantly vs. keyword-based topics.

---

## PART 2: AI Agents and Advanced AI

**Q8: What are AI Agents in ServiceNow and what makes them different from Flow Designer?**
**A:** AI Agents are autonomous AI-driven processes that interpret goals, plan steps, make decisions, and execute multi-system workflows without a human scripting every step.

**Key differences from Flow Designer:**
| Flow Designer | AI Agents |
|--------------|-----------|
| Every step pre-defined by the developer | Agent decides steps dynamically based on goal and context |
| Fixed paths — cannot handle unexpected situations | Adapts to novel inputs and edge cases |
| Fails on anything outside the defined scope | Can escalate to humans when uncertain |
| Fast, reliable, predictable | More flexible, handles ambiguity |
| Suitable for: known, repeatable processes | Suitable for: variable, complex, judgment-intensive tasks |

**Example:** A Flow Designer flow for incident routing has explicit rules for every category/subcategory combination. An AI Agent for incident routing reads the incident, looks up CMDB context, checks recent KB articles, and makes a routing decision based on the full context — even for categories not explicitly programmed.

---

**Q9: What is Agent Studio and what can you build with it?**
**A:** **Agent Studio** is the ServiceNow development environment for building AI Agents. It provides:

- **Capability definition:** What can this agent do? (Look up CMDB, create incidents, send notifications, query external APIs)
- **Skill configuration:** Link the agent to specific actions, API calls, and data sources
- **Decision logic:** Define when the agent should escalate vs. act autonomously
- **Testing environment:** Test the agent against sample scenarios before deployment
- **Monitoring:** Track agent decisions, actions taken, escalation rate

**Things you can build in Agent Studio:**
- Incident triage agents that classify and route without human intervention
- Proactive monitoring agents that watch CMDB for configuration drift and create change tasks
- Employee onboarding agents that monitor lifecycle event task completion and proactively unblock stalls
- SecOps triage agents that assess alert severity and determine investigation priority

---

**Q10: How does AI Agent Fabric work?**
**A:** **AI Agent Fabric** is the orchestration layer that allows multiple AI Agents to collaborate on complex tasks — each agent handles its area of expertise, and Agent Fabric coordinates the handoff between agents.

**Example — Major Incident Response:**
1. **Alert Agent** detects correlated alerts in Event Management — determines a P1 condition exists
2. **Impact Analysis Agent** queries CMDB to identify affected business services and stakeholders
3. **Communication Agent** drafts stakeholder notifications based on impact analysis
4. **Resolution Agent** searches knowledge base for similar past incidents and suggests workaround
5. Agent Fabric coordinates all four agents in sequence — the overall task is "initiate major incident response" and each specialist agent contributes its part

This is analogous to a human team: the monitoring person, the impact analyst, the communications person, and the resolver each do their part on a P1 — Agent Fabric makes AI do the same.

---

**Q11: What is ServiceNow MCP and why does it matter?**
**A:** ServiceNow **MCP (Model Context Protocol)** is a standard protocol that exposes ServiceNow platform actions and data to external AI tools and agents. It allows AI assistants (like Claude, Copilot, or custom enterprise AI tools) to interact with ServiceNow through natural language.

**Why it matters:**
- Employees using enterprise AI assistants can create ServiceNow tickets, check incident status, or look up CMDB data without leaving their AI tool
- External AI agents can use ServiceNow as a data source and action executor as part of larger multi-system workflows
- Developers using AI coding assistants can interact with ServiceNow development APIs directly from their IDE

**How it works:** ServiceNow exposes "tools" via the MCP server (e.g., "create_incident," "get_ci_details," "query_knowledge_base"). External AI clients discover and call these tools as part of their reasoning. ServiceNow enforces authentication and authorization — only permitted actions by permitted users.

---

## PART 3: Implementation and Configuration

**Q12: What data prerequisites are needed before deploying Now Assist?**
**A:**
1. **Knowledge Base quality:** Now Assist grounds its answers in the KB. KB articles must be current, well-written, and cover the topics where AI responses are expected. Articles over 2 years old without review are likely to produce outdated AI responses.

2. **CSDM alignment:** Now Assist incident summarization is significantly more valuable when it can include business service context. Without CSDM Application Service data, summaries are technically accurate but lack business impact context.

3. **Historical data quality:** For Predictive Intelligence (which often runs alongside Now Assist), training data quality directly determines prediction accuracy. Clean historical data is required.

4. **AI Provider:** An LLM must be configured and accessible. ServiceNow NowLLM or Azure OpenAI must be set up in AI Administration before any Now Assist skill can function.

5. **User persona mapping:** AI Control Tower skill assignments require defined user groups. ITSM agents must be in a group, HR agents in another — so skills can be appropriately targeted.

---

**Q13: How do you measure the success of a Now Assist deployment?**
**A:** Key metrics to track in AI Control Tower and PA:

| Metric | What it measures | Target |
|--------|-----------------|--------|
| **Adoption rate** | % of eligible agents using the AI skill | >70% within 3 months |
| **Thumbs-up rate** | % of AI outputs rated positively | >75% |
| **Correction rate** | % of AI drafts modified before use | <30% for mature deployments |
| **Escalation rate (VA)** | % of VA conversations requiring human handoff | Trending downward over time |
| **Resolution time** | Change in mean time to resolve incidents after AI deployment | Measurable reduction |
| **KB deflection** | % of inquiries resolved by AI-suggested KB without creating a ticket | Track before/after |

**Important:** Measure against a baseline. Capture metrics 4 weeks before Now Assist rollout and compare to 8 weeks after. Without a baseline, it is impossible to demonstrate value.

---

**Q14: How does Now Assist handle sensitive data?**
**A:** ServiceNow provides several mechanisms for sensitive data protection:

1. **Data Masking:** Configure specific fields to be excluded from AI prompts — HR records containing salary, medical information, or PII can be masked before the prompt is sent to the LLM

2. **Field-Level Exclusions:** In the AI Skill configuration, specify which fields should NOT be included in the context sent to the LLM

3. **ServiceNow NowLLM:** For maximum data security, use ServiceNow's own hosted LLM — all prompts and responses stay within ServiceNow's infrastructure, nothing sent to external providers

4. **Data Residency:** AI Control Tower includes data residency controls to ensure prompts are only processed in approved geographic regions

5. **Audit Trail:** All LLM interactions are logged — who requested what, what prompt was sent, what response was returned — for compliance and investigation

---

**Q15: How do you set up Virtual Agent NLU for a new set of topics?**
**A:**
1. **Define intents:** List the top 10-20 things users will ask. Group related requests into intents (e.g., "Password Reset," "VPN Issues," "New Laptop Request").

2. **Write training utterances:** For each intent, write 20-30 example phrases users might type — vary the vocabulary, length, and phrasing. More diverse utterances = better NLU model.

3. **Identify entities:** What specific information needs to be extracted? (Ticket number, software name, date) Define entity types and example values.

4. **Train the NLU model:** In the NLU Workbench, upload intents and utterances, then train the model. Review precision/recall metrics per intent.

5. **Build topic flows:** For each intent, build the conversation flow — what questions does the bot ask, what lookups does it perform, what action does it take?

6. **Test extensively:** Use the test console to submit varied phrases and confirm correct intent classification and entity extraction.

7. **Deploy and monitor:** Activate the topics. Monitor intent classification accuracy and escalation rates. Add training utterances for any misclassified inputs.

---

**Q16: What is the difference between Now Assist summarization and traditional scripted summaries?**
**A:**
| Traditional Scripted Summary | Now Assist Summarization |
|-----------------------------|--------------------------|
| Developer writes a script that extracts specific fields and formats them | LLM reads the full record — all work notes, description, fields — and generates natural language |
| Only captures what the script was programmed to extract | Captures nuance, context, and important details the script would miss |
| Same template for every record | Contextually tailored to each specific record |
| Immediately readable — no variability | Requires quality monitoring — occasional hallucinations possible |
| No LLM cost | Consumes LLM API calls — has cost implications at scale |

**When scripted summaries are better:** For standardized, high-volume outputs where consistency and cost matter more than contextual richness.

**When Now Assist is better:** For complex, varied records where a human reader would benefit from a genuinely intelligent summary — major incidents, complex HR cases, long customer cases.
