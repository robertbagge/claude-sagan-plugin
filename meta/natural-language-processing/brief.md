# Natural Language Processing — Research Brief

**Domain**: `natural-language-processing`
**Output dir**: `meta/research/natural-language-processing/`
**Created**: 2026-05-01

## Goal

Create a dossier for someone interested in AI/NLP to understand how it started, important discoveries, popularisation, current state and where it might be going.

## Inspirations & references

- Jurafsky & Martin, *Speech and Language Processing* (canonical NLP textbook)
- Stanford CS224N / CS224U course materials and lecture videos
- Karpathy and 3Blue1Brown explainer content (transformers, language models, neural nets)
- The Gradient, Distill.pub, Papers with Code (long-form explainers and benchmark trackers)
- Foundational ML/NN/DL papers in the spirit the user named (e.g. AlexNet, YOLO, ResNet, AlphaGo) — referenced only where they directly shaped an NLP technique (see Round 1 Topic 4 prompt)

## Constraints & scope

- Skip non-English language traditions; stay focused on the English-language NLP arc.
- No speculation on closed details of GPT-4/5, Gemini, Claude internals beyond what's publicly documented.
- Skip pre-1950 linguistic theory (Saussure, structuralism, etc.); start the historical arc at the computational era (Shannon, Turing onwards).
- Pure speech (ASR/TTS/phonetics) is **not excluded** — include where it intersects with text NLP, but it isn't a focus.

## Source types

For this project, source preference is: **50% Type 1 (peer-reviewed papers), 40% Type 2 (textbooks, official docs, landmark whitepapers), 10% Type 3 (blogs, X, podcasts, conference talks)**.

The three types are defined below regardless of priority. When citing a source, use the format and tag for its type. No quotas — cover whichever types the topic actually justifies, weighted by the priority above.

- **Type 1** — published research papers, peer-reviewed work, conference proceedings. Cite as: source + page + paragraph. Example: `[T1] Vaswani et al. 2017, p. 4, ¶2`.
- **Type 2** — official documentation, technical specs, design system docs, white papers, books, renowned domain blogs. Cite as: source + subsection (if any) + line number. Example: `[T2] Jurafsky & Martin 3rd ed., ch. 9, p. 187`.
- **Type 3** — Twitter/X, Hacker News, Reddit, small blogs, podcasts, conference videos. Cite as: source + URL. Example: `[T3] @karpathy on X, https://...`.

Tag every inline citation with its type (`[T1]`, `[T2]`, `[T3]`) so the synthesis can spot coverage gaps relative to the stated preference.

## Conventions

- Each topic is researched by a parallel agent and saved to `meta/research/natural-language-processing/{topic-slug}.md`.
- Each topic agent's primary mode is external web research — fresh sources, not training data.
- Synthesis lives at `meta/research/natural-language-processing/synthesis.md` and is fully rewritten after each round.
- Rounds are appended below as `## Round 2`, `## Round 3`, etc. Earlier rounds and their topic files are never rewritten.

### Audience mode (applies across all topics)

Hybrid. Technical depth — math sketched, architectures named, key equations quoted — in the **science** topics: foundational ML/DL discoveries, word embeddings, tokenization, sequence models, transformer revolution, scaling laws, RLHF/alignment. Accessible narrative — intuition, history, named landmarks, no equations — in the **history and popularisation** topics: origins, symbolic era, popularisation, frontier directions. Statistical NLP, multimodal/agentic, evaluation, and open weights sit in between — pitch them at intelligent generalist with technical detail introduced when load-bearing.

## Round 1

### Origins (1950s–60s)

File: `origins.md`

