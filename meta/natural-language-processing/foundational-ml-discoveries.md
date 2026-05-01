# Foundational ML/DL Discoveries that Powered NLP

This dossier traces the machine-learning and deep-learning primitives that modern NLP was built out of. Scope is strict: every entry below has to point at a downstream NLP technique that actually shipped (word embeddings, ELMo, the Transformer, BERT, GPT). General-purpose computer-vision and RL milestones (AlexNet, YOLO, AlphaGo) are excluded unless their specific contribution flowed back into the NLP toolkit. The arc starts in 1958 with Rosenblatt's perceptron and ends with the late-2010s self-supervised pretraining paradigm — by then the Transformer toolkit (residual streams, LayerNorm, Adam, dropout, SGD with bias-corrected adaptive moments) was effectively closed.

## 1. The Perceptron and the First Setback (1958–1986)

Frank Rosenblatt's *The Perceptron: A Probabilistic Model for Information Storage and Organization in the Brain* (Psychological Review, 1958) introduced the first precisely specified, computational neural network with a learning rule [T1] Rosenblatt 1958, *Psychological Review* 65(6), pp. 386–408, ¶ "Theory and Experiments". The perceptron computes a thresholded linear combination y = step(w·x + b) and updates weights with the perceptron rule w ← w + η(t − y)x. Stripped to its essentials, every modern NLP model still embeds this primitive: a learned linear projection followed by a nonlinearity.

The reason this matters for NLP is structural rather than empirical. Rosenblatt's perceptron is the building block of the multi-layer perceptron (MLP), which is the building block of the feed-forward sublayer in every Transformer block. When BERT or GPT applies an "FFN" between attention layers, it is stacking two perceptron-like affine maps with a nonlinearity in between [T2] Jurafsky & Martin, *Speech and Language Processing*, 3rd ed. draft, ch. 9 ("Transformers"), https://web.stanford.edu/~jurafsky/slp3/.

The perceptron also produced the field's first multi-decade detour. Minsky and Papert's 1969 *Perceptrons* showed single-layer perceptrons cannot represent XOR or any non-linearly-separable function, and the book was widely read as an indictment of the entire connectionist program [T2] Minsky & Papert, *Perceptrons: An Introduction to Computational Geometry*, MIT Press 1969, ch. 0–1; see also [T2] Wikipedia, "Perceptrons (book)", https://en.wikipedia.org/wiki/Perceptrons_(book). The downstream effect on NLP was the dominance of symbolic and statistical approaches (HMMs, n-gram language models, log-linear taggers) until backpropagation reopened the door.

## 2. Backpropagation: The Credit-Assignment Engine (1986)

Rumelhart, Hinton, and Williams's *Learning Representations by Back-Propagating Errors* (Nature, 1986) is the paper that made multi-layer networks trainable in practice [T1] Rumelhart, Hinton & Williams 1986, *Nature* 323, pp. 533–536, ¶1 "We describe a new learning procedure...". The contribution is an efficient way to compute ∂L/∂w_ij for every weight in a deep network by reverse-mode automatic differentiation — the chain rule applied layer by layer, backwards. The same paper also introduces a momentum term in the SGD update, a trick that survives into Adam four decades later [T2] Wikipedia, "Stochastic gradient descent", §"History", citing Rumelhart-Hinton-Williams 1986 borrowing from Polyak 1964.

Backpropagation is the foundation that lets every later NLP architecture in this dossier learn at all. Without it, word2vec, LSTMs, Transformers, and BERT cannot be optimized — there is simply no way to assign error to the embedding matrix at the bottom from a loss computed at the softmax on top. Hochreiter's 1991 diplom thesis identified the crucial caveat: when the chain rule is applied through many layers (or many timesteps in an RNN), the gradient magnitude shrinks or explodes geometrically — the *vanishing gradient* problem [T2] Hochreiter 1991 diplom thesis, summarized in Hochreiter et al., "Gradient Flow in Recurrent Nets", https://www.bioinf.jku.at/publications/older/2304.pdf, §1; [T2] Wikipedia, "Vanishing gradient problem", https://en.wikipedia.org/wiki/Vanishing_gradient_problem. This is the single observation that motivates the LSTM's constant-error-carousel, the Transformer's residual stream, and the Pre-LN variant — each is an architectural fix to keep ∂L/∂x_l from collapsing.

