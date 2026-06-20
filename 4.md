# AI-103 Study Guide for Text Analysis and Speech

## Exam context and how to use this guide

As of June 2026, AI-103 is the **Microsoft Certified: Azure AI Apps and Agents Developer Associate (beta)** exam. Microsoft describes it as an intermediate certification for Azure AI engineers and developers who build, manage, and deploy agents and AI solutions by using Python and Microsoft Foundry. The certification page also notes that the exam is still in beta, and the practice assessment is not yet available while the exam remains in beta.

The exam section you asked about is a **single domain, “Implement text analysis solutions,” weighted at 10–15%** of the exam. A common point of confusion is to treat speech as its own separate 10–15% domain; it is not. In Microsoft’s official skills outline (skills measured as of April 16, 2026), **“Implement speech solutions” is a subsection *inside* the text-analysis domain**, not a standalone domain. The genuinely separate fifth domain on the exam is **“Implement information extraction solutions” (10–15%)**, which is covered in a different guide. So when you budget study time, treat text analysis and speech together as one 10–15% slice, and do not double-count speech. Keep in mind that because AI-103 is still in beta (skills measured as of April 16, 2026; certification page last updated June 18, 2026), the domain weightings and skill bullets can still change before the exam reaches general availability, so re-check the official skills outline close to your exam date. Microsoft’s study guide explicitly lists the text-analysis tasks as extracting entities, topics, summaries, and structured JSON outputs; detecting sentiment, tone, safety issues, and sensitive content; translating text with Azure Translator or LLM-powered flows; and customizing outputs for domain tasks such as compliance summarization and domain extraction. It then lists the speech tasks (within that same text-analysis domain) as speech-to-text and text-to-speech workflows, speech as an agent modality including custom speech models, multimodal reasoning from audio inputs, and speech translation by using language models and Foundry Tools.

Use this guide in a very exam-focused way: first learn **which Azure service or Foundry capability fits which scenario**, then learn the **tradeoffs and limitations** that Microsoft calls out in the docs, and finally practice by building small end-to-end flows. That matches how Microsoft positions the exam overall and how the AI-103 course is framed for developers building AI apps and agents on Azure.

## Service map you need to know

For this part of AI-103, you should be fluent in six building blocks. **Azure Language** covers classical NLP capabilities such as language detection, named entity recognition, custom NER, PII detection, and health text analytics, and Microsoft notes that these capabilities are available through Foundry, REST APIs, client libraries, and even the Azure Language MCP server for agent scenarios.

**Azure OpenAI and Foundry models** are the core choice when you need generative prompting, flexible extraction, schema-constrained JSON, reasoning over mixed modalities, or voice-native agent experiences. Microsoft’s structured outputs documentation says structured outputs are recommended for function calling, extracting structured data, and multi-step workflows because they follow a supplied JSON Schema, while older JSON mode only guarantees valid JSON and not schema conformance.

**Azure AI Content Safety** is the right service when the problem is harmful or adversarial content rather than ordinary sentiment classification. Microsoft describes it as the service for detecting harmful user-generated and AI-generated content, and its Prompt Shields feature is specifically designed to detect and block adversarial prompts and risky documents before generation. Foundry guardrails use four harm categories, hate, sexual, violence, and self-harm, with severity levels safe, low, medium, and high.

**Azure Translator** is your dedicated multilingual text and document translation service. Current Microsoft docs describe it as a cloud-based neural machine translation service for text and documents, with support for more than 100 languages and dialects, customizable models through Custom Translator, and document translation that preserves structure and formatting. The newer text translation API also supports model choice between NMT and selected LLMs, adaptive custom translation, and tone controls for LLM-based translation flows.

**Azure Speech** is the main service for speech-to-text, text-to-speech, custom speech, speech translation, and voice synthesis. Microsoft’s Speech overview emphasizes real-time transcription, fast transcription, batch transcription, neural voices, custom voice, speech translation, and custom speech models for jargon-heavy or noisy audio.

**Azure Content Understanding** matters whenever the scenario is “take messy content and produce structured, grounded output.” Microsoft says it uses generative AI to process documents, images, video, and audio into user-defined output formats, and that analyzers define what content to process, which elements to extract, and how to structure the output as markdown, JSON fields, or segments. That makes it especially relevant for multimodal extraction questions on AI-103.

