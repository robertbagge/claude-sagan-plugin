# Scaling Laws & Emergent Abilities

The empirical scaling story is the closest thing modern NLP has to a quantitative theory. Between 2020 and 2023 it went through three distinct phases: Kaplan et al. argued loss falls as a clean power law in compute, parameters, and data, and that — under a fixed compute budget — you should spend most of it on parameters. Hoffmann et al. (Chinchilla) re-ran the experiment more carefully and concluded the opposite: parameters and tokens should grow at roughly equal rates, and existing models like GPT-3 and Gopher were badly under-trained. Then a parallel literature opened on *emergent abilities* — sharp, scale-dependent jumps in downstream capability — which Schaeffer et al. argued were largely artifacts of the metrics being used. This dossier walks through each of those threads with the actual equations and exponents on the table.

## 1. Kaplan et al. 2020: The Original Power-Law Picture

Kaplan and collaborators at OpenAI ran a sweep of decoder-only Transformers across roughly seven orders of magnitude of compute and reported that, when one of the three resources (non-embedding parameter count `N`, dataset size in tokens `D`, or compute `C`) is the binding constraint, the cross-entropy test loss on WebText2 follows a power law in that resource [T1] Kaplan et al. 2020, p. 1, abstract.

The headline equations (their Section 1.1, "Summary"):

- `L(N) = (N_c / N)^α_N` with `α_N ≈ 0.076`, `N_c ≈ 8.8 × 10^13` non-embedding parameters [T1] Kaplan et al. 2020, §1.1, eq. 1.5.
- `L(D) = (D_c / D)^α_D` with `α_D ≈ 0.095`, `D_c ≈ 5.4 × 10^13` tokens [T1] Kaplan et al. 2020, §1.1, eq. 1.5.
- `L(C_min) = (C_c^min / C_min)^α_C^min` with `α_C^min ≈ 0.050`, `C_c^min ≈ 3.1 × 10^8` PF-days [T1] Kaplan et al. 2020, §1.1, eq. 1.5.

Two qualitative claims fall out of this. First, **other architectural details barely matter**: depth-vs-width ratios, attention-head counts, feed-forward dimensions all wash out next to total parameter count, "within a wide range" [T1] Kaplan et al. 2020, p. 1, abstract. This is the "Bitter Lesson"-flavoured result that justified industrial-scale scaling: don't fiddle with the architecture, just throw more parameters at it.

Second, larger models are **more sample-efficient** — they reach a given loss after seeing fewer tokens. Combining this with the compute power law, Kaplan et al. concluded that compute-optimal training means training very large models on a relatively modest amount of data and stopping well before convergence [T1] Kaplan et al. 2020, p. 1, abstract. Operationally this turned into the rule-of-thumb that, given extra compute, parameters should grow roughly as `N ∝ C^0.73` and data only as `D ∝ C^0.27` — i.e. parameters should outpace data by a factor of ~2.7× per doubling of compute [T1] Kaplan et al. 2020, §6.

They also gave a combined two-variable fit:

`L(N, D) = [(N_c / N)^(α_N / α_D) + D_c / D]^α_D`

which lets you predict overfitting onset. Solving the gradient of this with respect to `N` at fixed `D` gives the empirical scaling `D ∝ N^0.74` for keeping overfitting under control [T2] Brenndoerfer, "Scaling Laws for Neural Language Models", https://mbrenndoerfer.com/writing/scaling-laws-neural-language-models-power-law-predictions.

A separate result that mattered for practical training was the **critical batch size**, the size beyond which data parallelism stops paying off:

`B_crit(L) = B_* / L^(1/α_B)`, with `B_* ≈ 2 × 10^8` tokens, `α_B ≈ 0.21` [T1] Kaplan et al. 2020, §1.1.

The Kaplan paper became, alongside the original Transformer paper, one of the two documents that justified the GPT-3-era scale-up: it gave a quantitative story for *why* scale should keep paying off and *how much* of it to spend where.

## 2. Hoffmann et al. 2022: Chinchilla Rewrites the Recipe

Two years later DeepMind's Chinchilla paper (Hoffmann et al., March 2022) re-ran the question with a much wider sweep — over 400 language models from 70M to 16B+ parameters, trained on 5B to 500B tokens — and reached an essentially incompatible conclusion: **for compute-optimal training, parameters and tokens should be scaled approximately equally** [T1] Hoffmann et al. 2022, p. 1, abstract.

They used three independent estimation approaches and the answers agreed:

- **Approach 1 (training-curve envelope):** vary training duration across fixed model families, take the lower envelope of loss-vs-FLOPs curves. Result: `a ≈ 0.50`, `b ≈ 0.50` [T1] Hoffmann et al. 2022, §3.1.
- **Approach 2 (IsoFLOP profiles):** at each fixed FLOP budget, sweep model size and fit a parabola to find the optimum. Result: `a ≈ 0.49`, `b ≈ 0.51` [T1] Hoffmann et al. 2022, §3.2.
- **Approach 3 (parametric loss fit):** fit `L(N, D) = E + A/N^α + B/D^β` and minimise under a FLOP constraint. Result: `a ≈ 0.46`, `b ≈ 0.54` [T1] Hoffmann et al. 2022, §3.3.

Here `a` and `b` are the exponents in `N_opt ∝ C^a` and `D_opt ∝ C^b`. Compare to Kaplan's `a ≈ 0.73, b ≈ 0.27`. The disagreement is large enough that the same compute budget would imply very different model sizes under the two recipes — and the Chinchilla numbers cleanly imply that essentially every flagship model trained between 2020 and 2022 was *under-trained on data* given its parameter count.

The validation experiment was dramatic. Using the exact same compute budget DeepMind had spent on the 280B-parameter Gopher model, the team trained **Chinchilla: 70B parameters on 1.4 trillion tokens** — four times the data, one quarter the parameters [T1] Hoffmann et al. 2022, p. 1, abstract. Chinchilla beat Gopher (280B), GPT-3 (175B), Jurassic-1 (178B), and Megatron-Turing NLG (530B) on a wide range of downstream tasks, and pushed MMLU from Gopher's ~60% to 67.5% — a >7 point jump from a *smaller* model [T1] Hoffmann et al. 2022, §4.

The colloquial reading: "20 tokens per parameter" became the rule of thumb practitioners adopted [T2] Alan D. Thompson, "Chinchilla data-optimal scaling laws: in plain English", https://lifearchitect.ai/chinchilla/. Llama-1 (Feb 2023) explicitly trained past this ratio in pursuit of better inference economics — paying extra training compute for a smaller, cheaper-to-serve model — and Llama-2 and most open releases since have done the same.

### A 2024 footnote: the replication attempt

Tamay Besiroglu, Ege Erdil and colleagues at Epoch AI tried to replicate Approach 3 from the Chinchilla paper by reconstructing data from the published plots (since the raw runs were never released). They found Hoffmann et al.'s reported parametric fit was inconsistent with their other two approaches, fitted the extracted data poorly, and reported confidence intervals so narrow they would require over 600,000 experiments to justify — when likely fewer than 500 were actually run [T1] Besiroglu et al. 2024, https://arxiv.org/abs/2404.10102, abstract. Their re-derivation of Approach 3 produced results compatible with Approaches 1 and 2; the 20-tokens-per-parameter headline holds, but the precision claimed in the original paper was overstated. This is a useful reminder that the "scaling laws" we lean on are noisy empirical fits, not physical constants.

## 3. Where Kaplan and Chinchilla Disagreed (and Why)

The interesting question is *why* the two papers reached such different exponents from very similar setups. The Chinchilla authors and several follow-ups attribute the gap to three methodological choices in Kaplan et al.:

1. **Learning-rate schedule.** Kaplan used a single cosine schedule mostly tuned to long runs, which under-trains short runs and biases the inferred frontier toward "more parameters" [T2] Brenndoerfer, "Chinchilla scaling laws", https://mbrenndoerfer.com/writing/chinchilla-scaling-laws-compute-optimal-training-resource-allocation.
2. **Embedding parameters.** Kaplan reported `N` as non-embedding parameters; for small models this materially shifts the apparent compute-per-parameter cost.
3. **Limited data range.** Kaplan's largest runs were ~1024×-smaller in tokens than Chinchilla's, so the data-side power law was extrapolated from a narrow window.

Whatever the precise weights, the practical takeaway after 2022 was that Kaplan was directionally right (loss does fall as a power law in each axis) but quantitatively wrong about how to *spend* a compute budget. The community quietly moved to Chinchilla-style ratios and most public models after mid-2022 were trained under that recipe.

## 4. Wei et al. 2022: Emergent Abilities

