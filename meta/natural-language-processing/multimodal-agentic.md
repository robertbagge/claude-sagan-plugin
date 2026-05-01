# Multimodal & Agentic NLP

The LLM era did not stay text-only for long. By 2021, the same transformer machinery that had digested the web's text was being trained against captioned images; by 2023, the same models were being taught to call APIs, retrieve documents, and chain steps. This dossier walks through the two arcs — multimodal grounding and agentic tool use — that turned NLP from an autocomplete engine into a general interface for software.

## 1. The bridge: CLIP and contrastive image-text pretraining

Before CLIP, computer vision and NLP were largely separate stacks. ImageNet-style supervised classifiers handled images; transformers handled text. The bridge came from a startlingly simple objective: given a batch of (image, caption) pairs scraped from the web, train two encoders so that the correct image-text pairs have higher cosine similarity than mismatched ones [T1] Radford et al. 2021, "Learning Transferable Visual Models From Natural Language Supervision," arXiv:2103.00020, abstract.

Three numbers explain the impact. The training set was 400 million (image, text) pairs collected from the public internet (the WIT dataset) [T1] Radford et al. 2021, p. 1, ¶3. The batch size was 32,768 — every step pulled tens of thousands of negatives into a single contrastive matrix, with InfoNCE-style cross-entropy along rows and columns [T2] open_clip docs, https://github.com/mlfoundations/open_clip; [T2] Hugging Face, openai/clip-vit-large-patch14 model card. And the headline result: the largest variant, ViT-L/14 at 336 pixels, hit roughly 75.3–76.2% zero-shot top-1 on ImageNet, matching the original ResNet-50 without seeing a single ImageNet training example [T1] Radford et al. 2021, abstract; [T2] open_clip PRETRAINED.md.

What made CLIP load-bearing for NLP, as opposed to "yet another vision paper," was the text encoder. Once trained, the language tower projected arbitrary natural-language strings into the same embedding space as images. That meant any image task could be reformulated as a text-classification problem ("a photo of a {dog, cat, ...}"), and any text query could retrieve images by nearest-neighbour search. The same encoder later became the conditioning backbone for DALL·E 2 (which is literally called "unCLIP" because it inverts CLIP image embeddings back into pixels via a diffusion decoder) and Stable Diffusion (which uses OpenCLIP-ViT/H as its frozen text encoder) [T2] HuggingFace diffusers docs, stable_unclip pipeline; [T3] Sapunov, "OpenAI and the road to text-guided image generation," Medium, https://moocaholic.medium.com/openai-and-the-road-to-text-guided-image-generation-dall-e-clip-glide-dall-e-2-unclip-c6e28f7194ea. CLIP was the first model where "language" became a general control surface for the visual world.

## 2. Visual language models: from Flamingo to LLaVA

CLIP could match images and text but couldn't *converse* about an image. The next wave was about wiring vision encoders into autoregressive language models so the LLM could read a picture as if it were a paragraph.

DeepMind's Flamingo (April 2022) was the architectural template. It took a frozen vision encoder and a frozen 70B-parameter Chinchilla language model, then inserted trainable "Perceiver Resampler" and gated cross-attention layers between them so visual features could flow into the LLM without retraining either tower from scratch [T1] Alayrac et al. 2022, "Flamingo: a Visual Language Model for Few-Shot Learning," arXiv:2204.14198, abstract. Crucially, Flamingo accepted *interleaved* sequences of images and text — meaning few-shot prompting (the same trick that made GPT-3 famous) now worked across modalities. Show it two captioned examples, then a third image, and it would caption it.

Salesforce's BLIP-2 (January 2023) made the recipe cheap. Instead of bespoke cross-attention, it introduced the Q-Former: a lightweight transformer with a fixed set of learnable query vectors that distil a frozen image encoder's output into a small bag of tokens, which are then fed to a frozen LLM [T1] Li et al. 2023, "BLIP-2: Bootstrapping Language-Image Pre-training with Frozen Image Encoders and Large Language Models," arXiv:2301.12597, abstract. Two stages of pretraining; almost everything stays frozen; you can swap the LLM at will.

