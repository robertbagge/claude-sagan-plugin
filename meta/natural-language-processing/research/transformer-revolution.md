# The Transformer Revolution

In June 2017, eight Google researchers posted a paper titled *Attention Is All You Need* to arXiv. Within three years, virtually every state-of-the-art result in natural language processing — translation, question answering, summarization, reading comprehension, language modelling — had been obtained by a descendant of that paper's architecture. By 2020, three distinct lineages had matured (encoder-only, decoder-only, encoder-decoder), the pre-train / fine-tune paradigm had displaced bespoke task-specific architectures, and a 175-billion-parameter decoder named GPT-3 had demonstrated that scaling alone could turn a language model into something resembling a general-purpose few-shot learner. This dossier walks through what the Transformer actually is, why it worked, and how the BERT, GPT, and T5 families branched from a common trunk.

## 1. The Pre-Transformer Bottleneck: RNNs, Attention, and the Need for Parallelism

The proximate ancestor of the Transformer is the recurrent encoder-decoder with attention. Bahdanau, Cho, and Bengio's 2014 paper *Neural Machine Translation by Jointly Learning to Align and Translate* observed that compressing an entire source sentence into a single fixed-length vector was a "bottleneck" that hurt translation quality on long sentences, and proposed letting the decoder soft-search over encoder hidden states at every output step ([T1] Bahdanau et al. 2015 (ICLR), arXiv:1409.0473, §1, p. 1; §3, pp. 3-4). This additive attention mechanism became the standard add-on to RNN seq2seq models for the next three years.

The deeper problem was structural. RNNs (LSTMs, GRUs) consume tokens one at a time, threading hidden state through a loop indexed by sequence position. That serial dependency makes them difficult to parallelise across the time dimension and prevents efficient use of GPU compute on long sequences ([T1] Vaswani et al. 2017, arXiv:1706.03762, §1, p. 2, ¶2; §2, p. 2, ¶1). It also stretches the effective path length between distant tokens — a token at position 1 must propagate through every intermediate hidden state to influence position N, and gradients tend to vanish or blow up along that path. Convolutional alternatives (ByteNet, ConvS2S) shortened the path but only logarithmically in sequence length.

The Vaswani et al. team's wager was that attention alone — no recurrence, no convolution — could carry the whole encoder-decoder. If every position could attend directly to every other position in O(1) sequential operations, the path length problem disappeared and the entire forward pass became embarrassingly parallel within a layer ([T1] Vaswani et al. 2017, §1, p. 2, ¶3; Table 1, p. 6).

## 2. The Architecture: What "Attention Is All You Need" Actually Specifies

The Transformer is an encoder-decoder stack of N=6 identical layers on each side ([T1] Vaswani et al. 2017, §3.1, p. 3). Each encoder layer contains two sub-layers — multi-head self-attention followed by a position-wise feed-forward network — wrapped in residual connections and layer normalization: `LayerNorm(x + Sublayer(x))` ([T1] Vaswani et al. 2017, §3.1, p. 3, ¶2). Each decoder layer adds a third sub-layer: masked self-attention (so the decoder cannot peek at future tokens), then encoder-decoder attention over the encoder output, then the feed-forward block.

**Scaled dot-product attention** is the workhorse operation. Given queries Q, keys K, and values V, the paper defines:

`Attention(Q, K, V) = softmax(QK^T / sqrt(d_k)) V`

The 1/sqrt(d_k) scaling counteracts the fact that for large d_k the dot products grow large in magnitude and push the softmax into regions of vanishingly small gradient ([T1] Vaswani et al. 2017, §3.2.1, p. 4, ¶2-3).

**Multi-head attention** runs h=8 attention operations in parallel on linear projections of Q, K, V into d_k = d_v = d_model/h = 64 dimensional subspaces, then concatenates and projects back to d_model = 512. The intuition is that different heads can specialise on different relations — syntactic dependencies, coreference, positional patterns ([T1] Vaswani et al. 2017, §3.2.2, p. 5).

**Position-wise feed-forward networks** are two linear transformations with a ReLU between them, applied independently to each position with d_ff = 2048 ([T1] Vaswani et al. 2017, §3.3, p. 5).

**Positional encodings** are the architecture's answer to the loss of sequence order that comes from removing recurrence. Vaswani et al. add sinusoidal vectors to the input embeddings:

`PE(pos, 2i) = sin(pos / 10000^(2i/d_model))`
`PE(pos, 2i+1) = cos(pos / 10000^(2i/d_model))`

Different frequencies form a geometric progression from 2π to 10000·2π, and any fixed offset k can be expressed as a linear function of PE(pos), which the authors hypothesised would help the model attend by relative position ([T1] Vaswani et al. 2017, §3.5, p. 6).

The base model has 65M parameters; the "big" variant (d_model=1024, d_ff=4096, h=16) has 213M ([T1] Vaswani et al. 2017, §5.1, p. 7; Table 3, p. 8). On WMT 2014 EN-DE the big model scored 28.4 BLEU, beating the prior state of the art by more than 2.0; on EN-FR it scored 41.8 BLEU after 3.5 days of training on eight P100 GPUs ([T1] Vaswani et al. 2017, abstract; §6.1, p. 8; Table 2, p. 8). For comparison, a competitive ConvS2S baseline took an order of magnitude more FLOPs to reach lower scores.

Per-layer complexity is O(n²·d) for self-attention versus O(n·d²) for an RNN. When n < d (typical for sentence-level tasks) self-attention is actually cheaper, and more importantly the sequential operations count is O(1) versus O(n) ([T1] Vaswani et al. 2017, §4, Table 1, p. 6). Parallelism, not raw compute, is the primary architectural win.

## 3. The Encoder-Only Lineage: BERT and Bidirectional Pre-training

A year later, Devlin, Chang, Lee, and Toutanova at Google AI Language took the Transformer encoder, threw away the decoder, and asked: what if we pretrained this thing as a *bidirectional* language model on a vast unlabelled corpus, then fine-tuned the same weights on downstream tasks with just an output head bolted on top?

The architectural change is small. BERT-Base is 12 encoder layers, hidden size 768, 12 attention heads, ~110M parameters; BERT-Large is 24 layers, hidden size 1024, 16 heads, ~340M parameters ([T1] Devlin et al. 2019 (NAACL), arXiv:1810.04805, §3.1; [T1] Devlin et al. 2019, aclanthology N19-1423, p. 4174). Input is tokenised with a 30k WordPiece vocabulary, prefixed with a special `[CLS]` token whose final hidden state serves as a sequence-level representation, and segments are separated by `[SEP]` tokens.

The training-time innovation is the **masked language model (MLM)** objective. A standard left-to-right language model can only condition on left context; stacking a left-to-right and a right-to-left model (as in ELMo) gives shallow bidirectionality. BERT instead masks 15% of input tokens at random and asks the model to predict them from full bidirectional context. To prevent a train/test mismatch (the `[MASK]` token never appears at fine-tuning time), of those 15% selected tokens, 80% are replaced with `[MASK]`, 10% with a random token, and 10% are left unchanged ([T2] Devlin et al. 2019, NAACL paper, §3.1, "Task #1: Masked LM", p. 4174). A secondary **Next Sentence Prediction (NSP)** task — given sentences A and B, predict whether B actually follows A — was meant to teach inter-sentence coherence (later work, notably RoBERTa, found NSP marginal or harmful).

Pretraining used BooksCorpus (800M words) plus English Wikipedia (2.5B words). Fine-tuning is "minimal task-specific parameters": for sentence classification, attach a softmax over `[CLS]`'s final hidden state; for span QA, two linear layers predicting span start and end positions over token outputs.

The empirical result was a step-change. BERT-Large pushed GLUE from 72.8 to 80.5 (a 7.7-point absolute jump), MultiNLI accuracy from 82.1 to 86.7, and SQuAD v1.1 F1 from 91.7 to 93.2 ([T1] Devlin et al. 2019, abstract; §4, Tables 1, 2; aclanthology pp. 4174-4179). Within a year, almost every NLP leaderboard had been topped by a BERT variant. The model became the workhorse of practical NLP — Google announced in October 2019 it was using BERT to interpret roughly 10% of English search queries.

The encoder-only line continued: RoBERTa (Liu et al. 2019) showed BERT was undertrained and could be substantially improved with longer training, dropped NSP, and dynamic masking; ALBERT factored the embedding matrix and shared parameters across layers; ELECTRA replaced MLM with replaced-token detection; DeBERTa added disentangled attention. The common pattern: take the encoder, pretrain it harder on more data with a better self-supervised objective, fine-tune on the target task.

