# Sequence Models: From Vanilla RNNs to Encoder-Decoder Attention (1990–2017)

Between roughly 2014 and 2017 a small family of recurrent architectures — LSTM, GRU, the seq2seq encoder-decoder, and additive/multiplicative attention — dethroned a quarter-century of hand-engineered statistical NLP and became the production substrate of Google Translate, Skype Translator, smart-reply systems, and the first wave of neural dialogue agents. This dossier traces that arc, sketches the mathematics, and explains why the very feature that made these models powerful — sequential, time-recursive computation — is also what doomed them once GPUs grew big enough to make the Transformer practical.

## 1. The Vanishing Gradient Problem and Why Vanilla RNNs Failed

A vanilla (Elman) RNN updates a single hidden state h_t = tanh(W_h h_{t-1} + W_x x_t + b) and is trained by Backpropagation Through Time (BPTT). The trouble is structural. Bengio, Simard and Frasconi proved in 1994 that "learning long-term dependencies with gradient descent is difficult": when the spectral radius of the recurrent Jacobian is less than 1, gradients shrink geometrically toward zero as they are propagated back across time steps, and when it is greater than 1 they explode ([T1] Bengio, Simard & Frasconi 1994, IEEE TNN 5(2):157–166, §3). The paper formalised what practitioners had been running into empirically — a network that in principle has unbounded memory cannot, in practice, learn to use it.

Pascanu, Mikolov and Bengio revisited the problem nearly two decades later and gave the canonical modern treatment, showing that gradient norms can be bounded above by products of Jacobian norms across time, proposing gradient-norm clipping for the exploding case and a soft regularisation constraint for the vanishing case ([T1] Pascanu, Mikolov & Bengio 2013, ICML, p. 1, §1; gradient clipping, §3). Their analysis is the reason that, even today, every serious recurrent training loop ships with `clip_grad_norm_`.

Two architectural responses dominated. The first, gating, ultimately won: rather than fight the multiplicative chain of Jacobians, replace it with an additive update path that gradients can flow through unimpeded.

## 2. LSTM: Constant Error Carousels and Multiplicative Gates

Hochreiter and Schmidhuber published Long Short-Term Memory in Neural Computation in 1997, presenting what they called a "novel, efficient, gradient-based method" to bridge "minimal time lags in excess of 1000 discrete-time steps" ([T1] Hochreiter & Schmidhuber 1997, Neural Computation 9(8):1735–1780, abstract). The central idea is the **Constant Error Carousel (CEC)**: a memory cell whose recurrence is the linear identity, c_t = c_{t-1} + (writes), so the local gradient through the cell is exactly 1 and error can be carried backward across arbitrarily many steps without decay ([T2] Hochreiter & Schmidhuber 1997, §3, "Constant Error Backprop").

Around the CEC sit two multiplicative gates: an **input gate** that decides what new content is written into the cell, and an **output gate** that decides what content is exposed to the rest of the network ([T2] Hochreiter & Schmidhuber 1997, §4). Note that the original 1997 paper has no forget gate — the cell only ever accumulates. Gers, Schmidhuber and Cummins added the forget gate in 2000 ("Learning to Forget"), and that three-gate variant is what almost every modern textbook draws as "the LSTM" ([T2] Jurafsky & Martin, Speech and Language Processing 3rd ed., ch. 9, "RNNs and LSTMs"). The full modern equations, with σ the logistic sigmoid and ⊙ elementwise product:

```
i_t = σ(W_i x_t + U_i h_{t-1} + b_i)        # input gate
f_t = σ(W_f x_t + U_f h_{t-1} + b_f)        # forget gate
o_t = σ(W_o x_t + U_o h_{t-1} + b_o)        # output gate
ĉ_t = tanh(W_c x_t + U_c h_{t-1} + b_c)     # candidate cell
c_t = f_t ⊙ c_{t-1} + i_t ⊙ ĉ_t             # cell state (the CEC)
h_t = o_t ⊙ tanh(c_t)                       # hidden state
```

The c_t equation is the load-bearing line. Its additive structure plus sigmoid gates near 1 lets information persist; gates near 0 let it be cleanly erased. Hochreiter and Schmidhuber demonstrated empirically that LSTM solved synthetic benchmarks that vanilla RNNs and other contemporary methods (BPTT, RTRL, Elman nets) failed to crack at all ([T1] Hochreiter & Schmidhuber 1997, Neural Computation 9(8):1735–1780, experiments §5). For about fifteen years almost no one used the architecture; then language modelling, machine translation and speech recognition benchmarks fell to it in succession between 2012 and 2015.