One high-yield exam nuance is that Microsoft is clearly steering **new development toward Foundry models** for several older Language features. The documentation for key phrase extraction, summarization, and sentiment analysis all says these features are scheduled to retire on **March 31, 2029**, and recommends that new projects move to Microsoft Foundry models. Two related retirement dates are worth noting for precision: **Entity Linking retires earlier, on September 20, 2028**, and **Azure Language Studio retires on March 20, 2027**, both consistent with the broader “move to Foundry models” direction but on their own timelines. For exam preparation, that means you should know both the older Azure Language capabilities and the newer LLM-first patterns, but you should expect Microsoft’s architecture direction to be LLM-centric.

## Text analysis study guide

The first recurring exam skill is **extracting entities, topics, summaries, and structured JSON outputs**. For classical extraction, Azure Language gives you **prebuilt NER** for common entity types and **custom NER** when your entities are domain-specific, such as legal clauses or finance-specific entities. For summaries, Azure Language supports text, conversation, and native document summarization. For generative extraction, Microsoft recommends **structured outputs** in Azure OpenAI because they can force the model to match a JSON Schema, which is exactly what you want when the downstream system expects reliable machine-readable output.

A practical exam rule is this: if the question asks for a **stable, reusable schema**, choose **structured outputs** over plain prompting and over JSON mode. Microsoft explicitly says structured outputs guarantee adherence to the supplied schema, while JSON mode only guarantees valid JSON. That distinction is the kind of service-selection detail Microsoft likes to test. Also remember the current limitation that structured outputs are not supported in bring-your-own-data scenarios in the cited documentation.

For **topic extraction**, you should understand both the older **key phrase extraction** approach and the newer LLM-based patterns. Key phrase extraction still exists in Azure Language and is useful for fast topic surfacing, but Microsoft recommends Foundry models for new work. On the exam, if the scenario wants lightweight topic surfacing from plain text, key phrase extraction is still fair game; if it wants richer domain-aware topic labels or nested output, LLM prompting with structured outputs is the stronger choice. That second point is an inference from the exam wording plus Microsoft’s current product guidance.

For **sentiment, tone, safety issues, and sensitive content**, break the objective into separate subproblems. **Sentiment and opinion mining** belong to Azure Language, which returns positive, neutral, and negative labels and confidence scores at the sentence and document level, with opinion mining for aspect-level detail. **Sensitive content** belongs mainly to **PII detection**, which identifies, classifies, and redacts sensitive data across text, conversations, and native documents. **Safety issues** belong to **Azure AI Content Safety** and Prompt Shields.

The one item that is a little less direct in the docs is **tone detection**. There is no equivalent first-class Azure Language “tone detector” in the materials cited here. The most exam-ready interpretation is that **tone** is something you classify with an LLM by using a carefully designed system message plus structured outputs, while **sentiment** remains the dedicated Azure Language capability. Microsoft’s system-message guidance says you can steer role, tone, output format, and safety constraints through the system message, which aligns well with tone classification and tone-controlled outputs. That is an inference, but it is strongly supported by the exam wording and the current docs.

For **translation**, know the split between **Translator** and **LLM-powered translation flows**. Azure Translator is the dedicated service for high-scale multilingual translation and has strong support for real-time text translation, document translation, transliteration, language detection, glossaries, and custom translation. The current text translation docs also note that the latest API supports choosing NMT or an LLM deployment, adaptive custom translation, and tone or gender controls on LLM-based translation. Document translation is especially important because it can preserve layout and formatting, and the portal currently supports only synchronous single-file document translation while the REST API and SDKs support batch workflows.

For **domain customization**, learn two patterns. The first is **task-specific Azure Language customization**, such as **custom text classification** and **custom NER**, which both use labeled data and an iterative label-train-evaluate-deploy lifecycle. The second is **LLM customization by prompt design and output constraints**, including system messages and structured outputs. Microsoft’s docs say custom text classification is for user-defined classes, while system messages let you enforce role, tone, output format, and safety constraints. For AI-103, that means “compliance summarization” may be best answered with LLM prompting and structured outputs, while “classify text into internal policy categories” may be better answered with custom text classification if the category set is fixed and well labeled.

A final high-yield concept is **Content Understanding**. Microsoft describes it as a way to define analyzers that process content and return structured output such as markdown, JSON fields, and segments. If the exam scenario asks for a configurable analyzer that extracts structured data from complex text, audio, or other unstructured content, Content Understanding is often the intended answer rather than plain chat prompting.

## Speech and audio study guide

The exam’s speech objective starts with the basics: **speech to text** and **text to speech** for agentic interactions. Microsoft’s Speech docs make the STT choices explicit: **real-time transcription** for streaming audio, **fast transcription** for synchronous prerecorded audio, and **batch transcription** for large asynchronous workloads. For TTS, Azure Speech supports neural voices, custom voices, long-form batch synthesis, and SSML for fine-grained control of voice, pace, pronunciation, pitch, pauses, and emphasis.