LLaVA (April 2023) made it a weekend project. Liu et al. used GPT-4 (text-only) to synthesise a multimodal instruction-following dataset from image captions and bounding boxes, then fine-tuned a small projection layer mapping CLIP visual features into a Vicuna LLM's embedding space [T1] Liu et al. 2023, "Visual Instruction Tuning," arXiv:2304.08485, abstract. The model scored 85.1% relative to GPT-4 on a synthetic multimodal benchmark and was released open-source, kicking off a Cambrian explosion of LLaVA forks [T1] Liu et al. 2023, abstract.

OpenAI's GPT-4V was the productised version of this idea. The system card, dated September 25, 2023, notes that vision training was completed in 2022 and that early access began in March 2023 — meaning OpenAI sat on the capability for roughly a year doing safety work before release [T2] OpenAI, "GPT-4V(ision) System Card," September 25, 2023, https://cdn.openai.com/papers/GPTV_System_Card.pdf. GPT-4o (May 13, 2024) collapsed the pipeline further: a single end-to-end model trained jointly on text, vision, and audio rather than three models stitched together, with audio I/O latency low enough for natural conversation [T2] OpenAI, "Hello GPT-4o," May 13, 2024, https://openai.com/index/hello-gpt-4o/.

## 3. Audio joins the chat: Whisper

The audio side of multimodal NLP came from a paper with an unfashionable thesis: if you train a vanilla encoder-decoder transformer on enough weakly-labelled audio-text data, you don't need clever architectures. OpenAI's Whisper (December 2022) trained on 680,000 hours of multilingual, multitask audio scraped from the web, with the decoder predicting not just transcripts but also language ID, voice activity, and translation as special tokens [T1] Radford et al. 2022, "Robust Speech Recognition via Large-Scale Weak Supervision," arXiv:2212.04356, abstract.

The result generalised zero-shot to standard ASR benchmarks — no fine-tuning per dataset — and approached human accuracy and robustness on noisy or accented speech [T1] Radford et al. 2022, abstract. Whisper matters for the NLP arc not because of the acoustic modelling but because it made *speech a solved upstream problem*. Once Whisper exists and runs cheaply, every voice-input feature in every LLM product — ChatGPT voice, Claude on mobile, the dictation pipeline behind GPT-4o's audio mode — is downstream of that one model. Speech-to-text is no longer a research bottleneck; it is glue code.

## 4. Retrieval-augmented generation: external memory for LLMs

Parallel to the multimodal arc, NLP researchers were attacking a different limitation: language models hallucinate facts they don't know and can't cite sources. Lewis et al. (May 2020, NeurIPS) proposed RAG — retrieval-augmented generation — combining a parametric BART seq2seq model with a non-parametric index of Wikipedia accessed by a Dense Passage Retriever [T1] Lewis et al. 2020, "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks," arXiv:2005.11401, abstract.

The architecture has two flavours. RAG-Sequence retrieves a set of passages once and conditions the entire generated answer on them. RAG-Token can switch passages on a token-by-token basis [T1] Lewis et al. 2020, abstract. On open-domain QA (Natural Questions, TriviaQA, WebQuestions), RAG beat both purely parametric seq2seq models and the older retrieve-and-extract pipelines, and on free-form generation it produced "more specific, diverse and factual" output [T1] Lewis et al. 2020, abstract.

RAG's practical legacy is enormous. Every "chat with your docs" product, every enterprise search-over-PDFs SaaS, every Cursor or Claude Code feature that pulls in a few relevant files before generating a patch — these are all variants of Lewis's recipe, usually with the BART decoder swapped for a frontier LLM and the DPR retriever swapped for a vector DB (FAISS, pgvector, Pinecone). Retrieval gave LLMs a way to cite, refresh, and scope their knowledge without retraining.

## 5. The agentic turn: ReAct and Toolformer

By 2022, two threads were converging. LLMs were clearly capable of *reasoning* in natural language (chain-of-thought, Wei et al. 2022). They were also clearly bad at *acting* in environments — they hallucinated facts, couldn't do arithmetic, and couldn't check their work. The agentic line of research asked: what if the LLM could call out to tools mid-generation?