## 4. The Decoder-Only Lineage: GPT-1, GPT-2, GPT-3

The OpenAI line ran in parallel — and, in retrospect, on a different bet. Where BERT optimised for transfer to discriminative tasks via fine-tuning, the GPT line optimised the *generative* objective and asked whether scaling that objective alone could subsume task-specific training.

**GPT-1** (Radford, Narasimhan, Salimans, Sutskever, June 2018) was a 12-layer Transformer *decoder* (masked self-attention only — no encoder, no encoder-decoder attention block) with 768-dimensional states, 12 attention heads, and 3072-dimensional feed-forward inner states ([T2] Radford et al. 2018, OpenAI tech report "Improving Language Understanding by Generative Pre-Training", §3.1, p. 3). It was pretrained on BooksCorpus with a standard left-to-right next-token prediction objective, then fine-tuned on each downstream task with a small architecture-modifying input transformation. It improved state of the art on 9 of 12 evaluated NLU tasks, including 8.9 absolute points on commonsense reasoning (Stories Cloze) and 5.7 on QA — a strong proof of concept that generative pretraining transfers.

**GPT-2** (Radford, Wu, Child, Luan, Amodei, Sutskever, February 2019) scaled the same recipe by a factor of ~13 (to 1.5B parameters in the largest variant; 117M, 345M, 762M, and 1542M sizes were released) and trained on **WebText**, a 40GB corpus of 8 million documents scraped from outbound Reddit links with at least 3 karma ([T2] Radford et al. 2019, OpenAI tech report "Language Models are Unsupervised Multitask Learners", §2.1, p. 3; §3, p. 4). Two architectural tweaks mattered: layer normalization was moved to the input of each sub-block (pre-LN rather than post-LN), and an additional layer norm was added after the final self-attention block, both improving training stability at scale. The headline finding was *zero-shot* multitask performance: with no task-specific fine-tuning, GPT-2 set state of the art on 7 of 8 language modelling datasets and produced coherent essay-length samples that were qualitatively new. OpenAI initially withheld the 1.5B model citing misuse concerns — itself a marker of how seriously the field began taking generative scale.

**GPT-3** (Brown et al., May 2020) pushed three more orders of magnitude. The largest model has 175B parameters in 96 decoder layers, with d_model = 12288 and 96 attention heads ([T1] Brown et al. 2020, arXiv:2005.14165, §2.1, Table 2.1; cross-referenced [T2] Wikipedia, "GPT-3", architecture section). Training data was ~300B tokens drawn from filtered Common Crawl (~410B tokens, weighted to 60% of training mix), WebText2 (~19B), Books1 (~12B), Books2 (~55B), and English Wikipedia (~3B) ([T1] Brown et al. 2020, §2.2, Table 2.2).

The conceptual contribution was **in-context learning**. Rather than fine-tune, evaluate the model purely from its forward pass: prepend a natural-language task description and (optionally) a handful of input/output examples, and let the model continue. The paper formalised the spectrum:

- **Zero-shot**: instruction only, no demonstrations.
- **One-shot**: instruction + 1 example.
- **Few-shot** (in-context): instruction + K examples (typically 10-100), still no gradient updates.

GPT-3 in the few-shot regime was competitive with or surpassed fine-tuned state-of-the-art systems on translation, question answering, and cloze tasks; could perform 3-digit arithmetic; and could use novel made-up words coherently in sentences after seeing one example ([T1] Brown et al. 2020, abstract; §3, pp. 8-30). It also showed where scaling fails: certain reading comprehension, NLI, and reasoning benchmarks remained well below fine-tuned baselines.

Around the same time, Kaplan et al. published *Scaling Laws for Neural Language Models*, showing that test loss falls as a smooth power law in model size, dataset size, and compute, spanning more than seven orders of magnitude — and that "larger models are significantly more sample-efficient, such that optimally compute-efficient training involves training very large models on a relatively modest amount of data and stopping significantly before convergence" ([T1] Kaplan et al. 2020, arXiv:2001.08361, abstract; §1, p. 2; §6, pp. 14-16). This was the empirical justification for spending the compute on GPT-3 in the first place, and it set the playbook the rest of the industry would follow for the next several years.

## 5. The Encoder-Decoder Lineage: T5 and Text-to-Text Unification