## 3. Distributed Representations: From Bengio 2003 to Word2Vec (2003–2013)

The first NLP-native exploitation of backprop-trained networks was Bengio, Ducharme, Vincent & Jauvin's *A Neural Probabilistic Language Model* (JMLR, 2003) [T1] Bengio et al. 2003, *JMLR* 3, pp. 1137–1155, https://www.jmlr.org/papers/volume3/bengio03a/bengio03a.pdf. The architecture is a feed-forward network that maps each context word through a shared lookup table C ∈ ℝ^|V|×d into a dense vector, concatenates the n−1 context vectors, runs them through a tanh hidden layer, and outputs a softmax over the vocabulary. The trained C is the first "word embedding" — a learned distributed representation in which similar words have nearby vectors. Bengio et al. report significant perplexity gains over a state-of-the-art trigram model on Brown and AP News [T1] Bengio et al. 2003, §5 "Experimental Results".

Collobert and Weston's 2008 *A Unified Architecture for NLP* extended this with multi-task learning over POS, chunking, NER and SRL using shared word embeddings, and used unlabeled text via a language-model pairwise ranking loss — an early form of self-supervised pretraining for NLP [T1] Collobert & Weston 2008, *Proc. 25th ICML*, pp. 160–167, https://ronan.collobert.com/pub/matos/2008_nlp_icml.pdf, §3 "Multi-Task Learning". The paper later won the ICML test-of-time award; Sebastian Ruder credits it as the origin of pretrained word embeddings and CNNs-for-text [T3] Ruder, "A Review of the Neural History of NLP", https://www.ruder.io/a-review-of-the-recent-history-of-nlp/, §"Multi-task learning".

Mikolov et al.'s 2013 *Efficient Estimation of Word Representations in Vector Space* (CBOW and skip-gram) and *Distributed Representations of Words and Phrases and Their Compositionality* turned these ideas into a tool millions of practitioners could run on a laptop [T1] Mikolov et al. 2013, "Efficient Estimation...", arXiv:1301.3781, p. 4 ¶"Continuous Bag-of-Words Model"; [T1] Mikolov et al. 2013, "Distributed Representations...", NeurIPS 2013, https://proceedings.neurips.cc/paper/2013/file/9aa42b31882ec039965f3c4923ce901b-Paper.pdf. CBOW predicts the center word from a sum of context vectors; skip-gram predicts each context word from the center word, optimised with hierarchical softmax or negative sampling. The famous "king − man + woman ≈ queen" arithmetic comes from these vectors [T2] Wikipedia, "Word2vec", https://en.wikipedia.org/wiki/Word2vec, §"Linguistic regularities". Word embeddings became the de facto first layer of every neural NLP system between 2013 and 2018.

## 4. Optimizers: SGD, Momentum, AdaGrad, RMSProp, Adam (1951–2014)

Robbins and Monro's 1951 stochastic-approximation work is the mathematical ancestor of every optimizer used to train modern NLP models [T2] Robbins & Monro 1951, *Annals of Mathematical Statistics* 22(3), pp. 400–407, summarised at [T2] Wikipedia, "Stochastic gradient descent", §"Background". SGD replaces the full-data gradient with a mini-batch estimate: w_{t+1} = w_t − η · ∇L_B(w_t). For NLP, where vocabulary embedding tables produce extremely sparse gradients (only the rows for tokens in the current batch get a nonzero update), per-parameter adaptivity matters disproportionately.

Three steps closed the gap between vanilla SGD and what people actually run today:

- **AdaGrad (Duchi, Hazan & Singer 2011)** scales each parameter by 1/√(Σ g²), giving sparse features (rare words, in NLP terms) larger effective learning rates [T1] Duchi, Hazan & Singer 2011, *JMLR* 12, pp. 2121–2159, §3 "Diagonal AdaGrad". The downside is that accumulated squared gradients monotonically grow, so the effective learning rate eventually goes to zero.
- **RMSProp (Hinton, Coursera lecture 2012)** replaces the unbounded sum with an exponential moving average of squared gradients. It was never published as a paper — Hinton presented it in his Coursera *Neural Networks for Machine Learning* lectures [T3] Hinton, Coursera lecture slide for "RMSProp", https://www.cs.toronto.edu/~tijmen/csc321/slides/lecture_slides_lec6.pdf, slide 29.
- **Adam (Kingma & Ba 2014)** combines momentum (an EMA of the gradient) with RMSProp (an EMA of the squared gradient) and adds a bias correction so the moments are unbiased early in training [T1] Kingma & Ba 2014, "Adam: A Method for Stochastic Optimization", arXiv:1412.6980, Algorithm 1.

The Adam update with default hyperparameters (α=0.001, β₁=0.9, β₂=0.999, ε=10⁻⁸) is:

```
m_t = β₁ m_{t-1} + (1 − β₁) g_t
v_t = β₂ v_{t-1} + (1 − β₂) g_t²
m̂_t = m_t / (1 − β₁^t)
v̂_t = v_t / (1 − β₂^t)
w_t = w_{t-1} − α · m̂_t / (√v̂_t + ε)
```

[T1] Kingma & Ba 2014, Algorithm 1; defaults defined on p. 2.

Adam's specific relevance to NLP is that the gradient signal across an embedding table or attention output projection is heteroscedastic — some directions see large, frequent updates, others tiny, sparse ones. Per-parameter adaptive moments make this learnable without painstaking per-layer learning-rate tuning. It is no accident that *Attention Is All You Need* trains with "the Adam optimizer with β₁=0.9, β₂=0.98, ε=10⁻⁹" and a custom warmup schedule [T1] Vaswani et al. 2017, *NeurIPS*, "Attention Is All You Need", §5.3 "Optimizer". Loshchilov & Hutter's 2017 AdamW correction (decoupled weight decay) is now the default for almost every transformer pretraining run [T1] Loshchilov & Hutter 2019, "Decoupled Weight Decay Regularization", ICLR, arXiv:1711.05101, §3.

## 5. Regularization: Dropout (2014)

Srivastava, Hinton, Krizhevsky, Sutskever, and Salakhutdinov's *Dropout: A Simple Way to Prevent Neural Networks from Overfitting* (JMLR 15, 2014, pp. 1929–1958) introduces a regularizer that randomly zeroes a fraction p of hidden units on each training example [T1] Srivastava et al. 2014, *JMLR* 15(56), pp. 1929–1958, https://jmlr.org/papers/v15/srivastava14a.html, §1. Conceptually, training-time dropout samples one of 2^n thinned subnetworks per minibatch; at test time the unthinned network's weights are scaled by p, giving a fast geometric-mean approximation of model averaging [T1] Srivastava et al. 2014, §2 "Motivation" and §3 "Model Description".

Dropout is the workhorse regularizer of every NLP system between AlexNet and modern transformers. Vaswani et al. apply dropout to (i) the output of each sub-layer before residual addition, (ii) the sums of embeddings and positional encodings, with rate P_drop = 0.1 [T1] Vaswani et al. 2017, §5.4 "Regularization". BERT applies dropout 0.1 to all attention probabilities and hidden activations [T1] Devlin et al. 2019, "BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding", NAACL 2019, arXiv:1810.04805, §A.2 "Pre-training Procedure". A specifically-recurrent variant — *variational dropout*, applying the same mask across timesteps — was crucial for getting LSTM language models to overfit less catastrophically [T1] Gal & Ghahramani 2016, "A Theoretically Grounded Application of Dropout in RNNs", NeurIPS 2016, arXiv:1512.05287.