ReAct (Yao et al., October 2022) is the cleanest formulation. The model is prompted to interleave three kinds of tokens: **Thought** (free-form reasoning), **Action** (a structured call to an external tool, e.g. `Search[Apollo program]`), and **Observation** (the tool's response, fed back into the context) [T1] Yao et al. 2022, "ReAct: Synergizing Reasoning and Acting in Language Models," arXiv:2210.03629, abstract. The reasoning trace lets the model plan; the actions let it ground claims in real data; observations feed back into the next thought. On HotpotQA and FEVER, ReAct interacting with a Wikipedia API beat chain-of-thought-only baselines on hallucination. On the interactive benchmarks ALFWorld and WebShop, it beat imitation-learning and RL baselines by 34 and 10 absolute success-rate points respectively, with only one or two in-context examples [T1] Yao et al. 2022, abstract.

ReAct's importance is that it is the architectural pattern almost every modern agent framework implements. LangChain agents, OpenAI's tool-use loop, Claude's `tool_use` / `tool_result` blocks, and the inner loop of Claude Code itself all run a thought-action-observation cycle, with the surface details (XML tags, JSON, function-call schemas) varying by vendor.

Toolformer (Schick et al., February 2023) approached the same problem from training rather than prompting. The pipeline: take a language model (the paper used GPT-J 6.7B), have it propose API calls inserted into ordinary text on a self-supervised basis, then *keep only the calls whose results reduce the perplexity of the subsequent tokens* [T1] Schick et al. 2023, "Toolformer: Language Models Can Teach Themselves to Use Tools," arXiv:2302.04761, abstract. Fine-tune on the surviving examples and you get a model that calls calculators, search, translation, QA, and a calendar API at the right moments. Tool-augmented Toolformer matched or beat GPT-3 175B on several downstream tasks despite being ~25× smaller [T1] Schick et al. 2023, abstract.

These two papers together established the design space. ReAct says "tool use is a prompting and decoding pattern." Toolformer says "tool use can be baked into weights via self-supervision." Production systems today blend both: the model is post-trained to emit tool calls in a specific format (Toolformer-style), and the orchestrator runs a thought-action-observation loop around it (ReAct-style).

## 6. Frameworks and platforms: from LangChain to MCP

Once the research patterns were clear, the tooling layer arrived fast.

**LangChain** released its first version on October 24, 2022 — about three weeks before ChatGPT — as an 800-line Python wrapper that let you compose prompts, LLM calls, and a few tools into "chains" [T3] Harrison Chase on X, https://x.com/hwchase17/status/1981791807160955231; [T2] Wikipedia, "LangChain," https://en.wikipedia.org/wiki/LangChain. ChatGPT's launch in November 2022 brought a wave of developers; LangChain became the default scaffolding for the "LLM + tools + memory" pattern.

**AutoGPT** (March 30, 2023) was the meme that made "AI agents" a household term. Toran Bruce Richards wired GPT-4 into a self-prompting loop that decomposed a high-level goal into subtasks, executed them, and re-planned [T2] Wikipedia, "AutoGPT," https://en.wikipedia.org/wiki/AutoGPT. It became the fastest-trending GitHub repository in history. It was also famously unreliable — costs spiralled, loops got stuck — but it set the public expectation that agents were imminent.

**OpenAI's function calling** (June 13, 2023) made tool use a first-class API feature. Developers could pass a JSON schema of available functions; the model would emit a structured JSON object with the chosen function and arguments rather than free-form text [T2] OpenAI, "Function calling and other API updates," June 13, 2023, https://openai.com/index/function-calling-and-other-api-updates/. This eliminated brittle regex-based parsing and was the single change that made production agent systems plausible.

**OpenAI Assistants API** (DevDay, November 6, 2023) bundled function calling with built-in code interpreter, retrieval (RAG), persistent threads, and stateful tool runs — explicitly framed as "agent-like experiences" [T2] OpenAI, "New models and developer products announced at DevDay," November 6, 2023, https://openai.com/index/new-models-and-developer-products-announced-at-devday/.

**Anthropic's computer use** (October 2024, with the upgraded Claude 3.5 Sonnet) generalised tool use to "the entire computer." Instead of bespoke APIs, Claude was trained to look at screenshots, move a cursor, click, and type — operating GUIs the way humans do [T2] Anthropic, "Introducing computer use, a new Claude 3.5 Sonnet, and Claude 3.5 Haiku," October 22, 2024, https://www.anthropic.com/news/3-5-models-and-computer-use. On the OSWorld benchmark for computer-using agents, it scored 14.9% in screenshot-only mode versus 7.8% for the next-best system [T2] Anthropic, October 2024 announcement.

**Model Context Protocol (MCP)** (November 26, 2024, Anthropic) tackled the "N×M" integration problem: every agent had to write custom connectors for every data source. MCP defines a JSON-RPC 2.0 protocol — message-flow inspired by the Language Server Protocol — for clients (LLMs) and servers (tools/data) to talk over a standard wire format [T2] Wikipedia, "Model Context Protocol," https://en.wikipedia.org/wiki/Model_Context_Protocol; [T2] Anthropic, "Introducing the Model Context Protocol," November 26, 2024, https://www.anthropic.com/news/model-context-protocol. SDKs shipped in Python, TypeScript, C#, Java, alongside reference servers for Slack, GitHub, Postgres, Drive, and Puppeteer. By 2025 it was a de facto industry standard; OpenAI, Google, and others adopted it.

**Claude Code** (research preview February 2025; GA May 2025) is the canonical example of a production agent. It is a CLI that runs the ReAct loop (Claude as planner, file/bash/edit/web tools as the action set), uses MCP for extensibility, and treats the developer's terminal as the environment [T2] Anthropic, Claude Code overview, https://code.claude.com/docs/en/overview; [T2] GitHub, anthropics/claude-code, https://github.com/anthropics/claude-code. Anthropic reported it reached roughly $1B in annualised revenue by November 2025 [T3] DataStudios, "Claude Code Explained," https://www.datastudios.org/post/claude-code-explained-how-anthropic-s-terminal-first-coding-agent-works-across-cli-sessions-ide-in.

## 7. Where this leaves things

Three observations on the current state, framed for someone trying to read the next two years:

**Agents are the interface, not the model.** The frontier labs ship LLMs; the agent layer (tool schemas, MCP servers, planning loops, memory, sandboxes) is where most user-visible product differentiation happens. Claude Code, Cursor, Devin, and ChatGPT's agent mode are all roughly the same architecture — the differences are tool curation, harness quality, and post-training for the specific tool set.

**Multimodality is converging on "any-to-any" single models.** GPT-4o trained text/vision/audio jointly end-to-end rather than stitching specialist models [T2] OpenAI, "Hello GPT-4o." The trend continues: video, action tokens (for robotics), and 3D inputs are likely to fold into the same backbone rather than spawn separate research tracks. The CLIP-style "two encoders into a shared space" pattern is being absorbed into "one big transformer that reads everything."

**RAG and tool use blur into the same thing.** From the model's point of view, "search Wikipedia," "query a vector index," and "call a function" are all the same emit-structured-call-and-condition-on-result loop. Production stacks increasingly drop the distinction; the model decides whether grounding evidence comes from a vector store, a SQL query, or a live API call.

The honest open questions are the unglamorous ones. Long-horizon reliability (can an agent run for hours without going off the rails?), evaluation (how do you benchmark a system that takes 200 actions across 12 tools?), cost (every reasoning step is tokens), and safety (computer use means the model can wire money or delete production data). The 2020-2024 arc was about *capability*. The 2025-onwards arc is mostly about getting that capability to work in environments where mistakes are expensive.

## Sources

- [T1] Radford, A., Kim, J. W., Hallacy, C., et al. (2021). "Learning Transferable Visual Models From Natural Language Supervision." arXiv:2103.00020. February 26, 2021. https://arxiv.org/abs/2103.00020
- [T1] Alayrac, J.-B., Donahue, J., Luc, P., et al. (2022). "Flamingo: a Visual Language Model for Few-Shot Learning." arXiv:2204.14198. April 29, 2022 (DeepMind). https://arxiv.org/abs/2204.14198
- [T1] Li, J., Li, D., Savarese, S., Hoi, S. (2023). "BLIP-2: Bootstrapping Language-Image Pre-training with Frozen Image Encoders and Large Language Models." arXiv:2301.12597. January 30, 2023 (Salesforce Research). https://arxiv.org/abs/2301.12597
- [T1] Liu, H., Li, C., Wu, Q., Lee, Y. J. (2023). "Visual Instruction Tuning" (LLaVA). arXiv:2304.08485. April 17, 2023; NeurIPS 2023 Oral. https://arxiv.org/abs/2304.08485
- [T1] Radford, A., Kim, J. W., Xu, T., et al. (2022). "Robust Speech Recognition via Large-Scale Weak Supervision" (Whisper). arXiv:2212.04356. December 6, 2022 (OpenAI). https://arxiv.org/abs/2212.04356
- [T1] Lewis, P., Perez, E., Piktus, A., et al. (2020). "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks." arXiv:2005.11401. May 22, 2020; NeurIPS 2020 (Facebook AI / UCL / NYU). https://arxiv.org/abs/2005.11401
- [T1] Yao, S., Zhao, J., Yu, D., et al. (2022). "ReAct: Synergizing Reasoning and Acting in Language Models." arXiv:2210.03629. October 6, 2022; ICLR 2023. https://arxiv.org/abs/2210.03629
- [T1] Schick, T., Dwivedi-Yu, J., Dessì, R., et al. (2023). "Toolformer: Language Models Can Teach Themselves to Use Tools." arXiv:2302.04761. February 9, 2023 (Meta AI). https://arxiv.org/abs/2302.04761
- [T2] OpenAI. "GPT-4V(ision) System Card." September 25, 2023. https://cdn.openai.com/papers/GPTV_System_Card.pdf
- [T2] OpenAI. "Hello GPT-4o." May 13, 2024. https://openai.com/index/hello-gpt-4o/
- [T2] OpenAI. "Function calling and other API updates." June 13, 2023. https://openai.com/index/function-calling-and-other-api-updates/
- [T2] OpenAI. "New models and developer products announced at DevDay" (Assistants API). November 6, 2023. https://openai.com/index/new-models-and-developer-products-announced-at-devday/
- [T2] Anthropic. "Introducing computer use, a new Claude 3.5 Sonnet, and Claude 3.5 Haiku." October 22, 2024. https://www.anthropic.com/news/3-5-models-and-computer-use
- [T2] Anthropic. "Introducing the Model Context Protocol." November 26, 2024. https://www.anthropic.com/news/model-context-protocol
- [T2] Anthropic. "Claude Code overview." Anthropic Docs (accessed 2026). https://code.claude.com/docs/en/overview
- [T2] Anthropic. anthropics/claude-code GitHub repository. https://github.com/anthropics/claude-code
- [T2] Wikipedia. "Model Context Protocol." (accessed 2026). https://en.wikipedia.org/wiki/Model_Context_Protocol
- [T2] Wikipedia. "LangChain." (accessed 2026). https://en.wikipedia.org/wiki/LangChain
- [T2] Wikipedia. "AutoGPT." (accessed 2026). https://en.wikipedia.org/wiki/AutoGPT
- [T2] Hugging Face. "openai/clip-vit-large-patch14" model card. https://huggingface.co/openai/clip-vit-large-patch14
- [T2] mlfoundations. open_clip PRETRAINED.md. https://github.com/mlfoundations/open_clip/blob/main/docs/PRETRAINED.md
- [T2] Hugging Face diffusers documentation. "Stable unCLIP" pipeline. https://huggingface.co/docs/diffusers/api/pipelines/stable_unclip
- [T3] Chase, H. (Harrison Chase, LangChain founder). X post on LangChain's third birthday. October 24, 2025. https://x.com/hwchase17/status/1981791807160955231
- [T3] Sapunov, G. "OpenAI and the road to text-guided image generation: DALL·E, CLIP, GLIDE, DALL·E 2 (unCLIP)." Medium. https://moocaholic.medium.com/openai-and-the-road-to-text-guided-image-generation-dall-e-clip-glide-dall-e-2-unclip-c6e28f7194ea
- [T3] DataStudios. "Claude Code Explained: How Anthropic's Terminal-First Coding Agent Works." https://www.datastudios.org/post/claude-code-explained-how-anthropic-s-terminal-first-coding-agent-works-across-cli-sessions-ide-in