## 3. GRU: A Simpler Gating Story

In June 2014 Cho et al. introduced the **Gated Recurrent Unit** as a side-product of their RNN encoder-decoder paper for statistical machine translation ([T1] Cho, van Merriënboer, Gulcehre, Bahdanau, Bougares, Schwenk & Bengio 2014, EMNLP, arXiv:1406.1078, §2.3). The GRU collapses LSTM's input and forget gates into a single **update gate** and drops the output gate and separate cell state entirely:

```
r_t = σ(W_r x_t + U_r h_{t-1} + b_r)             # reset gate
z_t = σ(W_z x_t + U_z h_{t-1} + b_z)             # update gate
ĥ_t = tanh(W_h x_t + U_h (r_t ⊙ h_{t-1}) + b_h)  # candidate
h_t = (1 - z_t) ⊙ h_{t-1} + z_t ⊙ ĥ_t            # interpolation
```

([T2] Wikipedia, "Gated recurrent unit", "Fully gated unit" subsection, retrieved 2026; matches the formulation in [T1] Cho et al. 2014, §2.3.) The reset gate r_t lets the unit ignore its past when computing the candidate; the update gate z_t linearly interpolates between the old hidden state and the new candidate — again, an additive-style path that protects the gradient.

Chung, Gulcehre, Cho and Bengio's empirical comparison later that year on polyphonic music and speech-signal tasks found that GRUs and LSTMs are roughly comparable, both substantially better than tanh RNNs, with no clear winner across tasks ([T1] Chung, Gulcehre, Cho & Bengio 2014, "Empirical Evaluation of Gated Recurrent Neural Networks on Sequence Modeling", arXiv:1412.3555, §4). For the next three years authors picked the cell that was cheaper for their hardware budget: GRUs (3 weight matrices vs LSTM's 4) became popular in shorter-sequence settings; LSTMs remained the default for translation and large language models.

## 4. Seq2Seq: A Single Vector Tries to Hold a Whole Sentence

Sutskever, Vinyals and Le's 2014 NIPS paper "Sequence to Sequence Learning with Neural Networks" was the architectural breakthrough that recast machine translation as a single end-to-end neural problem ([T1] Sutskever, Vinyals & Le 2014, NIPS 27, arXiv:1409.3215, abstract). The recipe: a deep LSTM **encoder** reads the source sentence one token at a time and compresses it into the final hidden state — a single fixed-length vector — and a separate deep LSTM **decoder** is initialised from that vector and emits target tokens autoregressively until it produces an end-of-sentence symbol.

Key design choices:

- **Depth and width.** Four LSTM layers, 1000 hidden units per layer, 1000-dimensional word embeddings, 160k source / 80k target vocabulary, trained on the WMT'14 English-French task ([T1] Sutskever et al. 2014, §3).
- **Source reversal.** Reversing the input word order — but not the target — "improved the LSTM's performance markedly" by introducing many short-term dependencies between matching source/target prefix words and easing optimisation ([T1] Sutskever et al. 2014, §3.3, "Reversing the Source Sentences").
- **Results.** A direct LSTM scored **34.8 BLEU** on WMT'14 EN-FR, beating the phrase-based SMT baseline at 33.3; using the LSTM to rerank the SMT system's 1000-best list pushed BLEU to **36.5** ([T1] Sutskever et al. 2014, §3.4 and Table 1).

Concurrently, Cho et al.'s RNN Encoder-Decoder paper proposed essentially the same architecture trained for phrase scoring inside a Moses-style SMT pipeline ([T1] Cho et al. 2014, arXiv:1406.1078, §1). The two papers together established the encoder-decoder pattern that — minus the recurrence — every modern Transformer-based translation, summarisation and dialogue system still uses.

The catch was visible in the design: the entire source sentence had to fit through a single hidden-state bottleneck. Sutskever et al. partly hid this by reversing inputs and using 1000-dim states, but Cho's group quickly showed that performance "drops rapidly as the length of an input sentence increases" — past about 30 tokens, BLEU collapsed ([T1] Cho, van Merriënboer, Bahdanau & Bengio 2014, "On the Properties of Neural Machine Translation: Encoder-Decoder Approaches", arXiv:1409.1259, §3, Figure 2).