Trace the computational-era foundations of NLP from Turing's 1950 *Computing Machinery and Intelligence*, Shannon's information theory and his 1951 work on the entropy of English, the Georgetown–IBM machine translation experiment (1954), the optimism of early MT, through to the 1966 ALPAC report and the resulting funding collapse / first AI winter for language work. Name the key figures (Turing, Shannon, Weaver, Bar-Hillel, Chomsky's early influence on the field), the foundational papers and reports, and frame why this era's promises outran its methods. Accessible narrative; minimal math.

### Symbolic / rule-based era (1960s–80s)

File: `symbolic-era.md`

Cover the symbolic AI / rule-based paradigm in NLP: ELIZA (Weizenbaum 1966), SHRDLU (Winograd 1972), conceptual dependency (Schank), augmented transition networks, formal grammars (CFG, LFG, HPSG), hand-built parsers, expert systems, and early dialogue/QA systems. Explain the underlying assumptions (linguistic competence as rules, language as logic), why this paradigm hit its ceiling (brittleness, scaling, lexical ambiguity, no common sense), and how the field's frustration with symbolic methods set up the statistical turn. Accessible narrative.

### Statistical NLP revolution (1990s–2000s)

File: `statistical-nlp.md`

Document the empirical / statistical turn in NLP: hidden Markov models for tagging and speech, the IBM Models for statistical machine translation (Brown et al. 1993), n-gram language models, smoothing techniques (Kneser–Ney, Good–Turing), the Penn Treebank as a shared resource (Marcus et al. 1993), maximum entropy models, conditional random fields (Lafferty et al. 2001), and the rise of supervised learning on corpora. Capture the cultural shift ("every time I fire a linguist, performance goes up" — Jelinek). Pitch: intelligent generalist, with formulas where load-bearing.

### Foundational ML/DL discoveries that powered NLP

File: `foundational-ml-discoveries.md`

Cover the ML/DL discoveries that NLP inherited and which shaped its modern toolkit, but **strictly limited to those with direct NLP impact**. In scope: perceptron (Rosenblatt 1958), backpropagation (Rumelhart, Hinton, Williams 1986), SGD and modern optimisers (Adam 2014), regularisation (dropout 2014), normalisation (LayerNorm 2016, central to Transformers), residual connections (ResNet 2015 → Transformer residual streams), self-supervised pretraining as an idea. Out of scope: cross-domain papers like AlexNet, YOLO, AlphaGo unless their specific contribution flowed into a named NLP technique. Technical depth; equations and architectural diagrams expected.

### Word embeddings & distributional semantics

File: `word-embeddings.md`

Trace distributional semantics from Firth's "you shall know a word by the company it keeps" through latent semantic analysis (LSA), to dense static word embeddings: word2vec (Mikolov et al. 2013, both skip-gram and CBOW), GloVe (Pennington et al. 2014), fastText (Bojanowski et al. 2017). Explain the math of negative sampling, the analogy results (king − man + woman ≈ queen) and their limitations, the move to contextual embeddings (ELMo, Peters et al. 2018) as the bridge to the Transformer era. Technical depth.

### Sequence models

File: `sequence-models.md`

Cover neural sequence modelling: vanilla RNNs and the vanishing gradient problem, LSTMs (Hochreiter & Schmidhuber 1997), GRUs (Cho et al. 2014), seq2seq with encoder-decoder (Sutskever, Vinyals, Le 2014), and the original additive attention mechanism (Bahdanau et al. 2014; Luong et al. 2015). Show why these architectures dominated NMT, summarisation, and dialogue from ~2014–2017, and why their sequential nature (no parallel training over time) primed the field for the Transformer. Technical depth; sketch the gating equations and the attention formulation.

### Tokenization & subword units

File: `tokenization.md`

Cover the evolution of tokenization in NLP: word-level tokenization and its OOV problem, character-level models, the move to subwords with Byte Pair Encoding (Sennrich, Haddow, Birch 2016 — originally for NMT), WordPiece (used in BERT), SentencePiece (Kudo & Richardson 2018), Unigram LM tokenization (Kudo 2018), byte-level BPE (GPT-2). Cover the recent re-examination: tokenizer pathologies (glitch tokens, "SolidGoldMagikarp"), tokenizer-free / byte-level models (ByT5, MambaByte), and the Megabyte / patch-based directions. Technical depth; examples of how the same string tokenises across schemes.

### The Transformer revolution

File: `transformer-revolution.md`

Anchor on *Attention is All You Need* (Vaswani et al. 2017). Cover the architecture (multi-head self-attention, positional encodings, feedforward blocks, residual + LayerNorm), the lineage that followed: BERT (Devlin et al. 2018) and the encoder-only / masked LM line, GPT-1/2/3 (Radford 2018, 2019; Brown et al. 2020) and the decoder-only / autoregressive line, T5 (Raffel et al. 2020) and encoder-decoder text-to-text. Explain the architectural why (parallelism, long-range dependencies, scalability) and the paradigm shift to pretraining + fine-tuning + (later) prompting. Technical depth.

### Scaling laws & emergent abilities

File: `scaling-laws.md`

Cover the empirical scaling story: Kaplan et al. 2020 (*Scaling Laws for Neural Language Models*), Hoffmann et al. 2022 (*Chinchilla* — compute-optimal scaling), the Chinchilla scaling laws and what they implied for "data-starved" earlier models. Then the emergent abilities debate: Wei et al. 2022 (*Emergent Abilities of Large Language Models*) vs. Schaeffer et al. 2023 (*Are Emergent Abilities a Mirage?*), and the metric-discontinuity argument. Touch on inverse scaling and BIG-bench. Technical depth; show the loss-vs-compute power-law form.

### RLHF, instruction tuning, and alignment

File: `rlhf-alignment.md`

Cover the post-pretraining alignment stack: instruction tuning (FLAN, T0), RLHF lineage from Christiano et al. 2017 (*Deep RL from Human Preferences*) → InstructGPT (Ouyang et al. 2022) → ChatGPT inflection (Nov 2022), the PPO-based RLHF recipe, Anthropic's Constitutional AI (Bai et al. 2022), Direct Preference Optimization (Rafailov et al. 2023) and successors (IPO, KTO, ORPO). Cover what alignment means in this context, the role of the reward model, reward hacking, and why DPO mattered. Technical depth.

### Multimodal & agentic NLP

File: `multimodal-agentic.md`

Cover the move beyond text-only: CLIP (Radford et al. 2021) as the modern bridge between text and images, vision-language models (Flamingo, BLIP-2, LLaVA, GPT-4V/4o), audio + text (Whisper). Then the agentic line: tool use (ReAct — Yao et al. 2022, Toolformer — Schick et al. 2023), retrieval-augmented generation (Lewis et al. 2020), and the agent framework wave (LangChain, AutoGPT, OpenAI Assistants API, Claude Code, function calling). Frame agents as the current frontier interface for LLMs. Pitch at intelligent generalist with technical detail where the architecture matters.

### Evaluation in the LLM era

File: `evaluation.md`

Cover how NLP evaluation broke and is being rebuilt. Pre-LLM era: GLUE (Wang et al. 2018), SuperGLUE (Wang et al. 2019), and benchmark saturation. LLM era: MMLU (Hendrycks et al. 2020), BIG-bench (Srivastava et al. 2022), HELM (Liang et al. 2022), HumanEval (Chen et al. 2021), and the rapid saturation problem. The data contamination crisis (training data leakage into benchmarks). Newer directions: capability evals, dangerous-capability evals (METR, Apollo), Chatbot Arena and human preference evals, agent benchmarks (SWE-bench, GAIA). Pitch at intelligent generalist; technical where benchmark mechanics matter.

### Open weights & the OSS ecosystem

File: `open-weights-ecosystem.md`

Cover the open-weights movement from BLOOM (BigScience 2022) → LLaMA / LLaMA 2 / 3 (Meta 2023–2024) → Mistral and Mixtral (2023–2024) → Qwen, DeepSeek, Phi, Gemma. The role of Hugging Face as infrastructure (Hub, Transformers library, Datasets, Spaces). The closed-vs-open debate: capability gap, safety arguments, regulatory positioning (EU AI Act, US executive orders), the licensing question (LLaMA's "community license" vs. Apache 2.0). Pitch at intelligent generalist; technical when architecture/training details differentiate models.

### Popularisation & cultural impact

File: `popularisation.md`

Trace how NLP/AI entered public consciousness: ELIZA's reception in the 1960s and Weizenbaum's discomfort, Watson on Jeopardy! (2011), Siri / Alexa / Google Assistant launching the assistant era, Tay (2016) and chatbot failures, Microsoft Tay vs. Xiaoice cultural divergence, AlphaGo / AlphaZero as adjacent moments. Then the ChatGPT inflection (Nov 2022): adoption curve, "GPT moment" framing, the *New York Times* / Bing / Sydney episode, public discourse shift. Cover hype cycles, doomer-vs-accelerationist polarisation, regulatory discourse (Bletchley, EU AI Act, SB-1047). Accessible narrative; cite Type 3 sources liberally for the cultural pulse.

### Frontier directions & possible futures

File: `frontier-directions.md`

Layered horizons, weighted **45% near-term, 45% medium-term, 10% long-term**.

- **Near-term (1–2 years, 45%)**: reasoning models (o1, o3, DeepSeek-R1, Claude with extended thinking) and the test-time compute paradigm; agents and computer use; on-device / small LLMs (Phi, Gemma, Apple Intelligence); current frontier debates around data walls, synthetic data, and post-training.
- **Medium-term (5 years, 45%)**: architectural shifts beyond vanilla Transformers (Mamba / state-space models, mixture-of-experts at scale, hybrid architectures); efficiency curves (algorithmic improvement vs. compute scaling); alignment maturity (interpretability progress, scalable oversight); post-LLM ideas (world models à la LeCun's JEPA, neurosymbolic revival).
- **Long-term (10–20 years, 10%)**: AGI debate (where the disagreement actually lies), civilisational implications, intelligence as a commodity arguments, and serious post-LLM bets.

Pitch as accessible narrative with technical specificity where it grounds the speculation. Distinguish *consensus*, *contested*, and *fringe* claims explicitly.

## Round 2

### Mechanistic interpretability

File: `mechanistic-interpretability.md`

Cover the rise of mechanistic interpretability as a research discipline: Olah/Distill's circuits work transferred from vision to language, Anthropic's transformer-circuits framework (Elhage et al. 2021, induction heads, the residual stream as communication channel), polysemanticity and superposition (Elhage et al. 2022), sparse autoencoders for feature decomposition (Cunningham et al. 2023, Anthropic SAE work 2023–2024), circuit tracing and attribution graphs (2024–2025), and activation-patching/influence-function toolkits. Pitch at intelligent generalist with technical depth; quote the residual-stream decomposition and SAE training objective where load-bearing.

### Long-context engineering and memory

File: `long-context-memory.md`

Trace context-length engineering: original 512-token windows through RoPE (Su et al. 2021), ALiBi (Press et al. 2022), NTK-aware/YaRN scaling, FlashAttention 1/2/3 (Dao et al. 2022+), ring attention and sequence parallelism, sliding-window and attention-sink tricks, KV-cache compression, to the production reality of 200K (Claude) and 1M+ (Gemini) windows. Distinguish long-context architectures from RAG as memory; cover "lost in the middle" (Liu et al. 2023) and long-context evaluation suites (RULER, NIAH, BABILong). Technical depth; equations for RoPE rotation and attention IO-cost reformulation.

### Pretraining data: curation, copyright, contamination

File: `pretraining-data.md`

Cover training data as a domain in its own right: canonical web corpora (Common Crawl, C4, The Pile, RefinedWeb, RedPajama, Dolma, FineWeb), filtering and deduplication research (Lee et al. 2022, quality classifiers, MinHash, SemDeDup), training-data contamination distinct from benchmark contamination, the synthetic-data turn (Phi training, persona-driven generation), and the legal arc (NYT v. OpenAI, Authors Guild, Bartz v. Anthropic 2024 settlement, Getty v. Stability, Books3, EU AI Act's training-summary requirement). Intelligent generalist; technical on filtering mechanics, accessible on legal/policy.

### Code-specific NLP

File: `code-models.md`

Trace the code-generation lineage: pre-LLM (TabNine, Kite), Codex (Chen et al. 2021) and Copilot, AlphaCode (DeepMind 2022), the open code-model family (StarCoder/StarCoder2 from BigCode, CodeLlama, DeepSeek-Coder, Qwen-Coder, Codestral), and the agentic coding-product wave (Cursor, Aider, Claude Code, Devin, Replit Agent). Cover what makes code different (executable verification, repo-level context, fill-in-the-middle, infill tokens), the code-eval arc (HumanEval saturation → MBPP → SWE-bench → Verified → Pro → BigCodeBench), and the link between code-RL with verifiable rewards and the broader reasoning-models story. Technical where architecture/objective matters.

### Inference infrastructure and the serving stack

File: `inference-infrastructure.md`

Cover the engineering layer that made LLM deployment economically real: KV cache mechanics, continuous batching (vLLM, Kwon et al. 2023 PagedAttention), speculative decoding (Leviathan et al. 2023, Medusa, EAGLE), inference quantization (GPTQ, AWQ, SmoothQuant, GGUF, llama.cpp's q4/q5/q8), frameworks (TensorRT-LLM, vLLM, SGLang, TGI, MLX, llama.cpp), and on-device serving (MLX, Qualcomm AI Hub, ExecuTorch). Frame these systems' role in the open-weights story (DeepSeek-V3 at 2200 tok/sec/H200, Phi/Gemma on phones) and the FP8/FP4 train-infer convergence. Technical depth; sketch paged attention and speculative-decoding accept/reject math.

### Speech ↔ text convergence

File: `speech-and-text.md`

Cover speech NLP as it converged with text: HMM/GMM ASR origins (DARPA evals, Kaldi), the deep-learning turn (DeepSpeech 1/2, Listen-Attend-Spell, RNN-T, Conformer), the self-supervised wave (wav2vec 2.0, HuBERT, w2v-BERT), Whisper (Radford et al. 2022) as the moment ASR became upstream glue, audio-LMs (AudioLM, AudioPaLM, Moshi, GPT-4o native audio), TTS in parallel (Tacotron 2, FastSpeech, VALL-E, NaturalSpeech, ElevenLabs, OpenAI voice), and the 2024–2026 unified text-speech tokenization trend (audio tokens, RVQ, neural codecs). Intelligent generalist; technical on architectural choices.

### Red-teaming, jailbreaks, prompt injection

File: `red-teaming-jailbreaks.md`

Cover the offensive/security side of LLM safety: early jailbreaks (DAN, role-play, suffix attacks, the GCG attack — Zou et al. 2023), academic adversarial-prompt research (Wei et al. on safety-tuning failure modes), Anthropic and OpenAI red-team methodologies (blue-team/red-team frameworks, OpenAI's Preparedness Framework), prompt injection as a class (Greshake et al. 2023's "indirect" injection, the agent-era escalation), post-training defenses (safety classifiers, refusal training and its limits, instruction hierarchy, output filtering), and the live arms race. Connect back to evaluation (Apollo's scheming evals, evaluation awareness) and the alignment arc. Intelligent generalist with technical depth on attacks/defenses.
