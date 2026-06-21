# AI-103 Study Guide for Implementing Computer Vision Solutions

> **Slide deck:** A companion presentation for this guide is available at [`Slides/5-AI-103_Computer_Vision_Mastery.pptx`](Slides/5-AI-103_Computer_Vision_Mastery.pptx).

> **Podcast episode:** A companion audio deep-dive for this guide is available at [Azure Vision Architecture for AI-103](https://stbighatpodcasts.blob.core.windows.net/podcasts/5-Azure_Vision_Architecture_for_AI-103.m4a).

## Scope and exam framing

This guide is aligned to **Exam AI-103: Developing AI Apps and Agents on Azure**. In Microsoft’s current study guide, the domain **“Implement computer vision solutions”** is weighted at **10–15%** of the exam, and the listed objectives exactly include the image generation, video generation, multimodal understanding, Content Understanding, and responsible AI topics you provided. Microsoft also states that most exam questions target general availability features, but the exam can still include preview features if they are commonly used.

AI-103 is not the older Azure AI Engineer exam. Note the naming precisely: **"Developing AI Apps and Agents on Azure"** is the title of the *study guide page* for the exam, but the credential it leads to is **"Microsoft Certified: Azure AI Apps and Agents Developer Associate,"** and the associated instructor-led course is **"Develop AI apps and agents on Azure" (AI-103T00-A)**. Microsoft identifies the exam by its code (AI-103) rather than by a separate verb-phrase title, so don't treat the gerund phrase as the certification name. The current credential path positions AI-103 around **AI apps and agents on Azure**, with emphasis on **Python**, **generative AI**, **multimodal models**, and **Microsoft Foundry**. That matters for study strategy: instead of memorizing only classic Computer Vision APIs, you should be ready to choose among **[Azure OpenAI](https://learn.microsoft.com/en-us/azure/foundry/concepts/foundry-models-overview)**, **[Azure Vision](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/overview-image-analysis)**, **[Azure Content Understanding](https://learn.microsoft.com/en-us/azure/ai-services/content-understanding/overview)**, **[Azure AI Search](https://learn.microsoft.com/en-us/azure/search/)**, **[Azure AI Content Safety](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/overview)**, and sometimes **Azure AI Video Indexer**, depending on the scenario.

The single most useful mental model for this objective is this: use **Azure OpenAI / Foundry** when the task is **generation or open-ended reasoning**, use **Azure Vision** when you need **specific visual features** such as OCR, captions, or object detection, use **Content Understanding** when you need **schema-based structured extraction from images or video**, use **Azure AI Search** when you need **grounded multimodal retrieval**, and use **Content Safety** when you need **moderation, guardrails, and risk detection**.

## Image and video generation on Azure

For **image generation**, Microsoft’s current Azure OpenAI guidance says the old `dall-e-3` deployment path is no longer the one to study as your primary choice: **DALL-E 3 was retired on March 4, 2026**, and Microsoft directs customers to the **`gpt-image-`** series instead. The current capability table also shows that **GPT-Image-2** is generally available, supports higher-resolution generation, and improves editing performance, while the `gpt-image-1.x` family supports text-plus-image inputs and mask-based editing workflows. Be precise about status within the `gpt-image-1` family: the original `gpt-image-1` (version `2025-04-15`) is still **Preview**, while `gpt-image-1-mini` and `gpt-image-1.5` are **GA**.

For the exam objective **“Implement a solution that generates images from text prompts and reference media,”** the key concept is that Azure OpenAI image models accept **text prompts and optional images**. In practice, you deploy an image-capable model in Foundry, send a generation request, and receive the result as base64 image data. The exam is likely to test whether you understand that **reference media is not a separate product** here; it is part of the model input pattern for image generation and editing.

For **image editing**, the core study points are **inpainting**, **mask-based edits**, and prompt-driven changes. Microsoft’s sample flow shows a separate **edits** endpoint in which you pass the source image, an edit prompt, and optionally a **mask**. The documentation explicitly notes that the **mask must be the same size as the input image**, which is exactly the kind of operational detail that often shows up in certification questions.

For the objective **“Select and apply appropriate generation and editing controls provided by the platform,”** memorize the main image controls: **size**, **quality**, **number of images (`n`)**, **transparency/background** for supported models, and in some flows **streaming partial images**. Microsoft documents that GPT-image models support `low`, `medium`, and `high` quality; multiple image counts per call; and, for supported models, transparent PNG backgrounds. It also notes that square images are faster, and that **GPT-Image-2** supports arbitrary resolutions within documented constraints: both edges must be **multiples of 16 px**, the **long edge can reach 3,840 px (4K)**, the **aspect ratio can go up to 3:1**, and the **total pixel count must fall between 655,360 and 8,294,400**.

For **video generation**, the current Azure/OpenAI answer is **Sora 2** in preview. Microsoft documents three important input modes: **text → video**, **image + text → video**, and **video + text → video**. That directly maps to the exam language about generating videos from **text prompts and reference media**. If the scenario is “animate this still image,” think **image + text prompt**. If it is “extend or modify an existing clip,” think **video + text prompt**.

Study the **video workflow shape** as well. Microsoft describes video generation as a **job-based process**: create the job, poll its status, then retrieve the generated video. It is not a simple synchronous one-call image-style workflow. The documentation also gives practical timing guidance: generation usually takes **1 to 5 minutes** depending on duration and resolution, and supported resolutions include **480×480, 720×720, 1080×1080, 1280×720, and 1920×1080**.

For **editing generated videos**, there are two high-yield patterns to remember. The first is **remix**, where you refer to a prior generated video by `video_id` and make a focused prompt-based change while preserving the existing structure. Microsoft explicitly recommends **one clearly articulated adjustment** for best fidelity. The second pattern is using **reference media and inpaint items**, where you specify frame placement, crop bounds, and variant counts for more directed generation or modification.

A likely exam trap is to forget that Azure’s generation stack already includes **responsible AI protections**. Microsoft states that image generation models include built-in responsible AI protections, and the Sora 2 overview says the service adds moderation and Azure-specific safeguards such as content filtering and abuse monitoring. The Sora 2 overview also notes a major current restriction: **Sora 2 blocks all IP and photorealistic content**.

## Multimodal visual understanding

The objective **“Build a solution that analyzes visual context by using multimodal models”** maps most directly to **vision-enabled chat models** in Azure OpenAI. Microsoft defines these as multimodal models that can **analyze images and provide textual responses about them**, and the current documentation lists supported families including the **o-series reasoning models**, **GPT-5 series**, **GPT-4.1 series**, **GPT-4.5**, and **GPT-4o series**. If the task is open-ended reasoning such as “What is happening in this image?” or “What safety checklist violations do you see here?”, this is the natural tool.

A second exam-worthy detail is that these vision chat requests are still basically **chat completion calls** with image content embedded in the message payload. Microsoft’s examples show the user message combining **text plus an `image_url` object** and using normal chat parameters such as `max_tokens`. That means the right mental model is not “special vision service,” but rather “a multimodal chat completion request.”

When the question is about controlling **how much visual detail** the model should use, remember the `detail` parameter in the image payload. Microsoft documents three values: **`low`**, **`high`**, and **`auto`**. `low` reduces cost and latency by processing a lower-resolution form of the image, while `high` uses more detailed processing and more tokens. That directly supports scenarios like “concise summary” versus “fine inspection.”

For **captions**, Azure Vision Image Analysis 4.0 is the most direct service to know. Microsoft distinguishes **Caption** from **Dense Captions**: Caption generates **one sentence for the whole image**, while Dense Captions generates **one-sentence descriptions for up to 10 regions** and also returns **bounding boxes** for those regions. That distinction is foundational for exam questions about concise versus detailed description.

Another easy-to-miss detail is that **Image Analysis 4.0 captioning is English-only and region-limited**. Microsoft says Caption and Dense Captions require an Azure Vision resource in supported regions, and it exposes a **`gender-neutral-caption=true`** option to replace gendered labels like “man” or “woman” with “person.” Those are exactly the kinds of implementation specifics exam writers like.

For **alt text**, Microsoft explicitly recommends using Image Analysis captioning as the generation mechanism. The alt-text use-case article says that Image Analysis produces one-sentence descriptions that Microsoft products such as **PowerPoint, Word, and Edge** use as generated alt text. But the exam objective is not just “generate a caption”; it explicitly says **aligned to accessibility guidelines**, so you also need the authoring rules.

The accessibility rules are important: Microsoft’s style guide says alt text should be **accurate, concise, and purpose-driven**, should **not begin with generic words like “Image,”** should include **embedded text** when that text matters, and should generally stay within about **150 characters** unless a more detailed description is needed elsewhere. W3C’s decision tree adds that **decorative images should use empty alt text**, while **complex images** should have the essential information available elsewhere on the page, often through a longer description. W3C also advises avoiding extra filler such as “image” or “picture” in the alt text.

For **question answering grounded in visual evidence**, there are really two patterns to study. The first is **single-image or few-image reasoning** with a vision-enabled chat model. The second is **corpus-scale grounded QA** using **Azure AI Search multimodal search**, where text and images are extracted, verbalized or embedded, stored together, and retrieved through hybrid/vector search. Microsoft explicitly frames multimodal search as the way to answer questions that depend on information buried inside diagrams, screenshots, infographics, or other visuals, while still preserving traceability back to sources.

## Content Understanding analyzers and video workflows

The strongest concept to master for this part of AI-103 is that **Azure Content Understanding** is not just one more OCR API. Microsoft describes it as a Foundry Tool that uses generative AI to process **documents, images, video, and audio** into a **user-defined structured output format**. The central building block is the **analyzer**, which defines the content type, what to extract, how to structure the result, and which AI models to use. Microsoft also distinguishes **base analyzers**, **RAG analyzers**, **domain-specific analyzers**, and **custom analyzers**.

This maps directly to the exam objective **“Implement visual understanding by configuring Azure Content Understanding in Foundry Tools to extract visual characteristics.”** For images, Microsoft says you define a **schema** with fields, descriptions, and output types, and the service returns **structured data** suitable for downstream workflows such as RAG, financial chart understanding, manufacturing defect analysis, or shelf analysis. In other words, the exam wants you to recognize when a problem is not “ask a model about the picture” but instead “extract a stable schema from many images consistently.”

You should also know the difference between **standard** and **pro** modes. Microsoft’s current documentation says **standard mode** handles **documents, images, video, audio, and text**, is optimized for lower cost and latency, and is ideal for **single-file, straightforward extraction**. **Pro mode** is for **multi-step reasoning**, **multiple input files**, and **reference data**, but the critical caveat is that **pro mode is currently only available for document data**. That last point is extremely important because the AI-103 skills list mentions pro-mode pipelines inside the broader computer vision domain; the correct interpretation is that you should understand the product distinction, not assume pro mode currently applies to image/video in the same way it applies to documents.

Microsoft’s older Foundry-classic guidance makes the “single-task” idea even clearer: standard mode is for **processing single files with straightforward field extraction**, while pro mode is for **cross-file analysis**, **knowledge-base/reference data**, and **advanced reasoning**. If an exam scenario sounds like “take one image or one video and extract fields,” think **standard**. If it sounds like “compare multiple files against reference material,” think **pro**, and remember that today this is a **document-focused preview feature**.

For **video analysis workflows**, learn Microsoft’s current prebuilt output model. The Content Understanding video overview says the prebuilt video analyzer produces **RAG-ready output**, including **WEBVTT transcript**, **ordered key-frame thumbnails**, **natural-language segment descriptions**, and **automatic scene segmentation** based on categories you define. Microsoft explicitly says this output can drop directly into a vector store with no post-processing required, which makes it highly relevant for grounding and retrieval scenarios.

The deeper audiovisual reference page adds more implementation detail. Content Understanding can return **transcript phrases**, **timing information**, **key frame times**, **camera shot times**, and a **contents collection** that expands per segment when segmentation is enabled. This is a high-yield study area because it connects several exam verbs at once: process, interpret, segment, ground, and extract. If a question asks which tool gives you structured segment-level output with timing and speech context, Content Understanding is the strong answer.

For **identifying objects, components, or regions within images**, Azure Vision Image Analysis remains important. Microsoft says object detection returns **bounding box coordinates** for each object found, not just tags. For region-aware textual description, Dense Captions also returns **bounding boxes**. For **logos and brands**, Azure Vision offers **brand detection**, which is effectively a specialized object-detection mode backed by a large logo database.

For **video-level object and event insights**, it is also worth remembering **Azure AI Video Indexer**. Microsoft describes Video Indexer as a full video/audio insight solution supporting transcription, translation, object detection, and summarization, and separate documentation highlights scene, shot, and keyframe detection. On the exam, this can be a distractor against Content Understanding: choose **Video Indexer** when the scenario emphasizes packaged deep video indexing across uploaded or live video, and choose **Content Understanding** when the scenario emphasizes **custom schema extraction**, **custom segments**, or **RAG-ready structured output**.

Finally, study **analyzer design best practices**, because they often show up as “why is extraction quality poor?” questions. Microsoft recommends writing **detailed field descriptions**, including **aliases**, using structured **arrays and objects** for repeated data, and explicitly choosing the method **`extract`**, **`generate`**, or **`classify`** per field. Microsoft also notes that **`extract` is only supported for document analyzers**, which again reinforces the importance of knowing product boundaries rather than assuming every mode applies to every modality.

## Responsible AI and multimodal safety

For unsafe or disallowed visual content, the primary guardrail service to study is **Azure AI Content Safety**. Microsoft says the service supports moderation across **text, images, and multimodal content**, and the harm-categories guidance states that the same broad harm flags apply to **both text and images**. In the Foundry “try it out” experience, Microsoft documents configurable image moderation thresholds that return severity levels and an accepted/rejected result based on your policy settings.

For multimodal content specifically, Microsoft provides a **Multimodal API** that analyzes both the **image content and text content** together. The quickstart explains that the API can examine the image itself, the text that appears inside the image, and the associated text supplied with the request. The request example also exposes an **`enableOcr`** flag, which is important because it shows how Azure can incorporate embedded text in an image into the moderation pipeline.

For **indirect prompt injection via embedded text in images**, the current Microsoft product story is slightly nuanced. **Prompt Shields** itself is documented as a detector for adversarial **prompts and documents**, and Microsoft’s FAQ says Prompt Shields works with **text content only**, while multimodal moderation supports **images with text + OCR**. A sound implementation pattern, and the one I would study for the exam, is therefore: **extract or include image text via OCR/multimodal analysis, then run the suspicious text through Prompt Shields or a broader guardrail pipeline before letting it influence a downstream model**. That is partly an inference from multiple Microsoft sources, but it is the most defensible design pattern for the exam objective.

Azure OpenAI also contributes default safety at the model platform layer. Microsoft’s default safety-policy documentation says Azure OpenAI includes built-in protections such as **content filtering models**, **blocklists**, **prompt transformation**, and **content credentials**. The image-generation documentation separately says Azure provides **input and output moderation** across image generation models plus abuse monitoring. On the exam, this means the best answer is often **layered controls**, not a single moderation API call.

For **watermarks and provenance**, the Azure OpenAI concept to know is **Content Credentials**. Microsoft says all AI-generated images from Azure OpenAI include Content Credentials, based on the **C2PA** specification, and that these credentials are represented by a cryptographically signed manifest tracing back to Azure OpenAI. Microsoft also says this is **automatic** for supported Azure OpenAI image-generation outputs. For exam purposes, Content Credentials are the native Azure answer for provenance and transparency.

For **brand usage requirements**, **prohibited symbols**, and **organization-specific policy rules**, combine detection and policy logic. Azure Vision **brand detection** identifies logos and returns names and bounding boxes, while Azure AI Content Safety **custom categories** lets you define and train your own moderation categories that match business-specific policies. That combination is what you want for scenarios such as “reject images containing competitor logos,” “flag a banned symbol,” or “send content with sensitive iconography for human review.”

Azure Vision also remains useful for older-style visual screening such as **adult, racy, or gory content** and for extracting **OCR** or **object detections** that can feed policy engines. Even when Azure AI Content Safety is the preferred moderation product, Vision features are still useful upstream in a richer multimodal safety workflow.

If you see a privacy-sensitive scenario around faces, study the current Content Understanding behavior. Microsoft’s FAQ says face blurring is enabled by default in image and video content, while the image-overview page says face description is a **limited-access** capability enabled through analyzer configuration. That means good responsible-AI answers do not just maximize extraction quality; they also take account of privacy, consent, and data-protection obligations around biometric or face-related data.

## High-yield facts to memorize

**Generate or edit images:** think **Azure OpenAI image generation models**. The current family to know is **`gpt-image-*`**, not DALL-E 3, and you should remember text-plus-image input, mask-based editing, quality controls, and size controls.

**Generate or edit videos:** think **Sora 2 in Azure OpenAI**. Memorize text → video, image → video, video → video, asynchronous job flow, supported resolutions, remix, and reference-media inputs.

**Open-ended reasoning over an image:** think **vision-enabled chat models**. Use a multimodal chat completion request and tune `detail` for lower-cost or higher-detail image understanding.

**Concise versus detailed captions:** think **Caption** versus **Dense Captions** in **Azure Vision Image Analysis 4.0**. Caption describes the whole image; Dense Captions describes multiple regions and returns region boxes.

**Alt text and accessibility:** use AI-generated captions as a starting point, but align them to accessibility rules: concise, purpose-first, no “image,” include meaningful embedded text, empty alt for decorative images, and longer descriptions for complex images.

**Schema-based extraction from images or video:** think **Azure Content Understanding analyzers**. If the requirement is consistent structured JSON or markdown output, especially for downstream automation or RAG, this is stronger than a free-form vision chat answer.

**Standard versus Pro in Content Understanding:** standard is the default for multimodal single-file extraction across images, video, audio, and documents; pro is for advanced, multi-step, cross-file reasoning and is **currently document-only**.

**Grounded visual QA across a repository:** think **Azure AI Search multimodal search**. It extracts text and images, verbalizes or embeds them, stores them together, and supports grounded retrieval for RAG.

**Unsafe visual content:** think **Azure AI Content Safety** with harm-category thresholds, plus multimodal analysis when images contain text.

**Indirect prompt injection in image text:** the best study answer is a layered pipeline using **OCR or multimodal OCR-enabled analysis**, then **Prompt Shields** and other guardrails before handing content to the downstream model. That is the most exam-ready interpretation of Microsoft’s current product boundaries.

**Brand and symbol policies:** think **Vision brand detection** plus **Content Safety custom categories** plus app-level business rules. Azure gives you the detection signals; your workflow enforces the organizational policy.

**Provenance and watermark-style transparency:** the Azure-native concept to remember is **Content Credentials**, which Azure OpenAI automatically applies to supported generated images and signs using the C2PA standard.

This is the highest-yield summary of the objective domain: know **which service to choose**, know the **important control parameters**, know where **preview limitations** still matter, and know how to combine **generation, grounding, extraction, and safety** into one coherent Azure architecture.

## Further reading

Microsoft Learn pages for this domain (verified June 2026):

- [What is Image Analysis? (Azure Vision)](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/overview-image-analysis)
- [What is Azure Content Understanding in Foundry Tools?](https://learn.microsoft.com/en-us/azure/ai-services/content-understanding/overview)
- [Microsoft Foundry Models overview](https://learn.microsoft.com/en-us/azure/foundry/concepts/foundry-models-overview)
- [What is Microsoft Foundry?](https://learn.microsoft.com/en-us/azure/foundry/what-is-foundry)
- [Azure AI Search documentation](https://learn.microsoft.com/en-us/azure/search/)
- [What is Azure AI Content Safety?](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/overview)
- [Exam AI-103 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-103)