## 5. Attention: Letting the Decoder Look Back

Bahdanau, Cho and Bengio's "Neural Machine Translation by Jointly Learning to Align and Translate" (submitted September 2014, ICLR 2015) attacked the bottleneck head-on: instead of forcing the encoder to summarise the source into one vector, let the decoder dynamically pick a different weighted combination of all encoder hidden states at every output step ([T1] Bahdanau, Cho & Bengio 2014, arXiv:1409.0473, §1; published as ICLR 2015 conference paper).

The encoder is a **bidirectional RNN** producing forward states →h_j and backward states ←h_j; their concatenation h_j = [→h_j; ←h_j] is the "annotation" of source position j. At decoder step i, with previous decoder state s_{i-1}:

```
e_ij  = a(s_{i-1}, h_j)                       # alignment score (small MLP)
α_ij  = exp(e_ij) / Σ_k exp(e_ik)              # softmax over source
c_i   = Σ_j α_ij · h_j                         # context vector
s_i   = f(s_{i-1}, y_{i-1}, c_i)               # decoder update
```

This is **additive (concat) attention**: the alignment function a is a feed-forward network with a tanh non-linearity over [s_{i-1}; h_j] ([T1] Bahdanau et al. 2014, §3.1, Eq. 6 and Appendix A.1.2). The α_ij are interpretable as soft alignments; visualising them recovers the source-target word correspondences that classical statistical MT had to estimate with separate alignment models. On WMT'14 EN-FR, **RNNsearch-50** (attention model trained on sentences up to 50 tokens) reached **28.45 BLEU** and crucially "did not exhibit any performance loss with increasing sentence length" out to 50+ tokens, while the vanilla **RNNencdec-50** baseline degraded sharply past 20 tokens ([T1] Bahdanau et al. 2014, §6 / Fig. 2 and Table 1).

Luong, Pham and Manning's EMNLP 2015 follow-up, "Effective Approaches to Attention-based Neural Machine Translation", refined the mechanism into the form most engineers now teach ([T1] Luong, Pham & Manning 2015, EMNLP, pp. 1412–1421, arXiv:1508.04025). They simplified the decoder pipeline — using the current decoder state h_t (rather than h_{t-1}) to compute attention, then combining context and state in one feed-forward step — and proposed three score functions:

```
score(h_t, h̄_s) = h_t · h̄_s                                   # dot
                = h_t^T W_a h̄_s                                 # general (multiplicative)
                = v_a^T tanh(W_a [h_t; h̄_s])                    # concat (Bahdanau-style)
```

([T1] Luong et al. 2015, §3.1, Eq. 7–10.) They also introduced **local attention**, which predicts an aligned source position p_t and weights only a window of source states around it with a Gaussian, trading a small BLEU cost for cheaper computation on long sequences ([T1] Luong et al. 2015, §3.2). Their global-dot and local-p models pushed WMT'15 EN-DE results to a then-state-of-the-art 25.9 BLEU as an ensemble, a +1.0 BLEU improvement over the previous best NMT-plus-reranker system ([T1] Luong et al. 2015, §5, Table 2).

The shorthand the field settled on: "Bahdanau attention" = additive, decoder state from previous step; "Luong attention" = multiplicative (dot or general), decoder state from current step. Multiplicative attention is the direct ancestor of the Transformer's scaled dot-product attention.

## 6. Why Recurrent Seq2Seq Dominated 2014–2017

For roughly three years, encoder-decoder LSTMs with attention were the architecture that worked across NLP tasks where the output was a sequence:

- **Neural machine translation.** Google announced GNMT in production in September 2016: 8 LSTM encoder layers, 8 LSTM decoder layers, 1024 units per layer, attention from the bottom decoder layer to the top encoder layer, residual connections and wordpiece tokenisation ([T1] Wu et al. 2016, arXiv:1609.08144, §3, model architecture). Side-by-side human evaluation showed translation errors reduced by **55–85%** on major language pairs versus the previous phrase-based system, with Chinese-English the first language pair launched, handling about 18M queries/day ([T2] Google Research Blog, "A Neural Network for Machine Translation, at Production Scale", 27 Sep 2016).
- **Abstractive summarisation.** Rush, Chopra and Weston's 2015 ACL paper on neural attention summarisation, and Nallapati et al.'s 2016 CoNLL paper using full encoder-decoder LSTMs with attention, established the headline-generation benchmarks that every subsequent system was measured on.
- **Dialogue.** Vinyals and Le's "A Neural Conversational Model" (2015) demonstrated that the same seq2seq+attention recipe could carry on coherent IT-helpdesk and movie-dialogue conversations end-to-end without hand-written rules.
- **Image captioning, parsing, code generation.** "Show and Tell" (Vinyals 2015), "Grammar as a Foreign Language" (Vinyals et al. 2015) and many others reused the encoder-decoder template with different encoders.