When the exam says **speech as an agent modality**, think in two architectures. The first is the classic pipeline: **Speech STT → language model → Speech TTS**. Microsoft even documents a speech-to-speech chat pattern where Speech recognizes audio, sends text to Azure OpenAI, and then synthesizes the response back to the user. The second is a **voice-native model architecture** using **Azure OpenAI Realtime API**, which Microsoft says is designed for low-latency speech-in, speech-out conversations such as voice assistants, customer support agents, and real-time translators.

That service-selection choice is important. Microsoft’s Azure Architecture Center says Azure OpenAI audio models are the right fit when you want audio together with language understanding, reasoning, or generation in a **single model call**, while Azure Speech remains the dedicated service for transcription, synthesis, and translation workflows. In exam language, choose **Azure OpenAI audio or Realtime** when the problem is a reasoning-heavy voice agent, but choose **Azure Speech** when the problem is production speech plumbing, voice quality, speech translation, STT throughput, or speech customization.

For **custom speech models**, Microsoft says custom speech improves recognition accuracy when base models are not enough because of ambient noise, industry-specific jargon, or specialized pronunciation. The customization workflow is iterative: create a project, upload test data, measure **word error rate**, train a model with text and optionally audio data, test, and deploy. That means any exam question about domain-specific transcription accuracy in healthcare, field service, legal, or contact center audio should make you think **custom speech**.

For **multimodal reasoning from audio inputs**, know at least two modern Azure options. First, Microsoft’s audio-enabled Azure OpenAI chat/completions docs say some models support **text, audio, and text + audio**, extending chat completions to audio analysis and voice interactions. Second, Azure Content Understanding audio analyzers can extract **transcripts, summaries, sentiment, key topics, and structured fields** from conversational audio. This second option is especially exam-friendly because it turns raw audio into grounded, structured output rather than just a transcript.

For **speech translation**, Azure Speech supports **real-time speech-to-text translation** and **speech-to-speech translation** of audio streams, including multiple target languages. In parallel, Microsoft now offers **LLM Speech**, which is an API where an LLM enhances a speech model to improve quality, contextual understanding, multilingual support, and prompt-tuning capabilities for transcription and translation. That means the likely exam distinction is: use **Speech translation** for mainstream real-time translation pipelines, and consider **LLM Speech** when the scenario emphasizes richer context, prompt-guided behavior, or higher quality on prerecorded audio.

One nice cross-service detail to remember is that **Custom Translator** can also customize **speech translation** when used with Azure Speech. So if the exam mentions enterprise-specific bilingual terminology in spoken translation, the best answer may involve both **Azure Speech** and **Custom Translator** together.

## High-yield study plan and labs

The official learning resources line up well with this exam slice. Microsoft’s AI-103 course is the umbrella resource for developers building AI apps and agents on Azure. The **Develop natural language solutions in Azure** learning path specifically includes modules for analyzing text, building a text-analysis agent with the Azure Language MCP server, developing a speech-capable generative AI application, and creating speech-enabled apps with Azure Speech. The training catalog also has a dedicated module for translating text and speech with Foundry Tools.

If you want a compact study sequence, use this order:

- **Start with text analysis fundamentals** by working through *Get started with text analysis in Azure* and *Analyze text with Azure Language in Foundry Tools*. These cover language detection, NER, and PII, which are foundational service-selection topics on the exam.
- **Then study structured outputs and system messages** in Azure OpenAI. This is the fastest path to mastering extraction into machine-readable JSON and domain-controlled prompting.
- **Then cover safety and moderation** with Azure AI Content Safety, Prompt Shields, and Foundry guardrails.
- **Then cover translation** with Azure Translator text translation, document translation, glossaries, and Custom Translator, plus the Microsoft training module on translating text and speech.
- **Then master speech** through the Speech module, STT/TTS docs, SSML, speech translation, and custom speech overview.
- **Finish with multimodal audio reasoning** by reviewing Azure OpenAI audio docs and Content Understanding audio analyzers.

The best lab set for this domain is not large; it is **targeted**. Build one small project per exam bullet. For text analysis, create a structured-output extractor that turns customer emails into JSON with fields like issue type, entities, urgency, sentiment, and next action; then add PII redaction and Content Safety checks before the result is stored. For translation, build a support-message translator with glossary control, plus a second version that uses an LLM-based translation flow when tone matters. For speech, build a simple loop with STT → model → TTS, then a second version with Azure OpenAI Realtime or audio-enabled chat, then a third version that transcribes jargon-heavy audio with custom speech. For multimodal audio, run a recorded call through Content Understanding and require a transcript, summary, topics, and sentiment-style fields in structured output. These labs map directly to Microsoft’s documented capabilities and force you to make the same architecture choices the exam is likely to test.