## 6. Normalization: From BatchNorm to LayerNorm (2015–2016)

Ioffe and Szegedy's 2015 BatchNorm is the better-known parent paper, but it is fundamentally ill-suited to NLP. Sequence models have variable batch dimensions, padding, and tokens whose statistics depend on position in the sequence — and recurrent computation makes per-batch statistics ill-defined across timesteps [T1] Ioffe & Szegedy 2015, "Batch Normalization", ICML, arXiv:1502.03167, §1.

Ba, Kiros, and Hinton's *Layer Normalization* (2016) is the form of normalization that actually shipped in the Transformer [T1] Ba, Kiros & Hinton 2016, "Layer Normalization", arXiv:1607.06450, §3. LayerNorm normalizes across the *feature* dimension of a single training example, not across the batch:

```
μ = (1/H) Σ_i a_i           (mean across hidden units i = 1..H)
σ² = (1/H) Σ_i (a_i − μ)²
ĥ_i = γ_i · (a_i − μ) / √(σ² + ε) + β_i
```

The learned γ and β give the layer adaptive gain and bias. Crucially, LayerNorm is *batch-size independent* and *identical at training and test time*, two properties batch-norm lacks [T2] Ba, Kiros & Hinton 2016, summarised in [T2] CITS4012 NLP course notes, "Layer Normalization", https://weiliu2k.github.io/CITS4012/transformer/LayerNorm.html.

Vaswani et al. apply LayerNorm in the canonical "post-norm" position: x_{l+1} = LayerNorm(x_l + Sublayer(x_l)) [T1] Vaswani et al. 2017, §3.1 "Encoder and Decoder Stacks". Xiong et al. (2020) showed empirically and theoretically that this post-LN placement causes the gradient norm at the top of the network to scale as O(√d_model) at initialization while the bottom layers' gradients vanish, requiring the warmup schedule that the original Transformer paper had to introduce as a hack. Moving LayerNorm *inside* the residual block (pre-LN: x_{l+1} = x_l + Sublayer(LayerNorm(x_l))) gives gradients that are bounded independent of depth and enables training without warmup [T1] Xiong et al. 2020, "On Layer Normalization in the Transformer Architecture", *ICML* PMLR 119, https://proceedings.mlr.press/v119/xiong20b/xiong20b.pdf, Theorem 1. Every modern LLM (GPT-2 onwards, LLaMA, Mistral) uses pre-LN. RMSNorm — a simplified LayerNorm without the centering step — has further replaced LayerNorm in LLaMA-style models [T1] Zhang & Sennrich 2019, "Root Mean Square Layer Normalization", NeurIPS 2019, arXiv:1910.07467.

## 7. Residual Connections: From ResNet to the Residual Stream (2015–2021)

He, Zhang, Ren, and Sun's *Deep Residual Learning for Image Recognition* (CVPR 2016) is a vision paper, but its architectural primitive is now the most-used building block in NLP [T1] He et al. 2015, "Deep Residual Learning for Image Recognition", arXiv:1512.03385, §3.1. The contribution: instead of having a stack of layers learn an underlying mapping H(x) directly, learn the *residual* F(x) = H(x) − x and add the input back via a skip connection: y = F(x, {W_i}) + x. The motivation was the *degradation problem* — empirically, naively stacking more layers in a plain network increased *training* error, even when the deeper net could in principle represent the shallower one's function [T1] He et al. 2015, §1 "Introduction", Fig. 1.

The Transformer adopts this construct twice per block — once around multi-head attention, once around the FFN — and Devlin et al.'s BERT inherits it unchanged [T1] Vaswani et al. 2017, §3.1; [T1] Devlin et al. 2019, §3.