Karpathy's May 2015 essay "The Unreasonable Effectiveness of Recurrent Neural Networks" did much of the popularisation work. It demonstrated that a vanilla character-level LSTM trained on Shakespeare, Wikipedia, LaTeX algebraic-geometry papers and the Linux kernel source could produce locally plausible text in each domain — including matched brackets, valid markdown structure, and even compilable C-like syntax — and framed RNN training as "optimisation over programs" rather than over functions ([T3] Karpathy, "The Unreasonable Effectiveness of Recurrent Neural Networks", http://karpathy.github.io/2015/05/21/rnn-effectiveness/, 21 May 2015). Karpathy was also explicit about the limitations: char-level models "forget whether it was doing a proof or a lemma" over long contexts and routinely mis-match closing tags.

## 7. The Sequential Bottleneck and the Setup for the Transformer

The technical story of why Transformers were almost inevitable starts here: every recurrent variant — Elman, LSTM, GRU, bi-LSTM — has the same fundamental constraint. Computing h_t requires h_{t-1}, which requires h_{t-2}, and so on. **Time cannot be parallelised within a sequence on the forward or backward pass.** Batch parallelism helps; sequence parallelism does not exist.

By 2016 several converging pressures made this fatal:

1. **GPU economics.** Nvidia's V100 made matrix multiplications cheap and memory bandwidth abundant, but rewarded large-batch dense linear algebra. Sequential LSTM steps left tensor cores idle.
2. **Sequence lengths kept growing.** Document-level summarisation, longer translation contexts, and multi-turn dialogue all wanted contexts of hundreds to thousands of tokens. Recurrent training time scaled linearly with that.
3. **Attention had already taken over the heavy lifting.** Bahdanau- and Luong-style attention was doing most of the modelling work. The recurrence increasingly looked like scaffolding around the attention layer rather than the other way around.
4. **Convolutional alternatives appeared.** Gehring et al.'s ConvS2S (2017) demonstrated that you could match or beat LSTM-based NMT with fully-convolutional encoders and decoders, removing recurrence and gaining substantial speedups — proof of concept that recurrence wasn't doing essential work.

Vaswani et al.'s "Attention Is All You Need" (NeurIPS 2017) closed the loop: drop the RNN entirely, use stacked self-attention plus position embeddings, and parallelise the entire forward and backward pass over the sequence dimension. Recurrent encoder-decoders with attention had quietly demonstrated that the attention mechanism alone was enough; the Transformer was the architecture that finally trusted that demonstration.

## 8. What to Take Away

- The vanishing-gradient diagnosis (Bengio 1994; Pascanu 2013) made gating the obvious response. LSTM (1997) and GRU (2014) are two implementations of the same insight: replace a multiplicative recurrence with a gated additive one so gradients survive the chain rule.
- Sutskever et al.'s 2014 seq2seq paper showed that a fixed-vector encoder-decoder could beat a 30-year-old hand-engineered SMT pipeline on WMT'14 EN-FR. That single result legitimised end-to-end neural NLP.
- Bahdanau et al.'s 2014 attention mechanism removed the fixed-vector bottleneck and gave the decoder dynamic, interpretable access to the source. Luong et al.'s 2015 simplifications (current-step decoder state, dot/general/concat scoring) became the dominant teaching formulation and the direct ancestor of scaled dot-product attention.
- 2014–2017 was a remarkable run: GNMT in production, neural dialogue, neural summarisation, neural captioning — all sharing one recipe.
- The recipe died of its own success. Once attention was clearly carrying the modelling load, the recurrence was just an obstacle to GPU parallelism. The Transformer was the natural conclusion of the seq2seq+attention research program, not a break from it.

## Sources