If you only have one weekend, prioritize these ideas in order: **structured outputs vs JSON mode**, **Azure Language vs LLM extraction**, **PII vs Content Safety vs sentiment**, **Translator vs Speech translation**, **Speech service vs Azure OpenAI audio**, and **custom speech for domain jargon**. Those are the most testable decision boundaries in the documentation.

## Practice questions with answer key

**Question:** You need a model to extract contract metadata into a response that must always match a specific schema used by downstream software. Would you use JSON mode or structured outputs?

**Answer:** Use **structured outputs**. Microsoft says structured outputs enforce a supplied JSON Schema, whereas JSON mode only guarantees valid JSON and does not guarantee that the output matches a specific schema.

**Question:** You need to classify user reviews as positive, neutral, or negative and also identify which product feature each opinion refers to. Which service is the most direct fit?

**Answer:** Use **Azure Language sentiment analysis with opinion mining**. Microsoft documents sentiment labels and confidence scores at sentence and document level, and opinion mining adds aspect-level detail tied to words or attributes in the text.

**Question:** You need to redact credit-card numbers, phone numbers, and other sensitive fields from chat transcripts before they are stored. What should you choose?

**Answer:** Use **PII detection** in Azure Language. Microsoft says it identifies, classifies, and redacts sensitive data across text, conversations, and native documents, returning structured output plus redacted results.

**Question:** Your application must block jailbreak-style prompts before a generation step even happens. What capability is being tested?

**Answer:** That is **Prompt Shields** in Azure AI Content Safety. Microsoft describes Prompt Shields as a unified API that detects and blocks adversarial user input attacks on LLMs by analyzing prompts and documents before content is generated.

**Question:** You must translate PowerPoint and Word files while preserving layout and formatting. The business also wants glossary support. Which Azure service is the best fit?

**Answer:** Use **Azure Translator document translation**. Microsoft says document translation preserves original structure and formatting, supports custom glossaries, and supports both asynchronous batch and synchronous single-file workflows.

**Question:** You are building a low-latency voice assistant where users speak and the model responds with speech in near real time. Should you start with batch STT plus TTS, or a realtime audio model?

**Answer:** The most direct answer is **Azure OpenAI Realtime API** if the scenario emphasizes live speech-in, speech-out conversation and low latency. Microsoft describes it as designed for exactly those voice-assistant and customer-support scenarios.

**Question:** A hospital transcription system performs poorly because the audio contains medical jargon and specialized pronunciation. What should you add?

**Answer:** Add **custom speech**. Microsoft says custom speech improves recognition accuracy when base models are insufficient because of ambient noise, industry jargon, or specialized vocabulary, and that you evaluate progress with metrics such as word error rate.

**Question:** You need to turn call-center recordings into transcript, summary, topics, and structured insight fields for downstream analytics. Is plain STT enough?

**Answer:** Plain STT is not the best fit by itself. The stronger answer is **Azure Content Understanding audio analyzers**, because Microsoft says they can produce transcription and diarization plus structured fields such as summaries, sentiment, and key topics, and they can map unstructured dialogue into meaningful structured insights.

**Question:** When would you choose custom text classification over a prompted LLM classifier?

**Answer:** Choose **custom text classification** when the classes are stable, user-defined, and you can invest in labeling data and iterating on train/evaluate/deploy cycles. Use an LLM when the task needs more flexible reasoning, richer output, or prompt-defined behavior and format constraints. The first part is directly documented by Microsoft; the second part is an inference from Microsoft’s guidance on structured outputs and system-message design.

**Question:** The exam mentions “tone.” If the scenario asks you to detect or enforce a tone such as formal, informal, apologetic, or authoritative, what is the safest current study assumption?

**Answer:** The safest assumption is that **tone** is handled with **LLM prompting plus structured outputs or prompt constraints**, not with the dedicated Azure Language sentiment API. Microsoft’s system-message guidance explicitly says you can set tone and output format through the system message, while Azure Language sentiment is documented around sentiment labels and opinion mining rather than tone labels.

If you can answer those ten questions without hesitation and can explain **why** each choice is correct, you will be in a strong position for this AI-103 exam slice. The official Microsoft docs and training for this area consistently emphasize the same architectural patterns: use **dedicated services** where the problem is narrow and well defined, use **Foundry models and structured outputs** where the problem is flexible and generative, and use **Speech, Translator, Content Safety, and Content Understanding** when the scenario crosses modalities or requires production-grade operational features.