The mechanistic-interpretability community has reinterpreted these skip connections as a *residual stream*. Elhage et al.'s "A Mathematical Framework for Transformer Circuits" (Anthropic, 2021) describes the residual stream as a high-dimensional shared communication channel that every component (token embedding, attention heads, MLPs, unembedding) reads from via a linear projection and writes to additively [T2] Elhage et al. 2021, "A Mathematical Framework for Transformer Circuits", Anthropic, https://transformer-circuits.pub/2021/framework/index.html, §"Virtual Weights and the Residual Stream Bottleneck". Two consequences flow from the additivity:

1. Gradients can travel from the loss back to the embeddings along a *length-1* path (∂L/∂x_0 includes a direct copy of ∂L/∂x_L), avoiding the vanishing-gradient collapse Hochreiter identified in 1991.
2. Different attention heads and MLP sublayers can be analyzed as *independent operations* whose outputs sum into the stream — making mechanistic interpretability tractable in a way no recurrent architecture allows.

ResNet's residual is a vision discovery; the residual *stream* is a transformer-native reframing that has reshaped how the field thinks about computation inside language models.

## 8. Self-Supervised Pretraining as an Idea (2003–2018)

Self-supervised pretraining is less a single discovery than a paradigm shift, but it has clean ancestry. The core idea — train a model to predict parts of its input from other parts of its input, on cheap unlabeled text, then fine-tune on the supervised task — is implicit in Bengio's 2003 NLM (predict the next word) and explicit in Collobert & Weston 2008 (joint pairwise-ranking language-model loss alongside supervised tasks) [T1] Collobert & Weston 2008, §3.5 "Language Model".

Three lines made it the dominant paradigm:

- **Dai & Le 2015, *Semi-supervised Sequence Learning*** showed that pretraining an LSTM as a language model and fine-tuning it for classification beat training-from-scratch baselines [T1] Dai & Le 2015, NeurIPS, arXiv:1511.01432, §3.
- **Howard & Ruder 2018, ULMFiT** added discriminative learning rates and gradual unfreezing, demonstrating transfer-learning scaling laws on six text-classification benchmarks [T1] Howard & Ruder 2018, "Universal Language Model Fine-tuning for Text Classification", ACL 2018, arXiv:1801.06146.
- **Peters et al. 2018, ELMo** trained a bidirectional LSTM language model and used the layer-wise contextual representations as features for downstream tasks, posting state-of-the-art results on six NLP tasks simultaneously [T1] Peters et al. 2018, "Deep Contextualized Word Representations", NAACL 2018, arXiv:1802.05365, §3.

Radford et al.'s GPT (2018) replaced the LSTM with a Transformer decoder and kept the next-word objective; Devlin et al.'s BERT replaced it with a Transformer encoder and a *masked language modelling* objective in which 15% of tokens are masked and the model must reconstruct them, plus a next-sentence-prediction auxiliary [T1] Devlin et al. 2019, §3.1, p. 4. The MLM objective is itself a denoising autoencoder applied to text — and it is the architectural decision that lets a single pretrained checkpoint solve QA, NLI, NER, and classification with a small task-specific head on top.

Sebastian Ruder identifies pretrained language models as the eighth and most consequential of his neural-NLP milestones [T3] Ruder, "A Review of the Neural History of NLP", https://www.ruder.io/a-review-of-the-recent-history-of-nlp/, §"Pretrained language models". The conceptual lineage — perceptron → backprop → distributed representations → Adam-trained transformer with dropout, residual connections and LayerNorm, fine-tuned from a self-supervised checkpoint — closes here.

## 9. What This Toolkit Looks Like Stacked Together

A modern NLP forward pass uses every primitive surveyed above:

1. **Embedding** (Bengio 2003 / Mikolov 2013): tokens → dense vectors via a learned lookup.
2. **Positional encoding + dropout** (Srivastava 2014): regularizes the input sum.
3. **LayerNorm** (Ba 2016) at the start of each block (pre-LN style, post-Xiong 2020).
4. **Self-attention sublayer**, with its **output added** back to the residual stream (He 2015).
5. **LayerNorm + FFN** (perceptron blocks, Rosenblatt 1958) **+ residual**.
6. Repeat L times.
7. **Final LayerNorm + linear unembedding** to vocab logits.
8. **Cross-entropy loss**, optimized by **AdamW** (Kingma 2014, Loshchilov 2019) with **backpropagation** (Rumelhart 1986).
9. **Pretrain self-supervised** (Devlin 2019) on web-scale text, fine-tune on task-specific labels.

