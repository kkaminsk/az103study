# AI-103 Study Guide for Generative AI and Agentic Solutions

> **Slide deck:** A companion presentation for this guide is available at [`Slides/1-AI-103_Agent_Engineering.pptx`](Slides/1-AI-103_Agent_Engineering.pptx).

> **Podcast episode:** A companion audio deep-dive for this guide is available at [The Microsoft AI-103 Agent Blueprint](https://stbighatpodcasts.blob.core.windows.net/podcasts/1-The_Microsoft_AI_103_agent_blueprint.m4a).

> **Accuracy note (verified June 2026).** This guide was fact-checked against current Microsoft Learn documentation. Key context to anchor your study:
> - **Exam:** AI-103, *Developing AI Apps and Agents on Azure*, leads to the **Microsoft Certified: Azure AI Apps and Agents Developer Associate** certification. It is the successor to **AI-102** (*Designing and Implementing a Microsoft Azure AI Solution* / Azure AI Engineer Associate), which **retires June 30, 2026**; the matching legacy course AI-102T00 retires April 30, 2026.
> - **Platform rebrand:** *Azure AI Foundry* (and earlier *Azure AI Studio*) is now **Microsoft Foundry** (announced at Ignite, Nov 2025). Older materials saying "Azure AI Foundry" or "Azure AI Studio" are using superseded names.
> - **Runtime/API shift:** the agent runtime moved from the **Assistants API** paradigm (threads, messages, runs, assistants) to the **Responses API (Agents v2)** model (conversations, items, responses, agent versions). The legacy Assistants API is reported to sunset **Aug 26, 2026**.
> - **SDK:** the current Python package is **`azure-ai-projects` 2.2.0+** (the 2.x line is **not** compatible with 1.x), authenticating with `DefaultAzureCredential` (Microsoft Entra ID is the only supported auth).

## Exam context and how to use this guide

Microsoft's official AI-103 study guide identifies **"Implement generative AI and agentic solutions"** as a **30–35%** exam domain, the largest of the five measured domains. The full blueprint is: Plan and manage an Azure AI solution (25–30%), **Implement generative AI and agentic solutions (30–35%)**, Implement computer vision solutions (10–15%), Implement text analysis solutions (10–15%), and Implement information extraction solutions (10–15%). The exam targets developers who build, manage, and deploy AI-infused apps and agents by using **[Microsoft Foundry](https://learn.microsoft.com/en-us/azure/foundry/what-is-foundry)**, and candidates should already be comfortable with **Python**, generative AI fundamentals, and Azure services. The skills in this section were published for the exam blueprint dated **April 16, 2026**.

> **Worth knowing for context:** on the older AI-102 exam this material was split into **two** smaller domains, *Implement generative AI solutions (15–20%)* and *Implement an agentic solution (5–10%)*. AI-103 consolidates them into the single, heavily weighted 30–35% domain, which is a strong signal of where to invest study time.

Your requested scope maps exactly to three clusters in the official objectives: **build generative applications by using Foundry**, **build agents by using Foundry**, and **optimize and operationalize generative AI systems**. A practical way to study is to treat each bullet in the blueprint as a hands-on capability: if you can explain when to use it, configure it in Foundry, and troubleshoot the common failure modes, you are preparing at the right depth. That emphasis on verbs such as *deploy, implement, integrate, configure, evaluate,* and *orchestrate* is visible throughout the official study guide.

Microsoft also has a matching official course, **[AI-103T00-A: Develop AI apps and agents on Azure](https://learn.microsoft.com/en-us/training/courses/ai-103t00)**, plus dedicated learning paths for **developing generative AI apps** and **developing AI agents on Azure**. Those resources are not a substitute for the study guide, but they are the closest official practice path for this exam section.

## Build generative applications by using Foundry

The first thing to know is the **Foundry application surface**. Microsoft describes the **[Foundry SDK](https://learn.microsoft.com/en-us/azure/foundry/how-to/develop/sdk-overview)** as a thin client over Foundry project APIs through a **single project endpoint**. The data-plane endpoint takes the form `https://<your-ai-services-account>.services.ai.azure.com/api/projects/<project-name>`, note the **`.services.ai.azure.com`** host. (Be careful: bare `<resource>.ai.azure.com` is the **Foundry portal** URL and also appears in some SDK code comments, but the load-bearing data-plane/REST host includes `services`.) In practice, that means you should be comfortable explaining how an app authenticates, stores the project endpoint, and then obtains an OpenAI-compatible client or agent client from that project context. For Python, Microsoft's current quickstart uses **`azure-ai-projects>=2.2.0`** with Azure authentication such as `DefaultAzureCredential`; the 2.x line is built on the GA "v1" Foundry data-plane REST APIs and is not compatible with 1.x code.

For **models**, [Microsoft Foundry Models](https://learn.microsoft.com/en-us/azure/foundry/concepts/foundry-models-overview) provides a catalog of **1,900+ models**, from Microsoft, OpenAI, Anthropic, Mistral, xAI, Meta, DeepSeek, Hugging Face, and more, including **foundation models, reasoning models, small language models, multimodal models, domain-specific models, and industry models**. (That figure is vendor-reported, so treat "over 1,900" as an approximate scale, not a fixed number.) The exam objective you listed specifically calls out **LLMs, small models, code models, and multimodal models**, so your study focus should be less about memorizing every vendor model name and more about being able to choose a model class for the job: reasoning-heavy work, low-latency work, code generation, multimodal understanding, or cost-sensitive inference.

You should also know how **deployment choices affect architecture**. Microsoft organizes deployments into two main categories, **standard** (pay-per-token) and **provisioned** (reserved capacity), and within each you choose **global**, **data zone**, or **regional** processing. Documented SKUs include **Global Standard**, **Global Provisioned (managed)**, **Data Zone Standard**, **Data Zone Provisioned (managed)**, **Standard (regional)**, and **Regional Provisioned**, alongside **Batch**, **Instant**, and **Developer** tiers. A very exam-friendly distinction is this: **Global** options maximize reach and quota, **Provisioned** options reduce latency variance for steady high-volume workloads, **Data Zone** options support US or EU processing constraints, **Regional/Standard** options keep processing in one region, and **Batch** targets large asynchronous jobs at lower cost.

For **RAG**, Microsoft says Foundry can connect a project to **[Azure AI Search](https://learn.microsoft.com/en-us/azure/search/)** for retrieval, and that this connection can appear either as a **project connection** or an **index asset ID**, depending on the API surface. Azure AI Search is the recommended index store for Foundry RAG scenarios, and Microsoft's RAG guidance emphasizes both **vector** and **textual** retrieval. The related Azure AI Search guidance says Azure AI Search supports both **classic RAG** and **agentic retrieval**, with support for chunking, vectorization, hybrid search, semantic ranking, and retrieval over multimodal content.

A high-value distinction for the exam is **file search versus Azure AI Search versus agentic retrieval**. In [Foundry Agent Service](https://learn.microsoft.com/en-us/azure/foundry/agents/overview), the **file search tool** is best when you upload documents directly and want the service to handle parsing, chunking, embeddings, vector storage, and hybrid retrieval for you. Microsoft says file search automatically rewrites queries, breaks complex queries into parallel searches, uses hybrid search, reranks results, and can attach vector stores to agents or conversations. By contrast, **Azure AI Search** is the better fit when you already manage search infrastructure or need broader indexing control, while **agentic retrieval** is the knowledge layer for harder agent-style questions that draw on chat context, proprietary content, and external sources.

For **workflows, tool-augmented flows, and multistep reasoning pipelines**, you should understand the escalation path from simple to complex. Microsoft's architecture guidance says you should start at the **lowest level of complexity** that reliably meets requirements: first a **direct model call**, then a **single agent with tools**, and only then **multiagent orchestration** if necessary. Foundry workflows are appropriate when you need repeatable orchestration, branching logic, variable handling, or human-in-the-loop steps such as approvals or clarifying questions.

For **evaluation**, Microsoft Foundry includes built-in evaluators for **quality, safety, and reliability**, and Foundry portal evaluations can be run before deployment or against production-style traffic. In RAG-specific scenarios, Microsoft explicitly calls out **groundedness**, **relevance**, and **response completeness**, with groundedness focused on precision against the supplied context and response completeness focused on recall against ground truth. The groundedness detection filter is also important: Microsoft describes it as a way to reduce fabricated or non-factual outputs by validating responses against source material.

Finally, for **integrating generative workflows into applications**, know the current integration model. Microsoft says Foundry can be used as a complete developer platform or selectively embedded into custom applications and partner solutions. The exam objective about "configure an application to connect to a Foundry project" is really about project endpoint access, authentication, project connections, and selecting the right SDK or REST path for the scenario. Connections matter because Microsoft documents them as the mechanism that lets Foundry projects authenticate to Microsoft and non-Microsoft resources, and they are required for scenarios such as **Standard Agents** and **agent knowledge tools**.

## Build agents by using Foundry

For the agent section, start with Microsoft's **runtime object model**. Under the current **[Responses API (Agents v2)](https://learn.microsoft.com/en-us/azure/foundry/openai/how-to/responses)** model, Foundry Agent Service uses three core runtime components: **agents, conversations, and responses**. An **agent** combines a model, instructions, and tools. A **conversation** persists the running history across turns. A **response** is the output generated when the agent processes input (with `previous_response_id` used to chain turns). If you can clearly define those three terms, and describe how they work together in a stateful, multi-turn app, you are covering several bullets in the blueprint at once.

> **Watch for the legacy-vs-current trap.** Older Foundry/Azure OpenAI material describes the runtime as **threads, messages, runs, and assistants** (the **Assistants API**, "Agents v0.5/v1"). The current model is **conversations, items, responses, and agent versions** (the **Responses API**, "Agents v2"). The Assistants API is reported to sunset **Aug 26, 2026**, so an exam answer built on "threads/runs" terminology is the outdated one.

When the blueprint says **define agent roles, goals, conversation-tracking approach, and tool schemas**, think of it this way: the **role and goal** live primarily in the agent's instructions and system behavior, the **conversation-tracking approach** lives in the conversation object or stateful response flow, and the **tool schema** is the contract that tells the model what it can call and how. Microsoft's quickstart reinforces this by creating an agent from a model plus instructions, while Foundry Agent Service documentation shows that a conversation preserves context across turns.

For **retrieval, function calling, and memory**, Microsoft provides separate building blocks. Retrieval can be done with **file search**, Azure AI Search-related tools, or knowledge sources; function calling lets you extend an agent with named functions, parameters, and descriptions; and memory is primarily handled through the conversation state. Microsoft's function-calling documentation is especially exam-worthy because it states the exact pattern: the model requests a function call, **your application executes the function**, and then your app returns the output so the model can continue the conversation using real-time data from your systems.

You should know the **major tool types** and when they fit. Microsoft's agent **tool catalog** shows a broad and growing set: **Azure AI Search**, **Azure Functions**, **Bing Grounding**, **Bing Custom Search** (Preview), **Code Interpreter**, **File Search**, **Function** (custom) tools, **MCP** (Model Context Protocol), **OpenAPI**, **Web Search**, **Microsoft SharePoint** (Preview), **Microsoft Fabric** and **Fabric IQ** (Preview), plus newer preview tools such as **Agent-to-Agent (A2A)**, **Browser Automation**, **Computer Use**, **Image Generation**, **Memory Search**, **Toolbox Search**, and **Work IQ**. **[Content Understanding](https://learn.microsoft.com/en-us/azure/ai-services/content-understanding/overview)** is also important for this exam domain because the blueprint explicitly mentions it, and Microsoft describes it as a Foundry Tool that turns documents, images, video, and audio into structured output for downstream workflows.

Two operational details deserve special attention because they are exactly the kind of thing Microsoft likes to test. First, **tool support depends on both region and model**, so a tool can fail even if the feature exists broadly. Second, Microsoft's tool best-practices guidance says `tool_choice` gives deterministic control over tool calling, with `auto`, `required`, and `none` as the main modes. That makes `tool_choice` one of the cleanest answers when a scenario asks how to force reliable tool usage.

For **multi-agent solutions**, Microsoft's architecture guidance says not every workload needs multiple agents. The default should often be one agent with tools. Move to **multiagent orchestration** when a single agent becomes overloaded by domain breadth, security boundaries, or prompt/tool complexity. Microsoft's orchestration guide lays out patterns such as **sequential orchestration**, **group chat**, **handoff**, and maker-checker or **reflection loops**, and the Foundry workflow documentation adds practical capabilities such as branching logic and human-in-the-loop steps.

For **autonomous and semiautonomous workflows with safeguards**, the key idea is controlled autonomy. Microsoft's workflow guidance explicitly supports **approvals** and clarifying-question steps, and the broader orchestration guidance says human checkpoints should be placed where risk is high, especially before sensitive tool invocations. That same guidance recommends persisting state at mandatory gates so work can resume cleanly, and it notes that approvals can be scoped to **specific tool invocations** rather than every agent output.

For **monitoring and error analysis**, memorize the relationship between **traces**, **Application Insights**, **dashboard metrics**, and **evaluations**. Microsoft says tracing in Foundry sends telemetry to **Azure Monitor Application Insights** through OpenTelemetry conventions, while the Agent Monitoring Dashboard surfaces **token usage, latency, success rates, and evaluation outcomes**. Agent evaluation guidance adds that you can run evaluator sets specifically for **quality, safety, and agent behavior**, which is the most direct match to the exam bullet on evaluating agent behavior and performing error analysis.

## Optimize and operationalize generative AI systems

For **tuning generation behavior**, the exam expects you to know both **prompt design** and **model parameters**. Microsoft's prompt engineering guidance says prompt construction can include instructions, examples, cues, and formatting constraints, while the parameter guidance highlights **temperature** as a major lever: lower values produce more focused and concrete output, while higher values produce more random and divergent output. Foundry playgrounds also expose practical knobs such as **temperature**, **top_p**, **max_tokens**, system prompts, and tool configuration, which makes playground experimentation a strong prep method.

For the **reflection, chain-of-thought evaluation, and self-critique** objective, the most important exam insight is that Microsoft distinguishes between **pattern design** and **reason extraction**. The architecture guide explicitly describes maker-checker loops, also called evaluator-optimizer, generator-verifier, critic loops, or reflection loops, and recommends clear acceptance criteria plus iteration caps to avoid infinite refinement. At the same time, Microsoft's prompt engineering guidance warns that **chain-of-thought prompting is only applicable to non-reasoning models**, and that attempts to extract hidden reasoning from reasoning models are unsupported and can violate policy. That is a very testable nuance.

For **observability**, Microsoft positions evaluation and observability as a joint discipline. The official observability overview says Foundry supports systematic evaluation and observability for safe, high-quality generative AI. In practice, that means you should be able to describe the full stack: **traces** for step-by-step execution, **Application Insights** as telemetry storage, the **Agent Monitoring Dashboard** for token, latency, and success metrics, and **continuous or repeated evaluations** for quality and safety trending over time.

For **token analytics and latency breakdowns**, connect the platform view to the model-inference view. Microsoft's monitoring dashboard explicitly surfaces token usage and latency at the agent level, while Azure OpenAI in Foundry documentation also calls out per-request token usage data in API responses as a useful input to throughput and workload analysis. The operational skill being tested is not just "find the metric," but also "use the metric to diagnose cost, performance, or scaling issues."

For **orchestrating multiple models, flows, or hybrid LLM-and-rules systems**, think in terms of design discipline. Microsoft's orchestration guidance says you should use the **lowest complexity level** that solves the problem and only add multiagent coordination when it buys specialization, reliability, or security separation. It also warns that more agents increase latency, coordination overhead, and failure modes. A hybrid design is usually the right answer when some substeps are deterministic enough for business rules or branching logic, and other substeps genuinely require model reasoning. That is exactly why Foundry workflows support variables, branches, and human gates alongside agent steps.

## Exam focus areas and high-value distinctions to memorize

If you only memorize a small set of distinctions for this exam domain, make them these. **Foundry project endpoint** (`…services.ai.azure.com/api/projects/…`) is the app entry point for SDK and project API access; **connections** authenticate Foundry to external resources; the **Responses API** gives stateful multi-turn responses; **Agent Service** provides managed agent runtime; and **workflows** add orchestration, branches, and human checkpoints. These are related, but they are not interchangeable.

Also memorize the retrieval split. **File search** is easiest when you upload files directly and want managed parsing, embeddings, and hybrid retrieval. **Azure AI Search** is the broader retrieval platform for managed search infrastructure and index design. **Agentic retrieval** is the higher-order retrieval layer for complex agent or app experiences that benefit from query planning, chat context, and multiple knowledge sources. **Content Understanding** complements retrieval by converting raw multimodal content into structured, grounded representations.

The main evaluation split is similar. **Built-in evaluators** help assess quality, safety, and reliability broadly. **RAG evaluators** focus on groundedness, relevance, and completeness. **Groundedness detection** is a targeted way to detect fabricated output relative to source material. If a scenario asks how to prove a RAG answer is well-grounded, you should immediately think of **groundedness**, **relevance**, and possibly **response completeness** rather than a generic safety check.

The last distinction to lock in is **single-agent versus multi-agent**. Microsoft's guidance is clear that one agent with tools is often the right default, and that multiagent designs are justified only when their extra complexity, latency, coordination cost, or security partitioning are genuinely needed. In other words, "use more agents" is not the sophisticated answer; "use the lowest complexity that reliably solves the task" is.

## Practice questions and answer guide

Use these as self-checks. If you cannot explain **why** an answer is right, revisit the matching objective.

**Question:** Your team wants a quick internal chatbot over a small set of uploaded PDFs, without standing up separate search infrastructure. What should you prefer first?  
**Answer:** Start with **file search** in Foundry Agent Service. Microsoft documents it as the built-in tool for uploaded files, with managed parsing, chunking, embeddings, vector storage, and hybrid retrieval.

**Question:** A regulated workload requires prompts and responses to remain processed inside the EU data zone. Which deployment family is the best fit?  
**Answer:** **Data Zone** deployment types, because Microsoft says Data Zone processing stays within the specified US or EU data zone.

**Question:** You need deterministic tool usage because the model sometimes answers from memory instead of calling your API. What control should you know?  
**Answer:** Use **`tool_choice`**, especially `required` when the model must call one or more tools. Microsoft's tool-best-practices guidance calls this the most deterministic control over tool calling.

**Question:** An agent needs to call your business logic and receive live data from enterprise systems. What is the core pattern?  
**Answer:** **Function calling**. You define a function name, parameters, and description; the model requests the call; your app executes it; and you return the result so the agent can continue.

**Question:** An answer uses retrieved passages but still invents details not present in the context. Which evaluator concept is most directly relevant?  
**Answer:** **Groundedness**, because Microsoft defines it as whether the response stays aligned to the supplied context without fabrication.

**Question:** When should you move from a single agent with tools to a multi-agent design?  
**Answer:** Only when one agent can no longer reliably handle the task due to prompt complexity, tool overload, domain specialization, or security-boundary requirements. Microsoft explicitly recommends using the lowest complexity level that works.

**Question:** What are the three core runtime components in Foundry Agent Service (current Responses API model)?  
**Answer:** **Agents, conversations, and responses**. Agents define behavior and tools, conversations persist history, and responses are the generated outputs. (If you were taught **threads, messages, runs, assistants**, that is the older Assistants API model, now superseded.)

**Question:** What is the cleanest way to connect a custom application to Foundry?  
**Answer:** Use the **Foundry project endpoint** (`https://<account>.services.ai.azure.com/api/projects/<project>`) with the **Foundry SDK / Azure AI Projects client** (`azure-ai-projects` 2.2.0+), authenticate with `DefaultAzureCredential`, and then obtain the appropriate model or agent client from the project context.

**Question:** What should you use to monitor token usage, latency, success rates, and evaluation outcomes for deployed agents?  
**Answer:** **Tracing plus Application Insights plus the Agent Monitoring Dashboard**. Microsoft documents this as the operational monitoring path for Foundry agents.

**Question:** What caution applies to chain-of-thought prompting in current Microsoft guidance?  
**Answer:** Microsoft says chain-of-thought prompting is only applicable to **non-reasoning models**, and attempts to extract hidden reasoning from reasoning models are unsupported and can violate policy.

## A realistic study plan for this exam section

A strong prep sequence is to study in the same order Microsoft's objective list uses. First, build one **simple generative app**: connect to a Foundry project, deploy or select a model, send a response request, and vary temperature or token settings. Then add **RAG** with either file search or Azure AI Search. After that, turn the app into a **stateful agent** with instructions, conversations, and at least one external tool such as function calling, code interpreter, MCP, or Azure Functions. Finally, add **evaluation, tracing, and monitoring** so you can explain not just how to build the app, but how to prove it is useful, relevant, safe, and observable.

If you want your final review to stay strictly aligned to official materials, prioritize these Microsoft resources: the **AI-103 study guide**, the **AI-103T00-A course**, the Foundry documentation for **models**, **RAG**, **agents**, **SDKs**, **connections**, **evaluations**, and **observability**, plus the Azure Architecture guidance on **agent orchestration patterns**. That combination covers nearly every bullet in the section you provided.

## Further reading

Microsoft Learn pages for this domain (verified June 2026):

- [What is Microsoft Foundry?](https://learn.microsoft.com/en-us/azure/foundry/what-is-foundry)
- [Microsoft Foundry Models overview](https://learn.microsoft.com/en-us/azure/foundry/concepts/foundry-models-overview)
- [Get started with Microsoft Foundry SDKs and endpoints](https://learn.microsoft.com/en-us/azure/foundry/how-to/develop/sdk-overview)
- [Microsoft Foundry architecture](https://learn.microsoft.com/en-us/azure/foundry/concepts/architecture)
- [What is Microsoft Foundry Agent Service?](https://learn.microsoft.com/en-us/azure/foundry/agents/overview)
- [Use the Azure OpenAI Responses API](https://learn.microsoft.com/en-us/azure/foundry/openai/how-to/responses)
- [Generate text responses with Microsoft Foundry Models](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/how-to/generate-responses)
- [Azure AI Search documentation](https://learn.microsoft.com/en-us/azure/search/)
- [What is Azure Content Understanding in Foundry Tools?](https://learn.microsoft.com/en-us/azure/ai-services/content-understanding/overview)
- [Exam AI-103 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-103)