In parallel with the scaling-law debate, Jason Wei and 15 co-authors compiled Wei et al. 2022, "Emergent Abilities of Large Language Models" [T1] Wei et al. 2022, https://arxiv.org/abs/2206.07682. Their core definition: an ability is **emergent** if it is *absent* in smaller models but *present* in larger ones, and cannot be predicted by extrapolating from smaller models [T1] Wei et al. 2022, p. 1, abstract.

The empirical content is a catalogue. Wei et al. and Wei's follow-up blog identified over 100 individual tasks where this pattern shows up [T3] Wei, "137 emergent abilities of large language models", https://www.jasonwei.net/blog/emergence. The largest single sources were:

- **BIG-bench** (Srivastava et al. 2022, the 204-task "Beyond the Imitation Game" benchmark contributed by 450 authors across 132 institutions) — 67 emergent tasks [T1] Srivastava et al. 2022, https://arxiv.org/abs/2206.04615.
- **MMLU** (Massive Multitask Language Understanding) — 51 emergent tasks [T3] Wei, https://www.jasonwei.net/blog/emergence.

Specific examples in the paper that became canonical talking points:

- **Modular arithmetic** (e.g. 3-digit addition mod some base): essentially flat near-zero accuracy below ~10^22 training FLOPs, then a sharp climb [T1] Wei et al. 2022, Fig. 2.
- **Multi-step word unscrambling**: similar threshold behaviour.
- **IPA transliteration** (mapping text into the International Phonetic Alphabet): another discontinuity.
- **MMLU**: roughly chance until some model size, then rises above chance.

A second category they highlight is **emergent prompting strategies**: techniques that *fail to help* small models and only become useful past a scale threshold. **Chain-of-thought prompting** is the canonical case — for math word problems it provides essentially no benefit (or hurts) below ~100B parameters and then becomes the dominant prompting technique for larger models [T1] Wei et al. 2022, §5. Few-shot in-context learning itself shows a milder version of the same shape: GPT-3-scale models exhibit it, GPT-2-scale models largely do not.

The framing was deliberately provocative: if abilities truly appear discontinuously, then loss-vs-compute extrapolation tells you very little about *which capabilities* you'll unlock. It also fed an AI-safety narrative — unpredictable capability jumps are exactly the dynamic safety researchers worry about.

## 5. Schaeffer et al. 2023: The Mirage Argument

Rylan Schaeffer, Brando Miranda, and Sanmi Koyejo's NeurIPS 2023 Outstanding Paper "Are Emergent Abilities of Large Language Models a Mirage?" pushed back, arguing that most of the apparent emergence is an artifact of metric choice rather than a real discontinuity in model behaviour [T1] Schaeffer et al. 2023, https://arxiv.org/abs/2304.15004, abstract.

The core mathematical move is simple. Suppose per-token cross-entropy loss falls smoothly as a power law: `ℒ_CE(N) ∝ N^(-α)`. Then the per-token correctness probability `p(N)` rises smoothly toward 1. But many benchmarks score with **exact-match accuracy on multi-token outputs** — you get the question right only if *all* `L` output tokens are right. Under (rough) independence,

`Accuracy(N) ≈ p(N)^L`

A smooth, continuous improvement in `p(N)` produces an *apparent* sharp transition in `Accuracy(N)` once `p(N)` crosses some threshold, simply because exponentiating by `L` is highly nonlinear. The same thing happens with discontinuous metrics like *Multiple Choice Grade* (you get full credit only for the top choice) [T1] Schaeffer et al. 2023, §3.

They make and confirm three predictions:

1. **Switch to a continuous metric and emergence vanishes.** Re-evaluating the same GPT-3/InstructGPT outputs on integer arithmetic with token edit distance instead of exact match shows continuous, predictable improvement, not a sharp jump [T1] Schaeffer et al. 2023, §4.
2. **Improve the statistics and emergence vanishes.** With more test samples, smaller models are seen to be slightly above chance on accuracy metrics — they were never at zero, just below the resolution of the test set [T1] Schaeffer et al. 2023, §4.
3. **The metric effects are predictable.** Accuracy decays geometrically with sequence length; edit distance decays roughly linearly. Both are derivable from the smooth per-token improvement.

The empirical heart is a meta-analysis of BIG-bench: over 92% of claimed emergent abilities show up *only* under "Multiple Choice Grade" or "Exact String Match", the two metrics they identify as discontinuous/nonlinear. Switch to Brier score on the same LaMDA outputs and the emergence disappears [T1] Schaeffer et al. 2023, §5. They go further and *artificially induce* "emergent" abilities in vision models (LeNet, autoencoders, ViTs) on tasks where emergence had never been claimed, just by choosing nonlinear metrics [T1] Schaeffer et al. 2023, §6.