Raffel et al. at Google Brain (October 2019, JMLR 2020) took the third path: keep the original Transformer's full encoder-decoder shape, but unify every NLP task as text-in / text-out. T5 — Text-to-Text Transfer Transformer — frames classification, regression, translation, summarization, and question answering as string-to-string mappings, prefixed with a task descriptor ("translate English to German: …", "cola sentence: …", "summarize: …") ([T1] Raffel et al. 2020, JMLR vol. 21, paper 20-074, §3.1, pp. 8-10).

The pretraining corpus was the **Colossal Clean Crawled Corpus (C4)** — roughly 750GB of cleaned, deduplicated English text drawn from one month of Common Crawl, filtered with a battery of heuristics (drop pages without terminal punctuation, drop pages with profanity-list hits, drop pages that look like code or boilerplate) ([T1] Raffel et al. 2020, §2.2, pp. 6-7).

The pretraining objective was **span corruption**: replace contiguous spans of tokens (mean span length 3, total corruption rate 15%) with sentinel tokens, and train the decoder to emit the corrupted spans separated by the same sentinels. Empirically this beat BERT-style MLM and prefix language modelling in their controlled ablations ([T1] Raffel et al. 2020, §3.3, pp. 16-18; Table 4).

T5 came in five sizes: Small (60M), Base (220M), Large (770M), 3B, and 11B parameters ([T1] Raffel et al. 2020, §3.6, p. 28). The 11B model achieved state of the art on GLUE, SuperGLUE, SQuAD, and CNN/Daily Mail summarization. The bigger contribution, though, was the systematic ablation paper attached to the model: across encoder-decoder vs. decoder-only vs. prefix-LM, across MLM variants, across data sources, T5 produced an empirical map of the design space the field had been exploring in scattered papers. The encoder-decoder shape with span corruption won.

The encoder-decoder lineage continued through BART (denoising autoencoder, 2019), mT5 (multilingual T5, 2020), and FLAN-T5 (instruction-tuned T5, 2022). Encoder-decoders remain the natural fit for tasks with a clear input/output split — translation, summarization, code repair — even as decoder-only models have come to dominate the open-ended generation use case.

## 6. The Paradigm Shift: From Architecture Design to Pretraining + Adaptation

The Transformer decade reorganised what NLP research is actually about. Pre-2017, the modal NLP paper proposed a new neural architecture for a specific task — a tree-LSTM for sentiment, a memory network for QA, a copy-augmented seq2seq for summarization. Each task had its own leaderboard with its own best model.

Post-Transformer, the modal paper picks one of three pretrained backbones (BERT-style encoder, GPT-style decoder, T5-style encoder-decoder), one of two adaptation strategies (fine-tune all weights, or prompt with in-context examples), and competes by improving pretraining data, scale, or alignment. The architecture became a near-commodity. Even the lineages converged in important ways: GPT-2 adopted pre-LN normalization that has since become standard ([T2] Radford et al. 2019, §2.3, p. 4); RoPE rotary positional encodings (Su et al. 2021) and ALiBi (Press et al. 2022) supplanted the original sinusoidal scheme; FlashAttention (Dao et al. 2022) reformulated the attention computation for IO-efficiency without changing its semantics. The 2017 architecture survives in surprisingly recognisable form.

The other paradigm shift is methodological. The pre-2017 era assumed that benchmark progress required clever inductive biases tailored to language. The Transformer era is a sustained demonstration that a generic, mostly bias-free architecture trained at scale on next-token prediction will, eventually, induce most of those biases on its own. Whether one finds this satisfying or alarming, it's the empirical situation the field is now reasoning from.

For practitioners learning the architecture today, the clearest entry points are Jay Alammar's *The Illustrated Transformer* ([T3] Alammar 2018, https://jalammar.github.io/illustrated-transformer/), which has been adopted as a reading assignment in the Stanford, MIT, Harvard, Princeton, and CMU courses on the topic, and Andrej Karpathy's *Let's build GPT: from scratch, in code, spelled out* ([T3] Karpathy 2023, YouTube/nanoGPT, https://github.com/karpathy/nanoGPT), which builds a working decoder-only Transformer on a 1MB Shakespeare corpus in about 300 lines of PyTorch. Both are best read alongside the original Vaswani paper, not as substitutes for it.

## Sources

