# Frontier Directions in NLP: Where the Field is Heading

*Research dossier — Round 1, Topic 16. Layered horizons: ~45% near-term (1–2 years), ~45% medium-term (5 years), ~10% long-term (10–20 years). Claims are flagged as **consensus**, **contested**, or **fringe**.*

The transformer era of NLP, opened by Vaswani et al. in 2017, has matured into something stranger than a single trajectory. By mid-2025 the field is simultaneously: doubling down on scale (DeepSeek-V3 is a 671B-parameter mixture-of-experts trained for under $6M [T1]); reorganising around a new training paradigm where models think before they answer (o1, o3, R1); shrinking aggressively onto phones (Apple's 3B on-device model, Phi-4, Gemma 3); and entertaining serious post-LLM bets (state-space models, JEPA, neurosymbolic revival). What follows is a structured tour, weighted toward the near and medium term, with the long-term debates kept honest about how much is actually known.

## 1. The Reasoning Turn: Test-Time Compute as a Second Scaling Axis

The most consequential near-term shift is the emergence of **reasoning models** as a distinct family. OpenAI's o1, released in September 2024, was the first widely-deployed system to make "thinking before answering" a first-class behaviour, with internal chains of thought generated via reinforcement learning [T2] OpenAI, *Learning to Reason with LLMs*, openai.com/index/learning-to-reason-with-llms. The blog post explicitly framed this as a new scaling law: performance "consistently improves with more reinforcement learning (train-time compute) and with more time spent thinking (test-time compute)." This is **consensus** — every frontier lab has shipped a reasoning variant within twelve months.

DeepSeek-R1, published in *Nature* in September 2025 (the first major open-weight LLM to clear independent peer review), made a sharper claim: reasoning ability can be *incentivised through pure reinforcement learning*, with no human-labelled reasoning trajectories required [T1] DeepSeek-AI, "DeepSeek-R1 incentivizes reasoning in LLMs through reinforcement learning," *Nature* 645, pp. 633–638 (2025), abstract ¶1–2. The R1-Zero variant developed self-reflection, verification, and dynamic strategy adaptation as *emergent* behaviours under an RL objective on verifiable tasks (math, code, STEM). The arXiv preprint (2501.12948) and the paper itself are now the canonical reference for the GRPO training algorithm and the RL-only recipe [T1] arXiv:2501.12948.

The implications are still being absorbed:

- **Compute split shifts.** Inference-time compute is no longer a fixed cost per token; reasoning models can spend orders of magnitude more compute on a single hard problem. NVIDIA and serving stacks have optimised around this — vLLM benchmarks now report sustained ~2,200 tokens/sec/H200 on DeepSeek-class MoE models [T3] vLLM Blog, "Large Scale Serving," blog.vllm.ai/2025/12/17/large-scale-serving.html.
- **Verifiability matters.** RL-only reasoning training works best where reward signals are crisp (proofs, unit-tested code). It is **contested** whether it generalises to fuzzy domains like literary analysis or strategic judgment. A 2025 EMNLP findings paper specifically evaluated o1, R1 and others on legal reasoning and found uneven gains [T1] *Evaluating Test-Time Scaling LLMs for Legal Reasoning*, ACL Anthology 2025.findings-emnlp.742.
- **The "data wall" gets a temporary detour.** If reasoning quality scales with RL compute on verifiable tasks, and synthetic reasoning traces can be generated at scale from those tasks, the training-data bottleneck on the language side becomes less acute — at least in math/code/science. **Contested.**

## 2. Agents and Computer Use: Promise vs. Reliability

Closely linked to the reasoning turn is the **agentic** push: models that act in the world via tools, browsers, and operating systems. Anthropic shipped Claude 3.5 Sonnet's *computer use* in public beta in October 2024, with the model perceiving via screenshots and acting via cursor/keyboard primitives. OSWorld scores were 14.9% screenshot-only versus 7.8% for the next-best system [T2] Anthropic, "Introducing computer use, a new Claude 3.5 Sonnet, and Claude 3.5 Haiku," anthropic.com/news/3-5-models-and-computer-use. OpenAI followed with Operator (CUA model) in January 2025, then folded it into ChatGPT's agent mode by August 2025 [T3] AIWiki, "OpenAI Operator," aiwiki.ai/wiki/openai_operator.

What's **consensus**: agents work for narrow, well-bounded workflows (form-filling, data extraction, code execution loops). What's **contested**: how reliable they are over long horizons. The AI2 Incubator's 2025 "State of AI Agents" report frames the gap between demos and production deployment as the central practical question of the next 18 months [T3] AI2 Incubator, *Insights 15*, ai2incubator.com/articles/insights-15-the-state-of-ai-agents-in-2025. Reasoning models help here — extended thinking gives an agent room to plan and recover from errors — but cumulative error rates on multi-step tasks remain the dominant failure mode. This is the tightest near-term coupling between research progress and revenue: most enterprise AI budgets in 2025–2026 are betting on agents rather than chat.

## 3. Small Models, On-Device, and the Efficiency Frontier

While headlines chase frontier scale, the **other half of the field is shrinking**. Three reference points:

- **Apple Intelligence on-device** ships a ~3B-parameter dual-block model with shared KV cache (37.5% memory reduction), 2-bit quantisation-aware training, and a 65K context window. It "performs favorably against the slightly larger Qwen-2.5-3B across all languages" and is "competitive against the larger Qwen-3-4B and Gemma-3-4B in English" [T2] Apple ML Research, *Updates to Apple's On-Device and Server Foundation Language Models* (2025), machinelearning.apple.com/research/apple-foundation-models-2025-updates.
- **Microsoft Phi-4** (3.8B mini, 5.6B multimodal) doubles down on the "data quality over scale" thesis, with a training corpus heavy in synthetic data generated by GPT-4 [T3] Adnan Masood, "Small Language Models: The Rise of Compact AI," Medium.
- **Google Gemma 3** at 4B reports 71.3% HumanEval and 89.2% GSM8K — strong numbers historically associated with much larger models [T3] DataCamp, "Top 15 Small Language Models for 2026," datacamp.com/blog/top-small-language-models.

The structural driver is **algorithmic efficiency**. Epoch AI's analyses estimate the compute needed to reach a given capability is currently dropping by ~3× per year at the model-quality level, and ~18× per year at the cost-of-deployment level once hardware price decline is factored in [T2] Epoch AI, "Algorithmic progress in language models," epoch.ai/blog/algorithmic-progress-in-language-models. The Epoch March-2024 paper put the historical rate at "doubling computational power every 5 to 14 months" of effective progress [T1] Ho, Besiroglu et al., "Algorithmic Progress in Language Models," arXiv:2403.05812. **Consensus**: capabilities are commoditising downward fast. **Contested**: whether the same algorithmic gains lift the *frontier* as much as they compress the trailing edge.

This matters because it changes the distribution of where NLP runs. A 2026 forecast tracked by Meta's ExecuTorch shows 80%+ of popular edge models running out-of-the-box across Apple, Qualcomm, Arm, MediaTek and Vulkan backends, with edge AI deployment in manufacturing growing 3× from 2025 to 2026 [T3] localaimaster.com/blog/small-language-models-guide-2026.

## 4. The Data Wall and the Synthetic-Data Question

A central **medium-term** question: does pretraining hit a wall, and when? Epoch AI's widely-cited analysis projects the stock of high-quality public text will be fully used between 2025 and 2030 depending on overtraining ratios; 100× overtraining exhausts the stock by 2025 [T1] Villalobos et al., "Will we run out of data? Limits of LLM scaling based on human-generated data," arXiv:2211.04325.

Their follow-up "Can AI scaling continue through 2030?" lays out the constraint stack with surprising specificity [T2] Epoch AI, epoch.ai/blog/can-ai-scaling-continue-through-2030: training runs of ~2×10²⁹ FLOP look feasible by decade's end (roughly 5,000× Llama-3.1-405B's ~4×10²⁵ FLOP). The binding constraints, in order:

1. **Power** — single campuses 1–5 GW by 2030; distributed networks 2–45 GW.
2. **Chip manufacturing** — capacity for ~100M H100-equivalents (range 20M–400M).
3. **Data** — 400 trillion to 20 quadrillion effective tokens through multimodal sources.
4. **Latency** — ceiling around 3×10³⁰ to 1×10³² FLOP, less binding than the others.

Power, not data, is the most likely cap. **Consensus** on the rough ordering; **contested** on whether algorithmic and synthetic-data gains effectively dissolve the data constraint. Microsoft Research's SynthLLM work claims synthetic data scaling laws hold in narrow domains [T3] Microsoft Research, "SynthLLM: Breaking the AI 'data wall' with scalable synthetic data," microsoft.com/en-us/research/articles/synthllm-breaking-the-ai-data-wall. The honest reading is that synthetic data clearly works for math/code/structured reasoning and is unproven elsewhere.

## 5. Architectural Shifts: MoE, State-Space Models, and Hybrids

By 2025, **Mixture-of-Experts has effectively won** at the frontier. NVIDIA reports MoE underpins "virtually any frontier model today" and over 60% of open-source releases in 2025 — DeepSeek-R1, Kimi K2, Mistral Large 3 all MoE [T3] NVIDIA Blog, "Mixture of Experts Powers the Most Intelligent Frontier AI Models," blogs.nvidia.com/blog/mixture-of-experts-frontier-models. DeepSeek-V3's technical report details 671B total / 37B active per token, an auxiliary-loss-free load-balancing scheme, multi-token prediction, and FP8 mixed-precision training — all at $5.576M total cost [T1] DeepSeek-AI, "DeepSeek-V3 Technical Report," arXiv:2412.19437. **Consensus**.