The careful reading isn't "emergent abilities don't exist"; it's "much of what's been called emergence is metric-induced, and the genuinely scale-dependent claims need to be made on continuous metrics with adequate test resolution." Wei has not, as of the public materials, written a formal rebuttal; his blog post pre-dates Schaeffer and is silent on it [T3] Wei, https://www.jasonwei.net/blog/emergence.

A reasonable synthesis: smooth power-law improvement in cross-entropy is real and well-established; *whether you call the downstream capability change "emergent"* depends on whether you score with a metric that respects the smoothness or one that obliterates it.

## 6. Inverse Scaling and U-Shapes

A complementary line of work asked: are there tasks where bigger models do *worse*? The Inverse Scaling Prize (McKenzie et al. 2022, two rounds, June–October 2022) crowdsourced exactly this and surfaced 11 inverse-scaling tasks across models up to 280B parameters and 500 zettaFLOPs of training compute [T1] Wei, Kim, Tay, Le 2022, "Inverse scaling can become U-shaped", https://arxiv.org/abs/2211.02011, §2. Examples included tasks where models pattern-match a deceptive surface cue (e.g. "redefine `π` to mean 462") and confidently follow it more strongly with scale.

The follow-up "Inverse Scaling Can Become U-Shaped" (Wei, Kim, Tay, Le 2022) re-evaluated those 11 tasks on PaLM up to 540B parameters — about 5× more compute than the prize had access to — and found that **only 4 of 11 remained inverse-scaling**; six showed a U-shape: performance degrades as you scale up, then recovers and surpasses small-model performance at very large scale [T1] Wei et al. 2022 (U-shape), §3. They attribute the U-shape to distractor sub-tasks that only sufficiently large models learn to ignore, and show that 1-shot examples and chain-of-thought prompting can shift several tasks from inverse to positive scaling.

Inverse scaling and U-shapes complicate the simple "more is better" story without overturning it: the *pretraining loss* keeps falling with scale, but downstream task scores can move non-monotonically, especially on tasks that punish over-confident pattern matching.

## 7. BIG-bench: The Benchmark That Made the Debate Possible

Both the emergence and inverse-scaling literatures lean heavily on BIG-bench (Srivastava et al. 2022). It's worth flagging it as infrastructure: 204 tasks contributed by 450 authors across 132 institutions, deliberately drawn from "linguistics, childhood development, math, common-sense reasoning, biology, physics, social bias, software development" — i.e. things current models can't yet reliably do [T1] Srivastava et al. 2022, https://arxiv.org/abs/2206.04615, abstract. Without that diversity-by-design, the discontinuity arguments would have nothing to chew on; almost any individual benchmark is too narrow to reveal scale-dependent qualitative shifts.

A practical spinoff is BIG-Bench Hard (Suzgun et al. 2022), a 23-task subset where prior best models were below average human performance — used as the standard "still-hard" reasoning suite for chain-of-thought and tool-use evaluations [T2] Suzgun et al., BIG-Bench Hard repository, https://github.com/suzgunmirac/BIG-Bench-Hard.

## 8. Where This Leaves the Field

The 2026 consensus, roughly:

- **Pretraining loss vs. compute is a power law.** Kaplan got the functional form right. The exponents are not physical constants but they're stable enough across architectures and datasets to be useful for budget planning.
- **Compute-optimal training scales parameters and tokens together.** Chinchilla's 1:1 recipe (roughly 20 tokens per parameter at the compute-optimal frontier) replaced Kaplan's parameter-heavy recipe. For *inference*-optimal training — where you want a small model that's cheap to serve — you train past Chinchilla-optimal, which is what Llama and most open models do.
- **The data wall is now visible.** Hoffmann et al. assumed unlimited high-quality tokens; multiple 2024 follow-ups (e.g. Villalobos et al., "Will We Run Out of Data?") argue that high-quality public English text is bounded and that the Chinchilla recipe will hit a ceiling well before compute does. This has driven the post-2023 turn toward synthetic data, multi-epoch training, and RL-from-feedback as ways to keep extracting capability per FLOP.
- **"Emergent abilities" is a useful description, not a separate physical phenomenon.** Schaeffer et al.'s metric-discontinuity argument is widely accepted as the explanation for most of the dramatic-looking jumps. Genuinely scale-dependent capabilities — chain-of-thought working at 100B+ but not at 10B — exist but look much smoother on continuous metrics.
- **Loss is not capability.** Predicting downstream task scores from loss remains hard. This is the open frontier of scaling-law work in 2024–2026: predicting *what models can do* from *how well they predict the next token*, which loss-only laws fundamentally cannot tell you.