- [T1] Bengio, Y., Simard, P. & Frasconi, P. (1994). "Learning long-term dependencies with gradient descent is difficult." IEEE Transactions on Neural Networks 5(2):157–166. https://ieeexplore.ieee.org/document/279181/
- [T1] Hochreiter, S. & Schmidhuber, J. (1997). "Long Short-Term Memory." Neural Computation 9(8):1735–1780. MIT Press. https://direct.mit.edu/neco/article/9/8/1735/6109/Long-Short-Term-Memory ; full text: https://www.bioinf.jku.at/publications/older/2604.pdf
- [T1] Gers, F. A., Schmidhuber, J. & Cummins, F. (2000). "Learning to Forget: Continual Prediction with LSTM." Neural Computation 12(10):2451–2471. (Forget-gate addition; cited via standard textbook treatment.)
- [T1] Pascanu, R., Mikolov, T. & Bengio, Y. (2013). "On the difficulty of training Recurrent Neural Networks." ICML 2013. https://arxiv.org/abs/1211.5063 ; PMLR: https://proceedings.mlr.press/v28/pascanu13.pdf
- [T1] Cho, K., van Merriënboer, B., Gulcehre, C., Bahdanau, D., Bougares, F., Schwenk, H. & Bengio, Y. (2014). "Learning Phrase Representations using RNN Encoder-Decoder for Statistical Machine Translation." EMNLP 2014. https://arxiv.org/abs/1406.1078
- [T1] Cho, K., van Merriënboer, B., Bahdanau, D. & Bengio, Y. (2014). "On the Properties of Neural Machine Translation: Encoder-Decoder Approaches." SSST-8. https://arxiv.org/abs/1409.1259
- [T1] Sutskever, I., Vinyals, O. & Le, Q. V. (2014). "Sequence to Sequence Learning with Neural Networks." NIPS 27, pp. 3104–3112. https://arxiv.org/abs/1409.3215 ; https://papers.nips.cc/paper/5346-sequence-to-sequence-learning-with-neural-networks
- [T1] Bahdanau, D., Cho, K. & Bengio, Y. (2014). "Neural Machine Translation by Jointly Learning to Align and Translate." ICLR 2015. https://arxiv.org/abs/1409.0473
- [T1] Chung, J., Gulcehre, C., Cho, K. & Bengio, Y. (2014). "Empirical Evaluation of Gated Recurrent Neural Networks on Sequence Modeling." NIPS 2014 Deep Learning Workshop. https://arxiv.org/abs/1412.3555
- [T1] Luong, M.-T., Pham, H. & Manning, C. D. (2015). "Effective Approaches to Attention-based Neural Machine Translation." EMNLP 2015, pp. 1412–1421. https://aclanthology.org/D15-1166/ ; https://arxiv.org/abs/1508.04025
- [T1] Vinyals, O. & Le, Q. V. (2015). "A Neural Conversational Model." ICML Deep Learning Workshop. https://arxiv.org/abs/1506.05869
- [T1] Wu, Y. et al. (2016). "Google's Neural Machine Translation System: Bridging the Gap between Human and Machine Translation." arXiv:1609.08144. https://arxiv.org/abs/1609.08144
- [T1] Gehring, J., Auli, M., Grangier, D., Yarats, D. & Dauphin, Y. N. (2017). "Convolutional Sequence to Sequence Learning." ICML 2017. https://arxiv.org/abs/1705.03122
- [T1] Vaswani, A. et al. (2017). "Attention Is All You Need." NeurIPS 2017. https://arxiv.org/abs/1706.03762
- [T2] Jurafsky, D. & Martin, J. H. Speech and Language Processing, 3rd edition draft. Stanford. Especially ch. 9 ("RNNs and LSTMs"). https://web.stanford.edu/~jurafsky/slp3/
- [T2] Wikipedia contributors. "Gated recurrent unit." Retrieved 2026. https://en.wikipedia.org/wiki/Gated_recurrent_unit
- [T2] Google Research Blog. (2016). "A Neural Network for Machine Translation, at Production Scale." 27 September 2016. Author: Quoc V. Le and Mike Schuster. https://research.google/blog/a-neural-network-for-machine-translation-at-production-scale/
- [T3] Karpathy, A. (2015). "The Unreasonable Effectiveness of Recurrent Neural Networks." Personal blog, 21 May 2015. http://karpathy.github.io/2015/05/21/rnn-effectiveness/