Beyond MoE, the more interesting structural bet is **state-space models**. Mamba (Gu & Dao, 2023) introduced *selective* SSMs whose parameters are functions of the input, allowing content-aware information propagation along sequence dimensions [T1] Gu & Dao, "Mamba: Linear-Time Sequence Modeling with Selective State Spaces," arXiv:2312.00752, abstract. Headline claims: 5× higher throughput than Transformers, linear scaling in sequence length, Mamba-3B matching Transformers twice its size on language modeling. Mamba-2 in 2024 (Dao & Gu) framed Transformers and SSMs as duals under a "structured state space duality," with a 2–8× speedup over Mamba-1 [T1] arXiv:2405.21060.

The real-world picture is more nuanced — and instructive about how the field actually evolves. The Goomba Lab's 2025 retrospective on SSM/Transformer tradeoffs argues SSMs have specific weaknesses (in-context learning, induction heads, exact copying of long inputs) that Transformers handle well [T3] goombalab.github.io/blog/2025/tradeoffs. The empirical winner so far is **hybrid architectures** that interleave attention and SSM blocks. A 2025 *Scientific Reports* paper formalised this hybrid framing for sequence modeling broadly [T1] *Scientific Reports* 2025, nature.com/articles/s41598-025-87574-8. **Contested**: whether hybrid SSMs displace pure Transformers at the frontier in 5 years, or remain a niche optimisation for long-context regimes.

## 6. Alignment, Interpretability, and Scalable Oversight Maturing

In parallel with capability gains, **mechanistic interpretability** has moved from speculative research to a recognised engineering discipline. Anthropic's interpretability team has set an explicit target: "interpretability can reliably detect most model problems" by 2027 [T2] Anthropic, "Interpretability Research," anthropic.com/research/team/interpretability. Circuit tracing — open-sourced in 2025 — produces attribution graphs showing internal computation steps and revealing language-independent conceptual representations [T2] Anthropic, "Recommendations for Technical AI Safety Research Directions," alignment.anthropic.com/2025/recommended-directions.

**Scalable oversight** — supervising models smarter than us — remains the harder open problem. Anthropic's 2025 work on Automated Alignment Researchers tested whether Claude could autonomously make progress on oversight research itself, with mixed but non-trivial results [T2] Anthropic, "Automated Alignment Researchers: Using large language models to scale scalable oversight," anthropic.com/research/automated-alignment-researchers. **Consensus**: interpretability has made real progress on small/medium models; **contested**: whether techniques scale to frontier systems before capabilities outrun them. **Fringe**: the position that alignment is essentially solved or essentially impossible — neither has weight among working researchers in 2025.

## 7. Post-LLM Bets: World Models and the Neurosymbolic Revival

The medium-to-long-term **contested** territory is whether the LLM paradigm itself is the endgame.

**Yann LeCun's JEPA programme** is the most prominent post-LLM bet from someone with the track record to be taken seriously. JEPA (Joint-Embedding Predictive Architecture) predicts internal world representations rather than tokens or pixels, with the explicit thesis that next-token prediction on text is a fundamentally limited substrate for general intelligence. V-JEPA2 shipped in June 2025; LeCun published *LeJEPA* (Provable and Scalable Self-Supervised Learning Without the Heuristics) in late 2025, providing a theoretical framework [T3] Turing Post, "AI 101: What is LeJEPA?", turingpost.com/p/lejepa. On 27 October 2025 he predicted LLMs would be "useless within five years"; on 19 November 2025 he announced his departure from Meta to found a company focused on Advanced Machine Intelligence (AMI) [T3] Royfactory, royfactory.net/posts/ai/202510/yann-lecun-2025-llm-doomed-jepa-world-model. This is **fringe-to-contested** — high-conviction from a Turing laureate, but not the field consensus. A hybrid "LLM-JEPA" approach (arXiv:2509.14252, Sept 2025) suggests the camps are not fully disjoint [T1] arXiv:2509.14252.