The arc from Kaplan 2020 to Schaeffer 2023 is, in retrospect, the field learning to be careful with its own empirical claims — first about recipes, then about metrics. The results that survive are the ones that hold up under both more data and harder statistics.

## Sources

- [T1] Kaplan, Jared; McCandlish, Sam; Henighan, Tom; Brown, Tom B.; Chess, Benjamin; Child, Rewon; Gray, Scott; Radford, Alec; Wu, Jeffrey; Amodei, Dario. "Scaling Laws for Neural Language Models." arXiv:2001.08361, 23 January 2020. https://arxiv.org/abs/2001.08361 — peer-reviewed-style technical report from OpenAI, widely cited primary source.
- [T1] Hoffmann, Jordan et al. (DeepMind). "Training Compute-Optimal Large Language Models" (Chinchilla). arXiv:2203.15556, 29 March 2022; published in NeurIPS 2022. https://arxiv.org/abs/2203.15556 and https://proceedings.neurips.cc/paper_files/paper/2022/file/c1e2faff6f588870935f114ebe04a3e5-Paper-Conference.pdf
- [T1] Wei, Jason; Tay, Yi; Bommasani, Rishi; Raffel, Colin; Zoph, Barret; Borgeaud, Sebastian; Yogatama, Dani; Bosma, Maarten; Zhou, Denny; Metzler, Donald; Chi, Ed H.; Hashimoto, Tatsunori; Vinyals, Oriol; Liang, Percy; Dean, Jeff; Fedus, William. "Emergent Abilities of Large Language Models." arXiv:2206.07682, 15 June 2022; published in TMLR 2022. https://arxiv.org/abs/2206.07682
- [T1] Schaeffer, Rylan; Miranda, Brando; Koyejo, Sanmi. "Are Emergent Abilities of Large Language Models a Mirage?" arXiv:2304.15004, 28 April 2023; NeurIPS 2023 Outstanding Paper. https://arxiv.org/abs/2304.15004 and https://proceedings.neurips.cc/paper_files/paper/2023/file/adc98a266f45005c403b8311ca7e8bd7-Paper-Conference.pdf
- [T1] Srivastava, Aarohi et al. (450 co-authors). "Beyond the Imitation Game: Quantifying and extrapolating the capabilities of language models" (BIG-bench). arXiv:2206.04615, June 2022; published in TMLR 2023. https://arxiv.org/abs/2206.04615
- [T1] Wei, Jason; Kim, Najoung; Tay, Yi; Le, Quoc V. "Inverse scaling can become U-shaped." arXiv:2211.02011, 3 November 2022; EMNLP 2023. https://arxiv.org/abs/2211.02011 and https://aclanthology.org/2023.emnlp-main.963/
- [T1] Besiroglu, Tamay; Erdil, Ege; Barnett, Matthew; You, Josh. "Chinchilla Scaling: A replication attempt." arXiv:2404.10102, 15 April 2024; Epoch AI. https://arxiv.org/abs/2404.10102 and https://epoch.ai/blog/chinchilla-scaling-a-replication-attempt
- [T2] Suzgun, Mirac et al. "BIG-Bench Hard" (challenging subset of BIG-bench, 23 tasks). 2022. https://github.com/suzgunmirac/BIG-Bench-Hard
- [T2] Brenndoerfer, Michael. "Scaling Laws for Neural Language Models: Predicting Performance from Scale." Long-form explainer, 2024. https://mbrenndoerfer.com/writing/scaling-laws-neural-language-models-power-law-predictions
- [T2] Brenndoerfer, Michael. "Chinchilla Scaling Laws: Compute-Optimal Training and Resource Allocation for Large Language Models." Long-form explainer, 2024. https://mbrenndoerfer.com/writing/chinchilla-scaling-laws-compute-optimal-training-resource-allocation
- [T2] Thompson, Alan D. "Chinchilla data-optimal scaling laws: in plain English." Life Architect AI, 2022. https://lifearchitect.ai/chinchilla/
- [T3] Wei, Jason. "137 emergent abilities of large language models." Personal blog, 14 November 2022. https://www.jasonwei.net/blog/emergence
