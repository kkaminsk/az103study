# AI-103 Study Guides: Developing AI Apps and Agents on Azure

A set of dense, fact-checked study guides for Microsoft **[Exam AI-103: Developing AI Apps and Agents on Azure](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-103)**, which leads to the **[Microsoft Certified: Azure AI Apps and Agents Developer Associate](https://learn.microsoft.com/en-us/credentials/certifications/azure-ai-apps-and-agents-developer-associate/)** certification. AI-103 is the successor to **AI-102** (*Designing and Implementing a Microsoft Azure AI Solution*), which **retires June 30, 2026**.

Each guide maps to one official exam domain, built around the platform now called **Microsoft Foundry** (formerly *Azure AI Foundry*, earlier *Azure AI Studio*). The skills outlines referenced here are the blueprint dated **April 16, 2026**.

> **Who this is for:** Azure AI engineers and developers who build, manage, and deploy agents and AI solutions with Python and Microsoft Foundry, and who want a service-selection-focused review rather than a click-through tutorial.

## The guides

The guides are numbered in **suggested study order** (weight-first, building one small end-to-end project per guide), not in official exam-domain order. Start with the largest-weight generative and agentic material, layer in retrieval and grounding, then add the modality-specific skills.

| # | Guide | Exam domain | Weight |
|---|---|---|---|
| 1 | [Generative AI and agentic solutions](1-ai-103-study-guide-for-generative-ai-and-agentic-solutions.md) | Implement generative AI and agentic solutions | **30–35%** |
| 2 | [Plan and manage an Azure AI solution](2-ai-103-study-guide-for-planning-and-managing-an-azure-ai-solution.md) | Plan and manage an Azure AI solution | **25–30%** |
| 3 | [Information extraction solutions](3-ai-103-study-guide-for-implementing-information-extraction-solutions.md) | Implement information extraction solutions | **10–15%** |
| 4 | [Text analysis and speech](4-ai-103-study-guide-for-text-analysis-and-speech.md) | Implement text analysis solutions (speech is a subsection here) | **10–15%** |
| 5 | [Computer vision solutions](5-ai-103-study-guide-for-implementing-computer-vision-solutions.md) | Implement computer vision solutions | **10–15%** |

> **Note on weighting:** The generative AI and agentic domain (Guide 1) is the largest single slice of the exam at 30–35%, on AI-102 this material was split into two smaller domains, so it's a strong signal of where to invest study time. Speech is **not** its own domain; it's a subsection inside the text-analysis domain (Guide 4), so don't double-count it.

## What each guide covers

- **[1 · Generative AI and agentic solutions](1-ai-103-study-guide-for-generative-ai-and-agentic-solutions.md)**: Building generative apps on the Foundry project endpoint; the **Responses API (Agents v2)** runtime model (agents, conversations, responses); agent tools, function calling, and multi-agent orchestration; tuning, evaluation, tracing, and observability.
- **[2 · Plan and manage](2-ai-103-study-guide-for-planning-and-managing-an-azure-ai-solution.md)**: Choosing models, Foundry services, and tools; designing retrieval, indexing, memory, and knowledge integration; setting up Foundry resources, deployments (Standard / Provisioned / Batch / Data Zone), and CI/CD pipelines; operating, monitoring, securing (RBAC, Entra ID, private networking); and responsible AI governance with guardrails.
- **[3 · Information extraction](3-ai-103-study-guide-for-implementing-information-extraction-solutions.md)**: Retrieval and grounding pipelines (pull vs. push ingestion, full-text/vector/hybrid/semantic/agentic retrieval, enrichment skills); and document extraction with **Azure AI Document Intelligence** (deterministic fields) vs. **Content Understanding** (generative, schema-driven, RAG-ready output).
- **[4 · Text analysis and speech](4-ai-103-study-guide-for-text-analysis-and-speech.md)**: Entity/topic/summary extraction and structured JSON outputs; sentiment, PII, and content safety; Azure Translator vs. LLM translation flows; speech-to-text/text-to-speech, custom speech, the Realtime API, and multimodal audio reasoning.
- **[5 · Computer vision](5-ai-103-study-guide-for-implementing-computer-vision-solutions.md)**: Image generation (`gpt-image-*`, not the retired DALL-E 3) and video generation (Sora 2); vision-enabled chat models; captions and accessibility alt text; Content Understanding analyzers; and multimodal safety, provenance (Content Credentials / C2PA), and moderation.

## High-leverage facts that are easy to get wrong

These claims drift fast as Microsoft renames and re-scopes services through 2026:

- **The platform is Microsoft Foundry.** Don't reintroduce the old *Azure AI Foundry* / *Azure AI Studio* names except when contrasting legacy vs. current.
- **The exam is AI-103**, not AI-102.
- **Keyless model inference in new Foundry uses the `Foundry User` role**, not `Cognitive Services User` (the latter was the classic Azure OpenAI answer, watch the resource type in the question).
- **The runtime is the Responses API (Agents v2)**, conversations / items / responses / agent versions, not the legacy **Assistants API** (threads / messages / runs / assistants), reported to sunset **August 26, 2026**.

## Accuracy and verification

These guides make specific, drift-prone claims about Foundry, Azure AI Search, deployment types, RBAC roles, quotas, and guardrails. Every factual claim should be verifiable against current Microsoft Learn documentation. The guides are fact-checked per file; some open with a dated **"Accuracy note (verified \<month year\>)"** block recording when the check happened.

Each guide links its key services to the relevant Microsoft Learn pages, both inline on first mention and in a **"Further reading"** section at the end. The links target the current docs (new Microsoft Foundry under `azure/foundry`, the individual "Foundry Tools" services under `azure/ai-services`), and were verified live in June 2026.

## Further reading

- [Exam AI-103 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-103)
- [Microsoft Certified: Azure AI Apps and Agents Developer Associate](https://learn.microsoft.com/en-us/credentials/certifications/azure-ai-apps-and-agents-developer-associate/)
- [Course AI-103T00-A: Develop AI apps and agents on Azure](https://learn.microsoft.com/en-us/training/courses/ai-103t00)
- [What is Microsoft Foundry?](https://learn.microsoft.com/en-us/azure/foundry/what-is-foundry)
- [Microsoft Foundry documentation](https://learn.microsoft.com/en-us/azure/foundry/)