**Neurosymbolic AI** has a quieter but real revival. Gartner placed it in the 2025 AI Hype Cycle [T3] AllegroGraph, "The Rise of Neuro-Symbolic AI," allegrograph.com/the-rise-of-neuro-symbolic-ai. Amazon deployed neurosymbolic components in Vulcan warehouse robotics and the Rufus shopping assistant, with the explicit motivation of constraining hallucination and improving auditability. A 2025 review in the *Arabian Journal for Science and Engineering* surveyed the space for robustness, uncertainty quantification, and intervenability [T1] Springer, link.springer.com/article/10.1007/s13369-025-10887-3. Gary Marcus — long the most public neurosymbolic advocate — has flagged the renewed interest [T3] garymarcus.substack.com/p/even-more-good-news-for-the-future. **Contested**: whether neurosymbolic methods become a layer atop LLMs (consensus reading) or a genuine alternative paradigm (Marcus's stronger reading).

## 8. The AGI Question: Where Disagreement Actually Lies

The **long-term** section deserves epistemic humility. The honest framing of the AGI debate isn't "when?" but "what are we even arguing about?". The IEEE Spectrum 2025 piece on AGI benchmarks captures the diagnosis: "people strongly disagree on AGI's definition" — performance benchmarks, internal mechanisms, economic impact, or vibes [T3] IEEE Spectrum, "AGI Benchmarks: Tracking Progress Toward AGI Isn't Easy," spectrum.ieee.org/agi-benchmark.

The most rigorous benchmark explicitly designed to measure the gap is **ARC-AGI** (François Chollet, 2019; ARC-AGI-2 in 2025). The 2025 numbers are revealing [T1] Chollet et al., "ARC-AGI-2: A New Challenge for Frontier AI Reasoning Systems," arXiv:2505.11831:

- Human panels: 64.2% on ARC-AGI-1, 60% on ARC-AGI-2.
- o3-preview-low: 75.7% on ARC-AGI-1, **4% on ARC-AGI-2**.
- DeepSeek-R1: 15.8% / 0.3%.

In other words, the same reasoning models that look superhuman on ARC-AGI-1 collapse on ARC-AGI-2, whose tasks every problem was solved by ≥2 humans in ≤2 attempts. Chollet's operational definition: AGI is the gap between tasks easy for humans and hard for AI — when the gap is zero, AGI is achieved. By that ruler, we are clearly not there. **Consensus** that ARC-AGI-2 is unsolved; **contested** whether it's the right ruler.

The serious disagreements among working researchers in 2025:

1. **Is scale all you need?** Sutskever, Amodei, Altman lean yes (with reasoning + agents as the next ingredients). LeCun, Chollet, Marcus lean no.
2. **Are current models reasoning or pattern-matching at higher fidelity?** Reasoning models complicate this — chain-of-thought *plus* RL on verifiable rewards is empirically more than next-token prediction, but how much more is contested.
3. **What's the bottleneck this decade?** Epoch's data says power, then chips, then data. Compute optimists say algorithmic efficiency dissolves these. Compute pessimists say each constraint cascades.
4. **Civilisational implications.** "Intelligence as commodity" arguments (Altman, Karpathy at times) versus "transformation of work and governance" arguments (Hinton, Russell). **Fringe** at both ends: utopian abundance and existential risk certainty.

## 9. What to Watch Next

The single highest-information question for the next 24 months is whether **reasoning + agents + algorithmic efficiency** compound into a regime change, or whether the curves flatten as the data wall, power constraints, and reliability gaps bite. The 2027 timeline matters: Epoch projects training runs longer than ~9 months become inefficient at the current pace, hitting that ceiling around 2027 [T2] Epoch AI, "Frontier LLM training runs can't get much longer," epoch.ai/data-insights/longest-training-run. Anthropic's interpretability target is also 2027. ARC-AGI-2 has $700K Grand Prize unlocked at >85%, due 3 November 2025 (closed at the time of writing).

Three concrete things to track:

1. **Which open-weights reasoning model first matches o3 on ARC-AGI-2** — tells us whether RL-on-verifiable-tasks scales as a recipe.
2. **First production agent deployments crossing 90%+ task completion at human-comparable cost** — moves agents from demo to economic primitive.
3. **Whether a non-Transformer architecture (Mamba-class, JEPA-class) ships at frontier scale and matches Transformer quality on a generalist evaluation** — would be the first real evidence of architectural displacement.

If all three happen by 2027, the field has changed paradigm. If none do, we are scaling and refining the 2024 stack for another five years.

---

## Sources

- **[T1]** Vaswani et al., "Attention Is All You Need," NeurIPS 2017. https://arxiv.org/abs/1706.03762 — Type 1.
- **[T1]** DeepSeek-AI, "DeepSeek-R1 incentivizes reasoning in LLMs through reinforcement learning," *Nature* 645, 633–638, 18 Sept 2025. https://www.nature.com/articles/s41586-025-09422-z — Type 1.
- **[T1]** DeepSeek-AI, "DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning," arXiv:2501.12948 (Jan 2025; rev. Jan 2026). https://arxiv.org/abs/2501.12948 — Type 1.
- **[T1]** DeepSeek-AI, "DeepSeek-V3 Technical Report," arXiv:2412.19437 (Dec 2024). https://arxiv.org/abs/2412.19437 — Type 1.
- **[T1]** Gu & Dao, "Mamba: Linear-Time Sequence Modeling with Selective State Spaces," arXiv:2312.00752 (Dec 2023). https://arxiv.org/abs/2312.00752 — Type 1.
- **[T1]** Dao & Gu, "Transformers are SSMs: Generalized Models and Efficient Algorithms Through Structured State Space Duality," arXiv:2405.21060 (May 2024). https://arxiv.org/abs/2405.21060 — Type 1.
- **[T1]** Chollet et al., "ARC-AGI-2: A New Challenge for Frontier AI Reasoning Systems," arXiv:2505.11831 (May 2025). https://arxiv.org/abs/2505.11831 — Type 1.
- **[T1]** Villalobos et al., "Will we run out of data? Limits of LLM scaling based on human-generated data," arXiv:2211.04325. https://arxiv.org/pdf/2211.04325 — Type 1.
- **[T1]** Ho, Besiroglu et al., "Algorithmic Progress in Language Models," arXiv:2403.05812 (Mar 2024). https://arxiv.org/pdf/2403.05812 — Type 1.
- **[T1]** "LLM-JEPA: Large Language Models Meet Joint Embedding Predictive Architectures," arXiv:2509.14252 (Sept 2025). https://arxiv.org/abs/2509.14252 — Type 1.
- **[T1]** "Evaluating Test-Time Scaling LLMs for Legal Reasoning: OpenAI o1, DeepSeek-R1, and Beyond," ACL Anthology 2025.findings-emnlp.742. https://aclanthology.org/2025.findings-emnlp.742/ — Type 1.
- **[T1]** "A hybrid model based on transformer and Mamba for enhanced sequence modeling," *Scientific Reports* 2025. https://www.nature.com/articles/s41598-025-87574-8 — Type 1.
- **[T1]** "A Comprehensive Review of Neuro-symbolic AI for Robustness, Uncertainty Quantification, and Intervenability," *Arabian J. of Science and Engineering* (Springer, 2025). https://link.springer.com/article/10.1007/s13369-025-10887-3 — Type 1.
- **[T2]** OpenAI, "Learning to reason with LLMs," 12 Sept 2024. https://openai.com/index/learning-to-reason-with-llms/ — Type 2.
- **[T2]** Anthropic, "Introducing computer use, a new Claude 3.5 Sonnet, and Claude 3.5 Haiku," 22 Oct 2024. https://www.anthropic.com/news/3-5-models-and-computer-use — Type 2.
- **[T2]** Apple Machine Learning Research, "Updates to Apple's On-Device and Server Foundation Language Models" (2025). https://machinelearning.apple.com/research/apple-foundation-models-2025-updates — Type 2.
- **[T2]** Apple, "Apple Intelligence Foundation Language Models Tech Report 2025," arXiv:2507.13575. https://arxiv.org/pdf/2507.13575 — Type 2.
- **[T2]** Epoch AI, "Will we run out of data to train large language models?" https://epoch.ai/blog/will-we-run-out-of-data-limits-of-llm-scaling-based-on-human-generated-data — Type 2.
- **[T2]** Epoch AI, "Can AI scaling continue through 2030?" https://epoch.ai/blog/can-ai-scaling-continue-through-2030 — Type 2.
- **[T2]** Epoch AI, "Algorithmic progress in language models." https://epoch.ai/blog/algorithmic-progress-in-language-models — Type 2.
- **[T2]** Epoch AI, "Frontier LLM training runs can't get much longer." https://epoch.ai/data-insights/longest-training-run — Type 2.
- **[T2]** Anthropic, "Interpretability Research." https://www.anthropic.com/research/team/interpretability — Type 2.
- **[T2]** Anthropic, "Recommendations for Technical AI Safety Research Directions," 2025. https://alignment.anthropic.com/2025/recommended-directions/ — Type 2.
- **[T2]** Anthropic, "Automated Alignment Researchers: Using large language models to scale scalable oversight." https://www.anthropic.com/research/automated-alignment-researchers — Type 2.
- **[T3]** vLLM Blog, "Large Scale Serving: DeepSeek @ 2.2k tok/s/H200 with Wide-EP," 17 Dec 2025. https://blog.vllm.ai/2025/12/17/large-scale-serving.html — Type 3.
- **[T3]** AI2 Incubator, "Insights 15: The State of AI Agents in 2025." https://www.ai2incubator.com/articles/insights-15-the-state-of-ai-agents-in-2025 — Type 3.
- **[T3]** AIWiki, "OpenAI Operator." https://aiwiki.ai/wiki/openai_operator — Type 3.
- **[T3]** Adnan Masood, "Small Language Models: The Rise of Compact AI and Microsoft's Phi Models," Medium. https://medium.com/@adnanmasood/small-language-models-the-rise-of-compact-ai-and-microsofts-phi-models-cdc7cf20ea2d — Type 3.
- **[T3]** DataCamp, "Top 15 Small Language Models for 2026." https://www.datacamp.com/blog/top-small-language-models — Type 3.
- **[T3]** Local AI Master, "Best Small AI Models to Run with Ollama (2026)." https://localaimaster.com/blog/small-language-models-guide-2026 — Type 3.
- **[T3]** NVIDIA Blog, "Mixture of Experts Powers the Most Intelligent Frontier AI Models." https://blogs.nvidia.com/blog/mixture-of-experts-frontier-models/ — Type 3.
- **[T3]** Microsoft Research, "SynthLLM: Breaking the AI 'data wall' with scalable synthetic data." https://www.microsoft.com/en-us/research/articles/synthllm-breaking-the-ai-data-wall-with-scalable-synthetic-data/ — Type 3.
- **[T3]** Goomba Lab, "On the Tradeoffs of SSMs and Transformers," 2025. https://goombalab.github.io/blog/2025/tradeoffs/ — Type 3.
- **[T3]** The Gradient, "Mamba Explained." https://thegradient.pub/mamba-explained/ — Type 3.
- **[T3]** Royfactory, "Yann LeCun Declares LLMs 'Useless in Five Years'," Oct 2025. https://royfactory.net/posts/ai/202510/yann-lecun-2025-llm-doomed-jepa-world-model/ — Type 3.
- **[T3]** Turing Post, "AI 101: What is LeJEPA?" https://www.turingpost.com/p/lejepa — Type 3.
- **[T3]** Meta AI, "V-JEPA: The next step toward advanced machine intelligence." https://ai.meta.com/blog/v-jepa-yann-lecun-ai-model-video-joint-embedding-predictive-architecture/ — Type 3.
- **[T3]** AllegroGraph, "The Rise of Neuro-Symbolic AI: A Spotlight in Gartner's 2025 AI Hype Cycle." https://allegrograph.com/the-rise-of-neuro-symbolic-ai/ — Type 3.
- **[T3]** Gary Marcus, "Even more good news for the future of neurosymbolic AI," Substack. https://garymarcus.substack.com/p/even-more-good-news-for-the-future — Type 3.
- **[T3]** IEEE Spectrum, "AGI Benchmarks: Tracking Progress Toward AGI Isn't Easy." https://spectrum.ieee.org/agi-benchmark — Type 3.
- **[T3]** ARC Prize, "Announcing ARC-AGI-2 and ARC Prize 2025." https://arcprize.org/blog/announcing-arc-agi-2-and-arc-prize-2025 — Type 3.
- **[T3]** Sebastian Raschka, "The State Of LLMs 2025: Progress, Progress, and Predictions." https://magazine.sebastianraschka.com/p/state-of-llms-2025 — Type 3.