Every line in that pipeline traces back to a paper in this dossier. The genuinely NLP-native discoveries — attention, the Transformer, MLM — are out of scope for this round, but they sit on this exact stack.

## Sources

**Type 1 — peer-reviewed papers and conference proceedings**

- Rosenblatt, F. (1958). The Perceptron: A Probabilistic Model for Information Storage and Organization in the Brain. *Psychological Review* 65(6), 386–408. https://psycnet.apa.org/doi/10.1037/h0042519
- Rumelhart, D. E., Hinton, G. E. & Williams, R. J. (1986). Learning Representations by Back-Propagating Errors. *Nature* 323, 533–536. https://www.nature.com/articles/323533a0
- Hochreiter, S. & Schmidhuber, J. (1997). Long Short-Term Memory. *Neural Computation* 9(8), 1735–1780. https://direct.mit.edu/neco/article/9/8/1735/6109/Long-Short-Term-Memory
- Bengio, Y., Ducharme, R., Vincent, P. & Jauvin, C. (2003). A Neural Probabilistic Language Model. *Journal of Machine Learning Research* 3, 1137–1155. https://www.jmlr.org/papers/volume3/bengio03a/bengio03a.pdf
- Collobert, R. & Weston, J. (2008). A Unified Architecture for Natural Language Processing: Deep Neural Networks with Multitask Learning. *Proc. 25th ICML*, 160–167. https://ronan.collobert.com/pub/matos/2008_nlp_icml.pdf
- Duchi, J., Hazan, E. & Singer, Y. (2011). Adaptive Subgradient Methods for Online Learning and Stochastic Optimization. *JMLR* 12, 2121–2159. https://jmlr.org/papers/v12/duchi11a.html
- Mikolov, T., Chen, K., Corrado, G. & Dean, J. (2013). Efficient Estimation of Word Representations in Vector Space. *ICLR Workshop*, arXiv:1301.3781. https://arxiv.org/abs/1301.3781
- Mikolov, T., Sutskever, I., Chen, K., Corrado, G. S. & Dean, J. (2013). Distributed Representations of Words and Phrases and their Compositionality. *NeurIPS 2013*. https://proceedings.neurips.cc/paper/2013/file/9aa42b31882ec039965f3c4923ce901b-Paper.pdf
- Srivastava, N., Hinton, G., Krizhevsky, A., Sutskever, I. & Salakhutdinov, R. (2014). Dropout: A Simple Way to Prevent Neural Networks from Overfitting. *JMLR* 15(56), 1929–1958. https://jmlr.org/papers/v15/srivastava14a.html
- Kingma, D. P. & Ba, J. (2014). Adam: A Method for Stochastic Optimization. arXiv:1412.6980. https://arxiv.org/abs/1412.6980
- Ioffe, S. & Szegedy, C. (2015). Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift. *ICML 2015*, arXiv:1502.03167. https://arxiv.org/abs/1502.03167
- He, K., Zhang, X., Ren, S. & Sun, J. (2015). Deep Residual Learning for Image Recognition. arXiv:1512.03385. https://arxiv.org/abs/1512.03385
- Dai, A. M. & Le, Q. V. (2015). Semi-supervised Sequence Learning. *NeurIPS 2015*, arXiv:1511.01432. https://arxiv.org/abs/1511.01432
- Gal, Y. & Ghahramani, Z. (2016). A Theoretically Grounded Application of Dropout in Recurrent Neural Networks. *NeurIPS 2016*, arXiv:1512.05287. https://arxiv.org/abs/1512.05287
- Ba, J. L., Kiros, J. R. & Hinton, G. E. (2016). Layer Normalization. arXiv:1607.06450. https://arxiv.org/abs/1607.06450
- Vaswani, A. et al. (2017). Attention Is All You Need. *NeurIPS 2017*, arXiv:1706.03762. https://arxiv.org/abs/1706.03762
- Howard, J. & Ruder, S. (2018). Universal Language Model Fine-tuning for Text Classification. *ACL 2018*, arXiv:1801.06146. https://arxiv.org/abs/1801.06146
- Peters, M. E. et al. (2018). Deep Contextualized Word Representations (ELMo). *NAACL 2018*, arXiv:1802.05365. https://arxiv.org/abs/1802.05365
- Devlin, J., Chang, M.-W., Lee, K. & Toutanova, K. (2019). BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding. *NAACL 2019*, arXiv:1810.04805. https://arxiv.org/abs/1810.04805
- Loshchilov, I. & Hutter, F. (2019). Decoupled Weight Decay Regularization (AdamW). *ICLR 2019*, arXiv:1711.05101. https://arxiv.org/abs/1711.05101
- Zhang, B. & Sennrich, R. (2019). Root Mean Square Layer Normalization. *NeurIPS 2019*, arXiv:1910.07467. https://arxiv.org/abs/1910.07467
- Xiong, R. et al. (2020). On Layer Normalization in the Transformer Architecture. *ICML 2020 (PMLR 119)*. https://proceedings.mlr.press/v119/xiong20b/xiong20b.pdf
- Robbins, H. & Monro, S. (1951). A Stochastic Approximation Method. *Annals of Mathematical Statistics* 22(3), 400–407.

