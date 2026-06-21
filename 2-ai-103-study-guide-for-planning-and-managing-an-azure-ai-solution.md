# AI-103 Study Guide for Planning and Managing an Azure AI Solution

## Scope and exam framing

This guide covers the **“Plan and manage an Azure AI solution”** domain of AI-103, which Microsoft lists at **25–30%** of the exam. The official skills outline for this domain includes model and service selection, retrieval and indexing, Foundry setup and deployment, CI/CD integration, monitoring and security, and responsible AI governance for generative AI and agentic systems. Microsoft also notes that **most exam questions target generally available features**, but **preview features can appear if they are commonly used**, which matters because several Foundry capabilities in this area are still preview.

The best way to study this domain is to think like an architect and platform owner, not just like a prompt engineer. Microsoft describes the target audience as an Azure AI engineer who **builds, manages, and deploys agents and AI solutions** with [Microsoft Foundry](https://learn.microsoft.com/en-us/azure/foundry/what-is-foundry), collaborating with architects, DevOps engineers, and security engineers. In practice, that means AI-103 expects you to justify **why** a particular service, model, deployment type, identity pattern, or monitoring strategy fits a scenario.

A useful mental model is that **Microsoft Foundry** is the main application factory, while **[Azure AI Search](https://learn.microsoft.com/en-us/azure/search/)**, **Foundry Tools**, **Azure Monitor/Application Insights**, **Azure API Management via AI Gateway**, and sometimes **Azure Machine Learning** extend it for retrieval, multimodal extraction, observability, governance, and classical ML explainability. Foundry itself is positioned as a unified Azure platform that brings together **agents, models, tools, tracing, monitoring, evaluations, RBAC, networking, and policy controls** under a single management model.

## Choosing models, Foundry services, and tools

Start with the **workload**, then select the **model** and the **service**. Foundry’s model catalog now includes **more than 1,900 models**, spanning foundation models, reasoning models, small language models, multimodal models, domain-specific models, and industry models. The catalog includes both **Foundry Models sold by Azure** and **models from partners and community**, so exam questions can test both the capability fit and the governance implications of those two sourcing patterns.

For **deep reasoning, multi-step planning, hard coding, research, and agentic orchestration**, favor reasoning-oriented models such as the **o-series** or **GPT-5 family**. Microsoft’s guidance explicitly says reasoning models spend more time processing a request and are especially strong in **science, coding, and math**, while the GPT-5 choice guide recommends GPT-5 for **complex planning, analysis, synthesis, and agentic tool calling**. For **real-time chat, customer support, lightweight summarization, and cost-sensitive high-throughput workloads**, favor **GPT-4.1** or smaller/faster variants.

For **vision or multimodal understanding**, use a **multimodal model** or a multimodal search pipeline rather than forcing a text-only pattern. Microsoft states that vision-enabled chat models can analyze images and answer questions about them, and current supported vision-capable families include **o-series**, **GPT-5**, **GPT-4.1**, **GPT-4.5**, and **GPT-4o**. For AI-103, memorize the distinction that a multimodal **model** handles cross-modal reasoning at inference time, while Azure AI Search multimodal pipelines handle **ingestion and retrieval** across text and images.

For **small language models**, think **latency, throughput, footprint, and constrained environments**. Microsoft’s model leaderboards evaluate both **LLMs and SLMs** across quality, safety, cost, and throughput, and Foundry’s guidance encourages using the leaderboard and side-by-side comparisons to pick the best model for a scenario instead of defaulting to the biggest model. In exam terms, an SLM is often the right answer when the scenario emphasizes **cheap, fast, narrow, or embedded** inference rather than maximal reasoning depth.

For **service selection**, keep this mapping in mind:

| Need | Best-fit service or capability | Why it fits |
|---|---|---|
| Chat completion, reasoning, embeddings, model hosting | **[Foundry Models](https://learn.microsoft.com/en-us/azure/foundry/concepts/foundry-models-overview)** | Central model catalog, deployment, evaluation, and model comparison workflows. |
| Stateful agents, tools, conversations, publishing | **[Foundry Agent Service](https://learn.microsoft.com/en-us/azure/foundry/agents/overview)** | Agents, conversations, and responses are first-class runtime components. |
| Grounding, hybrid/vector retrieval, agentic retrieval | **Azure AI Search** | Native support for vector search, hybrid search, semantic reranking, and agentic retrieval patterns. |
| Permission-aware reusable knowledge layer | **Foundry IQ** | Managed knowledge layer with knowledge bases and knowledge sources for agents. |
| External tools and data sources | **MCP tool / Toolbox / project connections** | Standardized integration of remote tools and centralized auth. |
| Structured document extraction | **[Document Intelligence](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/overview?view=doc-intel-4.0.0)** | Deterministic extraction of text, tables, and key-value pairs. |
| Complex multimodal extraction from docs, images, audio, video | **[Content Understanding](https://learn.microsoft.com/en-us/azure/ai-services/content-understanding/overview)** | Generative AI based analyzers for unstructured and multimodal content. |
| Harmful-content detection and moderation | **[Azure AI Content Safety](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/overview) / Foundry guardrails** | Text, image, multimodal moderation plus guardrails and custom categories. |

A recurring exam pattern is the difference between **built-in agent tools** and **custom tools**. Foundry Agent Service provides built-in tools such as **web search**, **code interpreter**, **Azure AI Search**, and file search, while custom/MCP tools let you connect external APIs and tool servers. If the scenario needs **real-time public web grounding**, Microsoft explicitly calls **web search** the recommended built-in method. If it needs internal corpora or citations over enterprise data, prefer **Azure AI Search** or **Foundry IQ**.

## Designing retrieval, indexing, memory, and knowledge integration

For retrieval questions, first decide whether you want **classic hybrid RAG** or **agentic retrieval**. Microsoft recommends **agentic retrieval** for new chatbot or agent scenarios where you want the **highest relevance and accuracy**, especially for **complex or conversational queries** and **structured responses with citations**. Use **classic RAG** when you need maximum simplicity, lower latency, or finer manual control over orchestration. Note that agentic retrieval is **no longer purely preview**: core capabilities are **GA via the 2026-04-01 REST API** (programmatic access), while the Azure and Foundry **portal** experiences remain preview-only.

In Azure AI Search, **vector search** is the core mechanism for semantic similarity and supports **text, multilingual content, and even multiple content types such as text and images**. Hybrid search combines **keyword and vector search in parallel**, and Microsoft explicitly recommends hybrid strategies with **semantic reranking** as one of the strongest approaches for high relevance. That means exam answers that choose **hybrid + semantic reranker** over pure vector are often the more production-ready choice when text fields are available.

For indexing, decide between a **pull model** and a **push model**. Use **indexers** when Azure AI Search can crawl a supported source for you and optionally run a **skillset** for OCR, chunking, image extraction, or embedding generation. Use a **push model** when updates are too frequent or your pipeline already computes vectors outside Search. Microsoft notes that indexers can run on demand or on schedules as often as every five minutes, but for very frequent updates you may need push ingestion instead.

For **chunking and vectorization**, Azure AI Search supports **integrated vectorization**. That means you can use an indexer, skillset, chunking skill, and embedding skill so Search handles the pipeline end to end. For chunking, Microsoft highlights **Text Split** for simpler page/sentence chunking and **Azure Content Understanding skill** for **semantic, layout-aware chunking**. For most RAG scenarios, Microsoft recommends a **single child-chunk index with repeated parent fields**, implemented through **index projections**.

For **multimodal retrieval**, Azure AI Search can ingest text and images into the same pipeline. A common pattern is to extract text and inline images, **verbalize images** into natural language for RAG grounding, then create embeddings and run **hybrid queries**. This is often the exam-best answer when the authoritative content is trapped in **diagrams, screenshots, scanned forms, charts, or PDFs with embedded visuals**.

For **knowledge integration in agents**, distinguish among three layers:

- **Conversation history**: Foundry Agent Service conversations persist message history across turns.
- **Long-term memory**: Foundry memory stores persist user preferences and other reusable facts across sessions, devices, and workflows, with scope-based isolation.
- **Knowledge grounding**: Foundry IQ provides a **permission-aware managed knowledge layer** over enterprise data, while Azure AI Search provides the underlying retrieval engine and indexes.

For **hosted agents**, also distinguish **sessions** from **conversations**. A session is the **sandbox and persisted filesystem**, useful for uploaded files and `$HOME` state across turns; a conversation is the **message history**. Microsoft says hosted sessions can persist for **up to 30 days**, with a **15-minute idle timeout**, but reusing a session does **not** itself replay prior messages to the model. That distinction is an excellent exam trap.

For **tool integration**, use **MCP** when you need external tools or contextual data from remote systems, and use **Toolboxes** when you want a single managed MCP endpoint that bundles multiple tools such as web search, Azure AI Search, file search, code interpreter, and other MCP servers. Toolboxes reduce agent-side configuration and centralize auth and policy handling.

## Setting up Foundry resources, deployments, and pipelines

Architecturally, Microsoft recommends the **Foundry resource** as the top-level governance boundary and **projects** as development boundaries. The resource holds shared governance settings such as networking, security, connections, and deployments; projects isolate teams and assets such as **files, agents, and evaluations**. This is the default architecture for new AI development, especially when multiple teams share deployed models and common guardrails.

For infrastructure provisioning, use **Bicep** or **Terraform** when the requirement is repeatable environment setup, environment promotion, or governance consistency. Microsoft provides first-party quickstarts for creating a **Foundry resource and project** with Bicep or Terraform, and the Bicep guidance explicitly calls out reuse across environments as a benefit. This is the right answer whenever the exam mentions **landing zones, multiple environments, or standardized rollout**.

For **model deployments**, the default recommendation is **standard deployment in Foundry resources**, which Microsoft calls the **preferred and most capable** path. **Managed compute deployment** exists for OSS, partner, and custom models, but it is currently **preview**. The deployment-type decision tree is important:

- **Global Standard**: best starting point for pay-per-token, bursty traffic, and highest default quota.
- **Provisioned**: use when you need **predictable throughput** and **lower latency variance**.
- **Batch**: use for **large async jobs** and lower cost, including Microsoft’s published **50% cost savings** compared with Global Standard for eligible scenarios.
- **Data Zone / Regional**: use when the scenario requires **US/EU data-zone boundaries** or **single-region processing**.
- **Instant models**: useful for prototyping, but still **preview**.
- **Developer**: cost-efficient **fine-tuned model evaluation only**; no SLA, no data-residency guarantee, and a fixed **24-hour deployment lifetime** (auto-deleted after expiry).

For **agent deployments**, Hosted Agent Service emphasizes a separate deployment flow from model deployment. Microsoft supports agent deployment through **Azure Developer CLI (`azd`)** and the **Microsoft Foundry Toolkit for Visual Studio Code**. The hosted-agent quickstart shows the lifecycle clearly: scaffold, provision Azure resources, test locally, deploy the agent container, and invoke the published endpoint. That makes **azd** the strongest exam answer whenever the scenario asks for fast, repeatable local-to-cloud agent deployment.

For **CI/CD**, connect Foundry projects to infrastructure and application automation rather than treating them as portal-only assets. The strongest Microsoft-backed pattern is **IaC + azd + GitHub Actions**: deploy resources with **Bicep**, then use **`azd pipeline config`** to generate a GitHub Actions pipeline for provisioning and deployment. Microsoft also has broader GenAIOps guidance with prompt flow and DevOps tooling, but for this exam section the most durable study target is **Bicep/Terraform for infra** and **azd/GitHub Actions** for continuous delivery.

## Operating, monitoring, and securing AI systems

For quota and scaling, remember the layers of control. Azure OpenAI quota is **subscription-scoped**, while deployment limits include **tokens per minute**, **requests per minute**, and concurrency constraints. Microsoft’s guidance also reminds you that rate limiting can happen over much smaller intervals than one minute (and that you can see a **429** even when token metrics look below quota); the classic illustration is that a **600 RPM** deployment can still produce a **429** if more than **10 requests arrive in one second**. Exam questions often test whether you understand that “per-minute” limits still cause sub-minute throttling.

When the scenario asks for **team-level budget protection** or **cross-project token governance**, use **AI Gateway**. Microsoft positions AI Gateway as an Azure API Management-based layer between clients and Foundry models/tools, and it supports **project-level token limits and quotas** on a shared gateway. This is usually the best answer when one team must not monopolize shared capacity or when you need central usage ceilings without changing application code.

For **model and agent monitoring**, Foundry’s observability story has three pillars: **evaluation, monitoring, and tracing**. Microsoft says evaluators cover quality, safety, and agent-specific metrics such as **groundedness, relevance, tool-call accuracy, and task completion**; monitoring surfaces live operational metrics such as **token consumption, latency, error rates, and quality scores**; tracing captures **LLM calls, tool invocations, and execution flow**. For agents specifically, the Agent Monitoring Dashboard shows **token usage, latency, success rates, and evaluation outcomes** from production traffic.

For **detecting regression and drift**, Foundry’s official observability guidance includes **scheduled evaluation** to detect system drift and **scheduled red teaming** for ongoing risk testing. Foundry Control Plane also references **drift monitoring** as part of ongoing agent testing. If the exam asks how to monitor “drift” in generative systems, the best answer is usually **continuous/scheduled evaluation on production or sampled traffic**, not only classical feature-distribution drift. If the question explicitly moves into classical ML or managed endpoints beyond Foundry-native agents, Azure Machine Learning’s model monitoring for generative apps also supports recurring safety and quality checks.

For **search and grounding quality**, monitor Azure AI Search at three levels:

- **Service metrics** for query performance, indexing volume, and skill execution.
- **Query metrics** such as latency, QPS, and throttling, with diagnostic logs routed to Log Analytics for long retention and analysis.
- **Indexer health** through execution history, errors, warnings, document counts, and alerts.

For **relevance tuning**, Azure AI Search documents two primary good-answer strategies: **hybrid search with semantic reranker** and **agentic retrieval**. You can further tune classic relevance using **scoring profiles**, semantic ranker, freshness/proximity weighting, and hybrid/vector weighting. If a scenario complains that retrieval is “correct but not ideal,” the exam usually wants a retrieval-tuning answer rather than a model-switch answer.

For **security**, be very clear on identity surfaces. Microsoft Foundry separates **control plane** and **data plane**. Control plane activities include creating resources, projects, connections, networking, and deployments; data plane activities include inference, agents, evaluation, monitoring, and tracing. Microsoft recommends **Microsoft Entra ID** for security and traceability; API keys are still supported, but Microsoft says they **lack per-user traceability**.

Use **Foundry roles** to manage access to Foundry resources and projects. Microsoft defines **Foundry User** as the least-privilege role for building in projects, **Foundry Project Manager** for publishing agents, and **Foundry Owner / Foundry Account Owner** for broader management. A subtle but important exam nuance is that in **new Foundry**, keyless inference to model endpoints also uses the **Foundry User** role (assigned on the Foundry account scope), Microsoft explicitly advises *against* assigning **Cognitive Services** roles for Foundry work. The **Cognitive Services User** role was the keyless-inference answer for **classic Azure OpenAI / AI Services** resources, so watch the resource type in the question.

For **keyless credentials**, Microsoft’s keyless auth guidance uses **DefaultAzureCredential** and bearer tokens for the `https://ai.azure.com/.default` audience. Keyless auth is preferred because it eliminates API keys, aligns with RBAC, and supports managed identity patterns. In production, Microsoft also advises moving from generic `DefaultAzureCredential` to more deterministic credential choices when appropriate.

For **private networking**, memorize that Foundry Agent Service’s **Standard Setup with private networking** gives **no public egress**, uses **delegated subnets**, and supports access to private resources when credentials and authorization are in place. For this setup, Microsoft requires **bring-your-own Azure Storage, Azure AI Search, and Azure Cosmos DB** so that agent data stays in your tenant, and it stores Foundry Agent Service data at rest in those resources.

## Implementing responsible AI and governance

Microsoft frames trustworthy AI in Foundry as a lifecycle of **discovering risks, protecting against them, and governing them in production**. For AI-103, translate that into three action areas: **guardrails and moderation**, **evaluation and instrumentation**, and **oversight plus auditability**.

For **guardrails**, Microsoft Foundry lets you apply safety/security controls to both **models** and **agents**. A guardrail is a named collection of controls that specify the **risk**, the **intervention point**, and the **response action**. Foundry supports four intervention points: **user input**, **tool call**, **tool response**, and **output**. That is one of the most important exam facts in this domain because it distinguishes generative Foundry governance from simple pre/post moderation.

The risk catalog includes familiar harmful-content classes such as **hate, sexual, self-harm, and violence**, but it also includes modern generative/agentic risks such as **user prompt attacks, indirect attacks, protected material (text and code), PII, task adherence, groundedness, and spotlighting** (note: groundedness, PII, task adherence, and spotlighting are currently **preview**, and spotlighting/groundedness aren't yet supported for agents). For agents, remember that the assigned **agent guardrail overrides the model’s guardrail**, and some agent-guardrail features are still **preview**.

For **content moderation**, Azure AI Content Safety provides **text, image, and multimodal APIs** to detect harmful content, supports **custom categories**, and uses configurable severity thresholds. Microsoft is also explicit that Content Safety should be **evaluated on your real data**, monitored in production, and not treated as perfect. Exam questions may test whether you choose “use content safety plus human review” instead of “fully automate all decisions.”

For **responsible AI instrumentation**, use **Foundry evaluations** before and after deployment. Microsoft says evaluations can measure **performance, quality, and safety** and can run with built-in or custom evaluators. In generative systems, strong evaluator coverage includes **coherence, fluency, relevance, groundedness**, content-safety defects, and agent-specific behaviors such as **tool-call accuracy** and **task adherence**.

For **explanation tooling**, be careful: this is one area where the exam domain is broader than pure LLM operations. Microsoft’s **Responsible AI dashboard** in Azure Machine Learning provides **model interpretability, error analysis, fairness assessment, counterfactuals, and causal analysis**, and the companion scorecard supports stakeholder review. This is most relevant when your overall Azure AI solution includes **classical ML models** alongside Foundry agents or generative apps. It is less about explaining LLM chain-of-thought and more about **feature-level explanation and model debugging** for supported tabular ML models.

For **auditing and provenance**, use **tracing**, **citations**, and **approval workflows** together. Foundry traces are stored in **Azure Monitor Application Insights** using **OpenTelemetry**, and Microsoft says traces capture key details such as **inputs, outputs, tool usage, retries, latency, and cost**. For grounded answers from Azure AI Search, Microsoft recommends storing a **retrievable text field**, a **source URL field**, and optionally a title field so the agent can return useful citations. This is precisely the kind of provenance design choice the exam can test.

For **approval workflows and oversight modes**, Microsoft supports **human-in-the-loop** patterns for tool calls. Its tool-approval guidance explains that an agent run can stop and return a request for approval instead of completing with a final answer. MCP integration guidance also explicitly mentions the ability to **review and approve MCP tool calls**. That makes human approval the right answer whenever the scenario involves high-risk actions such as writing to systems, triggering workflows, or accessing sensitive downstream tools.

For **governing agent behavior**, combine three controls:

- **Oversight** through approval workflows and human-in-the-loop review.
- **Constraints** through guardrails, AI Gateway token policies, and Azure Policy for deployment governance.
- **Tool-access controls** through `tool_choice`, `allowed_tools`, project connections, least-privilege credentials, and secure tool instructions.

## What to memorize for exam day

If you only memorize a small set of high-yield ideas from this domain, make it these:

- **Foundry resource** is the top-level governance boundary; **project** is the team/development boundary.
- Use **reasoning models** for hard planning, coding, and analysis; use **faster non-reasoning models** for real-time chat and cost-efficient throughput.
- Use **Azure AI Search** for vector/hybrid retrieval; use **Foundry IQ** when you want reusable, permission-aware knowledge bases for agents.
- Use **agentic retrieval** for new, complex conversational grounding; use **classic RAG/hybrid** for simpler or GA-only scenarios.
- **Standard deployment** is preferred; **Provisioned** is for predictable high throughput/latency; **Batch** is for large async jobs; **Data Zone/Regional** is for residency/compliance.
- **AI Gateway** is the answer for project-level token limits, quota enforcement, and centralized routing/policy.
- **Evaluation + monitoring + tracing** is Foundry’s observability triangle.
- **Entra ID + RBAC + managed identity** beats API keys for production.
- **Private networking** for agents means no public egress and BYO Storage/Search/Cosmos DB.
- **Guardrails** can intervene at input, tool call, tool response, and output.
- **Session ≠ conversation** in hosted agents. Sessions keep sandbox/files; conversations keep message history.

## Open questions and preview caveats

Several capabilities relevant to this domain are still documented as **preview** or have preview subfeatures, including **agentic retrieval in the portal** (its REST API is now GA), **model leaderboards**, **managed compute deployments**, some **agent guardrails** (e.g., tool-call/tool-response intervention points, spotlighting, groundedness for agents), the **Agent Monitoring Dashboard**, and **hosted-agent session management**. Microsoft’s AI-103 study guide explicitly says preview features can appear on the exam if they are commonly used, so you should study the concepts and tradeoffs, but answer production-design questions carefully when the prompt asks for a **GA-only** solution.

The exam outline uses the phrase **“explanation tooling”** in this domain, but Microsoft’s clearest first-party documentation for that phrase today is in **Azure Machine Learning’s Responsible AI dashboard**, not directly in Foundry-native LLM observability. For study purposes, treat this as a signal that AI-103 expects you to understand how a broader Azure AI solution can mix **Foundry for generative/agentic apps** with **Azure Machine Learning for classical model explanation and fairness analysis**.

## Further reading

Microsoft Learn pages for this domain (verified June 2026):

- [What is Microsoft Foundry?](https://learn.microsoft.com/en-us/azure/foundry/what-is-foundry)
- [Microsoft Foundry architecture](https://learn.microsoft.com/en-us/azure/foundry/concepts/architecture)
- [Microsoft Foundry Models overview](https://learn.microsoft.com/en-us/azure/foundry/concepts/foundry-models-overview)
- [What is Microsoft Foundry Agent Service?](https://learn.microsoft.com/en-us/azure/foundry/agents/overview)
- [Get started with Microsoft Foundry SDKs and endpoints](https://learn.microsoft.com/en-us/azure/foundry/how-to/develop/sdk-overview)
- [Azure AI Search documentation](https://learn.microsoft.com/en-us/azure/search/)
- [What is Azure AI Content Safety?](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/overview)
- [Exam AI-103 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-103)