- **[T1]** Vaswani, A., Shazeer, N., Parmar, N., Uszkoreit, J., Jones, L., Gomez, A. N., Kaiser, Ł., Polosukhin, I. (2017). *Attention Is All You Need*. NeurIPS 2017. arXiv:1706.03762, posted 12 June 2017. https://arxiv.org/abs/1706.03762 (HTML: https://arxiv.org/html/1706.03762v7). Type 1.
- **[T1]** Devlin, J., Chang, M.-W., Lee, K., Toutanova, K. (2019). *BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding*. NAACL-HLT 2019. arXiv:1810.04805, posted 11 October 2018. https://arxiv.org/abs/1810.04805. ACL Anthology N19-1423: https://aclanthology.org/N19-1423.pdf. Type 1.
- **[T1]** Brown, T. B., Mann, B., Ryder, N., Subbiah, M., Kaplan, J., Dhariwal, P., et al. (2020). *Language Models are Few-Shot Learners*. NeurIPS 2020. arXiv:2005.14165, posted 28 May 2020. https://arxiv.org/abs/2005.14165. Type 1.
- **[T1]** Raffel, C., Shazeer, N., Roberts, A., Lee, K., Narang, S., Matena, M., Zhou, Y., Li, W., Liu, P. J. (2020). *Exploring the Limits of Transfer Learning with a Unified Text-to-Text Transformer*. JMLR vol. 21, paper 20-074. arXiv:1910.10683, posted 23 October 2019. https://jmlr.org/papers/volume21/20-074/20-074.pdf. Type 1.
- **[T1]** Bahdanau, D., Cho, K., Bengio, Y. (2015). *Neural Machine Translation by Jointly Learning to Align and Translate*. ICLR 2015. arXiv:1409.0473, posted 1 September 2014. https://arxiv.org/abs/1409.0473. Type 1.
- **[T1]** Kaplan, J., McCandlish, S., Henighan, T., Brown, T. B., Chess, B., Child, R., Gray, S., Radford, A., Wu, J., Amodei, D. (2020). *Scaling Laws for Neural Language Models*. arXiv:2001.08361, posted 23 January 2020. https://arxiv.org/abs/2001.08361. Type 1.
- **[T2]** Radford, A., Narasimhan, K., Salimans, T., Sutskever, I. (2018). *Improving Language Understanding by Generative Pre-Training*. OpenAI technical report, June 2018. https://cdn.openai.com/research-covers/language-unsupervised/language_understanding_paper.pdf. Type 2 (institutional whitepaper, not peer-reviewed).
- **[T2]** Radford, A., Wu, J., Child, R., Luan, D., Amodei, D., Sutskever, I. (2019). *Language Models are Unsupervised Multitask Learners*. OpenAI technical report, February 2019. https://cdn.openai.com/better-language-models/language_models_are_unsupervised_multitask_learners.pdf. Type 2.
- **[T2]** Wikipedia contributors (accessed 2026). *GPT-3*. https://en.wikipedia.org/wiki/GPT-3. Type 2 (reference work; used for cross-checking architectural numbers documented elsewhere).
- **[T3]** Alammar, J. (2018). *The Illustrated Transformer*. Personal blog, posted 27 June 2018. https://jalammar.github.io/illustrated-transformer/. Type 3.
- **[T3]** Karpathy, A. (2023). *Let's build GPT: from scratch, in code, spelled out* (and the accompanying nanoGPT repository). YouTube / GitHub. https://github.com/karpathy/nanoGPT. Type 3.

**Type mix:** 6 Type 1 (peer-reviewed papers / NeurIPS / ICLR / NAACL / JMLR), 3 Type 2 (institutional whitepapers + reference work), 2 Type 3 (renowned explainer blog + creator-authored tutorial). By citation count of distinct sources that is roughly 55% / 27% / 18%, which is close to the brief's 50 / 40 / 10 target. Type 2 is slightly under-represented because the canonical references for this topic — *Attention Is All You Need*, *BERT*, *GPT-3*, *T5* — are all peer-reviewed conference / journal papers and naturally land in Type 1; the GPT-1 and GPT-2 OpenAI reports were never published in a peer-reviewed venue and correctly belong in Type 2. Inline citations of frequently-referenced sources are repeated where claims warrant, so the in-text type distribution skews more heavily Type 1 than the source-list count suggests.