**Type 2 — textbooks, official documentation, landmark whitepapers, course notes**

- Jurafsky, D. & Martin, J. H. *Speech and Language Processing*, 3rd edition draft, ch. 9 "Transformers" and ch. 11 "Pretrained Language Models". https://web.stanford.edu/~jurafsky/slp3/
- Minsky, M. & Papert, S. (1969). *Perceptrons: An Introduction to Computational Geometry*. MIT Press.
- Elhage, N. et al. (2021). A Mathematical Framework for Transformer Circuits. Anthropic / Transformer Circuits Thread. https://transformer-circuits.pub/2021/framework/index.html
- Hochreiter, S., Bengio, Y., Frasconi, P. & Schmidhuber, J. Gradient Flow in Recurrent Nets: the Difficulty of Learning Long-Term Dependencies (review of Hochreiter's 1991 thesis). https://www.bioinf.jku.at/publications/older/2304.pdf
- Wikipedia. *Stochastic gradient descent*. https://en.wikipedia.org/wiki/Stochastic_gradient_descent
- Wikipedia. *Vanishing gradient problem*. https://en.wikipedia.org/wiki/Vanishing_gradient_problem
- Wikipedia. *Word2vec*. https://en.wikipedia.org/wiki/Word2vec
- Wikipedia. *BERT (language model)*. https://en.wikipedia.org/wiki/BERT_(language_model)
- Wikipedia. *Perceptrons (book)*. https://en.wikipedia.org/wiki/Perceptrons_(book)
- CITS4012 NLP course notes (UWA). Layer Normalization. https://weiliu2k.github.io/CITS4012/transformer/LayerNorm.html

**Type 3 — blogs, talks, lecture slides, podcasts**

- Ruder, S. A Review of the Neural History of Natural Language Processing. https://www.ruder.io/a-review-of-the-recent-history-of-nlp/
- Alammar, J. The Illustrated BERT, ELMo, and co. https://jalammar.github.io/illustrated-bert/
- Hinton, G. RMSProp lecture, *Neural Networks for Machine Learning* (Coursera 2012), slides at https://www.cs.toronto.edu/~tijmen/csc321/slides/lecture_slides_lec6.pdf
- Raschka, S. About LayerNorm Variants in the Original Transformer Paper. https://magazine.sebastianraschka.com/p/why-the-original-transformer-figure
