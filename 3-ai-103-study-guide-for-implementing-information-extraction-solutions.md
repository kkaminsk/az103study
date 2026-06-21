# AI-103 Study Guide for Implementing Information Extraction Solutions

## Scope and exam framing

This guide covers the **“Implement information extraction solutions”** domain of AI-103, which Microsoft weights at **10–15%** of the exam in the skills outline dated **April 16, 2026**. This is a distinct, standalone domain, it is *not* part of the text-analysis or computer-vision domains, even though it shares services with both. Microsoft’s official sub-objectives for this domain fall into two clusters:

- **Build retrieval and grounding pipelines**
  - Ingest and index content, such as documents, images, audio, and video.
  - Configure semantic search, hybrid search, and vector search for grounding.
  - Implement enrichment by using custom or built-in skills for text, images, and layout.
  - Configure RAG ingestion flow, including documents and using optical character recognition (OCR).
  - Connect retrieval pipelines directly to workflows and agent tools.
- **Extract content from documents**
  - Extract information by using multimodal pipelines that combine OCR, layout analysis, and field extraction.
  - Produce clean, grounded representations to use with agents and RAG by using Content Understanding.
  - Implement analyzers for generating structured or markdown outputs for downstream reasoning by using Content Understanding.

The right way to study this domain is to think like a **search and grounding engineer**. The exam wants you to choose the correct ingestion model, the correct query type, the correct enrichment skill, and the correct document-extraction tool for a scenario, and to justify *why*. Microsoft also reminds candidates that most questions target generally available features, though commonly used preview features can appear, which matters here because the **Azure AI Search Content Understanding skill** and parts of **agentic retrieval** are still maturing. (Note: the underlying Content Understanding service now has a **GA API version, `2025-11-01`**, the older preview API versions retire **July 15, 2026**, so it is the *Search skill* and *portal experiences*, rather than the core service, that still carry preview labels.)

## Service map you need to know

For this domain you must be fluent in four building blocks and know exactly where each one stops:

| Service / capability | Use it when the scenario is… | Why it fits |
|---|---|---|
| **[Azure AI Search](https://learn.microsoft.com/en-us/azure/search/)** | You need an index, retrieval at scale, vector/hybrid/semantic queries, or grounding for RAG and agents | The retrieval engine: indexers, skillsets, vector and hybrid search, semantic ranking, agentic retrieval, knowledge stores |
| **[Azure AI Document Intelligence](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/overview?view=doc-intel-4.0.0)** (Form Recognizer) | You need **deterministic** field extraction, layout, tables, and key-value pairs from forms and structured documents | Prebuilt + custom models with confidence scores and bounding regions; strongest for invoices, receipts, IDs, contracts |
| **[Azure Content Understanding](https://learn.microsoft.com/en-us/azure/ai-services/content-understanding/overview)** (Foundry Tool) | You need **generative, schema-driven** structured output from mixed/unstructured content (documents, images, audio, video) | Analyzers that turn raw multimodal content into markdown/JSON fields, RAG-ready and grounded |
| **[Azure Vision OCR / image analysis](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/overview-image-analysis)** | You need text or visual features pulled from images to feed an index or policy engine | OCR (Read) skill and image analysis as enrichment steps inside a skillset |

The single most testable distinction in this domain is **Document Intelligence vs. Content Understanding**. Choose **Document Intelligence** when the requirement is precise, repeatable extraction of *known fields* from *structured forms* with confidence scores (an invoice total, a policy number, a signature region). Choose **Content Understanding** when the requirement is to turn *messy, mixed, or multimodal* content into a *user-defined schema or LLM-ready markdown* for downstream RAG or agents. A second high-value distinction is **classic RAG vs. agentic retrieval**, covered below.

## Build retrieval and grounding pipelines

### Ingesting and indexing content (documents, images, audio, video)

Azure AI Search supports two ingestion models, and choosing correctly is a recurring exam pattern.

- **Pull model (indexers).** Use an **indexer** when Search can crawl a supported source for you. Built-in indexers exist for **Azure Blob Storage, ADLS Gen2, Azure SQL, Cosmos DB, Azure Table Storage, OneLake, and SharePoint Online**, among others. Indexers can run on demand or on a schedule, support **incremental/change-tracking** refresh, and can attach a **skillset** for enrichment. Microsoft notes indexers can run as often as every five minutes, so for content that changes faster than that, you need the push model.
- **Push model (REST/SDK).** Use the **push model** when updates are too frequent for a schedule, when your data has no supported indexer, or when you already compute embeddings in your own pipeline. You upload JSON documents directly to the index via the REST API or SDK. The push model gives you the lowest-latency updates and full control, at the cost of writing the ingestion code yourself.

For **non-text media**, remember that Search indexes *text and vectors*, not raw audio or video. The pattern is to **preprocess first, then index**: transcribe audio/video with **[Azure Speech](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/overview)** or **Azure AI Video Indexer** (or a **Content Understanding** audio/video analyzer), extract key frames or captions for images, and then push or pull the resulting text and embeddings into the index. Images can also be ingested directly into a multimodal pipeline (see enrichment below), where they are OCR’d, captioned/verbalized, and embedded.

### Semantic, hybrid, and vector search for grounding

Know the four query types and when each is the “best answer”:

- **Full-text (lexical, BM25).** Inverted-index keyword search with filters, facets, scoring profiles, stemming, and synonyms. Best for exact terms, codes, and filterable metadata.
- **Vector search.** Nearest-neighbor search over embedding fields, using **HNSW** (approximate) or **exhaustive KNN**, with **cosine, dot-product, or Euclidean** similarity. Best for semantic similarity, multilingual matching, and matching across content types. With **integrated vectorization**, you can submit text and let a configured **vectorizer** embed the query at search time.
- **Hybrid search.** Runs BM25 and vector search **in parallel** and fuses results with **Reciprocal Rank Fusion (RRF)**. This is the production default for RAG because it combines vector recall with keyword precision.
- **Semantic ranking.** A premium **L2 reranker** that re-orders the top results of a BM25 or hybrid query using Microsoft language models, and can return **captions/highlights** and extractive **answers**. The strongest classic-RAG relevance recipe Microsoft recommends is **hybrid search + semantic reranker**.

**Agentic retrieval** sits above all of these. Instead of a single-shot query, a **knowledge agent / knowledge base** uses an LLM to **decompose a complex or conversational question into subqueries**, runs them **in parallel** (over indexed and remote sources), and returns merged content **plus citations and an activity/reference log**. Choose agentic retrieval for new chatbot/agent scenarios that need the highest relevance on complex, multi-part questions; choose classic hybrid+semantic RAG when you need GA-only features, lower latency, simplicity, or fine manual control.

### Enrichment with custom and built-in skills (text, images, layout)

When an indexer ingests content, you attach a **skillset**, an ordered pipeline of **skills** that enrich each document and emit JSON that is mapped into index fields (via **output field mappings**) or projected into a **knowledge store**.

- **Built-in skills.** Common ones to know: **OCR skill** (printed/handwritten text from images and scanned PDFs), **Image Analysis skill** (tags, captions, objects), **Document Intelligence Layout skill** (tables, structure), **Text Split skill** (chunking by page/sentence/token), **Key Phrase Extraction**, **Entity Recognition**, **PII Detection**, **Language Detection**, **Translation**, and the **Azure OpenAI Embedding skill** (generates vectors during indexing).
- **Content Understanding skill (preview).** A multipurpose enrichment skill that runs Foundry analyzers to extract text, tables, and figures and emit **LLM-ready markdown**, including merging tables across pages. It is the strongest enrichment choice for complex, layout-heavy, or multimodal documents.
- **Custom skills.** When no built-in skill fits, implement a **custom skill** as an **Azure Function or Web API** that conforms to the custom-skill request/response contract, and slot it into the skillset like any other skill.

A frequently tested operational detail: to feed images to the OCR or Image Analysis skill from a Blob indexer, set **`imageAction: "generateNormalizedImages"`** so images are extracted to `/document/normalized_images` for downstream skills to consume.

### RAG ingestion flow, including OCR

A robust RAG ingestion pipeline on Azure has a consistent shape:

1. **Acquire** raw content via push or an indexer’s data source.
2. **Crack and OCR.** For PDFs/images, run the **OCR (Read) skill** or **Document Intelligence Layout** to recover text, including from scanned documents and embedded images. OCR is the step that makes scanned and image-only content searchable at all.
3. **Chunk.** Split content into semantically coherent pieces (often ~500–2000 tokens) with the **Text Split skill** or **Content Understanding** for layout-aware chunking.
4. **Enrich.** Add metadata (titles, summaries, entities, page numbers, source URLs).
5. **Vectorize.** Generate embeddings with the **Azure OpenAI Embedding skill** (integrated vectorization).
6. **Index / project.** Write text + vectors + metadata into the index. For chunked content, Microsoft recommends a **single child-chunk index with repeated parent fields**, implemented via **index projections**.

For traceable grounding, design the index so each chunk carries a **retrievable text field**, a **source/URL field**, and ideally a **title field**, so agents can return useful **citations**.

### Connecting retrieval pipelines to workflows and agent tools

The retrieval layer is only useful when something consumes it. Know the integration paths:

- **Foundry Agent Service tools.** An Azure AI Search index can be attached to an agent as a **knowledge/grounding tool**, or surfaced through **Foundry IQ** as a permission-aware knowledge base. The **file search** tool is the easy path when you upload files directly and want managed parsing/chunking/embedding/retrieval; **Azure AI Search** is the path when you already own the index and need control.
- **Connections.** A Foundry project connects to Search either as a **project connection** or by referencing an **index asset ID**, depending on the API surface.
- **Workflows and event-driven pipelines.** **Azure Logic Apps** has Azure AI Search connectors (and can now act as a **remote knowledge source** for vector stores), so search results can trigger or feed downstream automation. Agent frameworks (Microsoft Agent Framework, Semantic Kernel) can use the index directly as a knowledge source.

The exam-best mental model: the orchestrator (agent or app) issues a hybrid/semantic or agentic query, gets back **content + metadata + citations**, and packages those into the model prompt for a grounded answer.

## Extract content from documents

### Multimodal pipelines combining OCR, layout, and field extraction

This sub-objective is squarely **Azure AI Document Intelligence** territory when the goal is deterministic field extraction.

- **Read model / OCR.** Extracts printed and handwritten text, with language detection and page/line/word structure.
- **Layout model.** Adds **tables, selection marks, paragraphs, and structural elements**, with bounding regions, the backbone of layout analysis.
- **Prebuilt models.** Ready-made extractors for **invoices, receipts, IDs, business cards, W-2/tax forms, contracts, health insurance cards**, and more, returning typed key-value fields with **confidence scores**.
- **Custom models.** Train on your own documents when prebuilt models don’t fit: **custom template** (consistent layouts) and **custom neural** (varied layouts); combine several into a **composed model**. **Query fields** let you extract additional ad-hoc fields beyond a model’s standard schema.

The “multimodal pipeline” framing the exam uses means **chaining**: OCR recovers text, layout reconstructs structure and tables, and field extraction (prebuilt or custom) pulls typed values, and the combined JSON output flows into a search index or an agent. Confidence scores are the hook for **human-in-the-loop review** of low-confidence fields.

### Producing clean, grounded representations with Content Understanding

**Azure Content Understanding** is the generative, schema-first complement to Document Intelligence. It uses generative AI to process **documents, images, audio, and video** into a **user-defined output format**, and its central building block is the **analyzer**, which defines *what* content to process, *which* elements to extract, and *how* to structure the result.

Know the analyzer taxonomy and modes:

- **Analyzer types:** **prebuilt** analyzers, including general-extraction, **RAG-tuned** (output optimized for grounding), and **industry/domain-specific** analyzers, plus **custom** analyzers you define with a schema. (Microsoft documents these primarily as *prebuilt vs. custom*; the finer "base / RAG / domain-specific" grouping is a useful study framing rather than an official taxonomy.)
- **Standard vs. pro mode.** **Standard mode** handles **documents, images, video, audio, and text**, optimized for lower cost/latency and **single-file, straightforward extraction**. **Pro mode** targets **multi-step reasoning, multiple input files, and reference data**, but the critical exam caveat is that **pro mode is currently document-only**. If a scenario says “take one image or video and extract fields,” answer **standard**; if it says “reason across multiple files against reference data,” answer **pro**, and remember the document-only limitation. Two further pro-mode limits worth memorizing: pro mode **does not return confidence scores or grounding**, and it **supports only `classify` and `generate` fields, not `extract`** (so even though it is document-based, the deterministic `extract` method is unavailable in pro mode).
- **Output.** Analyzers emit **markdown and/or structured JSON fields and segments** that are **RAG-ready with no post-processing**, tables become markdown tables, figures get descriptions, and chunks carry source context.

The reason this is grounded and “clean”: instead of a free-form chat answer about a document, you get **stable, schema-conformant output** that downstream agents and RAG systems can consume reliably and cite.

### Implementing analyzers for structured or markdown output

When you author an analyzer, follow Microsoft’s field-design best practices, because “why is extraction quality poor?” is a classic exam question:

- Write **detailed field descriptions** and include **aliases** for terms that vary across documents.
- Use **arrays and objects** to model repeated or nested data rather than flattening it.
- Choose the extraction **method per field**, **`extract`**, **`generate`**, or **`classify`**. Remember that **`extract` is only supported for document analyzers**, which reinforces that product boundaries differ by modality.
- Prefer **markdown output** when the consumer is an LLM (it preserves tables and headings), and **typed JSON fields** when the consumer is downstream software that needs a stable contract.

## High-yield facts to memorize

- **Pull (indexer) vs. push:** indexers crawl supported sources on a schedule (as often as every 5 minutes) and can run skillsets; push gives lowest-latency, full-control ingestion for fast-changing or unsupported data.
- **Hybrid + semantic reranker** is the classic-RAG relevance default; **agentic retrieval** is the answer for complex, conversational, multi-source grounding with citations.
- **RRF** fuses keyword and vector results in hybrid search; the **semantic ranker** is an L2 reranker that also returns captions and answers.
- **Integrated vectorization** = indexer + skillset + chunking skill + Azure OpenAI embedding skill + a **vectorizer** for query-time embedding; chunked content uses a **child-chunk index with repeated parent fields via index projections**.
- **OCR is mandatory** to make scanned/image-only documents searchable; set **`imageAction: generateNormalizedImages`** to feed images to OCR/image skills.
- **Document Intelligence** = deterministic typed field extraction with confidence scores (forms, invoices, IDs); **Content Understanding** = generative schema/markdown extraction across modalities for RAG/agents.
- **Content Understanding standard mode** = all modalities, single-file; **pro mode** = multi-file, reference data, advanced reasoning, **document-only today**.
- **`extract` is document-only**; image/video/audio analyzers use **`generate`/`classify`**.
- For citations, store a **retrievable text field, a source/URL field, and a title field** per chunk.
- **Custom skills** are **Azure Functions / Web APIs** that conform to the custom-skill contract.

## Practice questions with answer key

**Question:** A document library in Blob Storage is updated nightly, and you want Azure AI Search to OCR scanned PDFs, chunk them, generate embeddings, and refresh automatically. Push or pull?
**Answer:** **Pull (indexer)** with a Blob data source and a **skillset** (OCR → Text Split → Azure OpenAI Embedding), on a schedule with change tracking. The push model is unnecessary because nightly updates fit an indexer schedule.

**Question:** Users ask complex, multi-part questions and you need the highest-relevance grounded answers with citations from several sources. Classic RAG or agentic retrieval?
**Answer:** **Agentic retrieval**, because it decomposes the query into parallel subqueries across indexed and remote sources and returns merged content plus citations and an activity log. Use classic hybrid+semantic only if you need GA-only features, lower latency, or simpler control.

**Question:** Retrieval returns relevant documents but ranking is “correct but not ideal.” What do you add before switching models?
**Answer:** Add **hybrid search with the semantic reranker** (and tune scoring profiles). This is a retrieval-tuning problem, not a model problem.

**Question:** You must extract the invoice number, total, and line items from thousands of vendor invoices with confidence scores. Which service?
**Answer:** **Azure AI Document Intelligence**, using the **prebuilt invoice model** (or a **custom neural** model for unusual layouts). It returns typed fields with confidence scores suitable for human-in-the-loop review.

**Question:** You need RAG-ready markdown, including tables merged across pages and figure descriptions, from complex mixed-content reports. Which tool and why?
**Answer:** **Azure Content Understanding** (or the **Content Understanding skill** inside a Search skillset), because it emits LLM-ready markdown and structured fields with no post-processing, unlike legacy layout skills.

**Question:** A scenario asks you to extract fields by reasoning across multiple related image files against reference data using Content Understanding pro mode. What’s the catch?
**Answer:** **Pro mode is currently document-only.** For images today, use **standard mode**; pro-mode cross-file reasoning applies to documents.

**Question:** How do you make image-only PDFs searchable in an indexer pipeline?
**Answer:** Set **`imageAction: "generateNormalizedImages"`** on the indexer so extracted images land in `/document/normalized_images`, then run the **OCR skill** over them; its text output (`/document/normalized_images/*/text`) is mapped into searchable index fields via output field mappings.

**Question:** Your enrichment needs a lookup against an internal API that no built-in skill provides. What do you build?
**Answer:** A **custom skill** implemented as an **Azure Function or Web API** that conforms to the custom-skill request/response schema, added to the skillset.

**Question:** What three index fields should each chunk carry so an agent can return useful citations?
**Answer:** A **retrievable text field**, a **source/URL field**, and a **title field**.

**Question:** When do you choose `generate` vs `extract` for an analyzer field?
**Answer:** Use **`extract`** for values literally present in the source (**document analyzers only**), **`generate`** when the value must be synthesized/summarized from content, and **`classify`** to assign a category. Modality determines which methods are available.

## A realistic study plan for this domain

Study in the order the official objectives use, building one small end-to-end pipeline per cluster:

1. **Ingestion + indexing.** Create an Azure AI Search index, configure a **Blob indexer with a skillset** (OCR → Text Split → embedding), and confirm scanned PDFs become searchable.
2. **Query types.** Run the same question as full-text, vector, hybrid, and **hybrid + semantic**, and compare results so you can articulate *why* hybrid+semantic wins.
3. **Agentic retrieval.** Stand up a knowledge base / knowledge agent and observe query decomposition and citation output.
4. **Document extraction.** Extract typed fields with a **Document Intelligence** prebuilt model, then re-do a messy document with a **Content Understanding** analyzer and compare deterministic fields vs. generative markdown.
5. **Wire it to an agent.** Attach the index (or Foundry IQ) to a Foundry agent and verify grounded answers with citations.

## Further reading

Microsoft Learn pages for this domain (verified June 2026):

- [Azure AI Search documentation](https://learn.microsoft.com/en-us/azure/search/)
- [Introduction to Azure AI Search](https://learn.microsoft.com/en-us/azure/search/search-what-is-azure-search)
- [What is Azure AI Document Intelligence in Foundry Tools?](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/overview?view=doc-intel-4.0.0)
- [Document Intelligence document processing models](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/model-overview?view=doc-intel-4.0.0)
- [What is Azure Content Understanding in Foundry Tools?](https://learn.microsoft.com/en-us/azure/ai-services/content-understanding/overview)
- [What is Image Analysis? (Azure Vision)](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/overview-image-analysis)
- [What is Azure Speech?](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/overview)
- [Exam AI-103 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-103)
