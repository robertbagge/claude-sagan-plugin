# Word Embeddings & Distributional Semantics

A working title for this topic could be "how NLP learned to do arithmetic on meaning." The arc runs from a one-line slogan in mid-twentieth-century British linguistics, through count-based matrices in 1990s information retrieval, into the dense-vector revolution of 2013–2017, and finally to the contextual representations that rendered the static-embedding era obsolete almost as soon as it had peaked. The technical thread is consistent throughout: meaning is approximated by patterns of co-occurrence. What changes is the algorithm and the representational dimensionality.

## 1. From Firth and Harris to count-based vectors

The distributional hypothesis is usually given two parents. Zellig Harris's "Distributional Structure" (1954) argued that linguistic units could be characterised by their distributional environments — the surrounding contexts that licensed substitution `[T1] Harris 1954, Word 10(2/3), pp. 146–162`. J.R. Firth's better-known 1957 phrasing — "you shall know a word by the company it keeps" — was less a methodological program than a slogan, but it became the rallying cry for the research line that followed `[T2] Wikipedia, "John Rupert Firth", https://en.wikipedia.org/wiki/John_Rupert_Firth, sec. "Notable contributions"`. Boleda's recent reappraisal stresses that Firth and Harris were doing different things: Harris was building structural linguistics from co-occurrence; Firth was making a stronger claim that distributional facts *are* meaning `[T1] Boleda 2022, arXiv:2205.07750, p. 2, ¶3`.

The first computational realisation that mattered for NLP was Latent Semantic Analysis. Deerwester, Dumais, Furnas, Landauer and Harshman (1990) constructed a term-by-document matrix `X` and applied truncated singular value decomposition: `X ≈ U Σ Vᵀ`, keeping the top-k singular triples to compress documents and terms into a shared k-dimensional space (typically k ≈ 100–300) `[T1] Deerwester et al. 1990, JASIS 41(6), p. 391, ¶1; pp. 397–398, §3`. The crucial trick was that the low-rank approximation made synonyms collapse onto similar directions even when their surface co-occurrence was sparse — the Eckart-Young theorem giving the best rank-k Frobenius approximation of the original matrix `[T2] Wikipedia, "Latent semantic analysis", https://en.wikipedia.org/wiki/Latent_semantic_analysis, sec. "Mathematics of LSA"`.

Subsequent work refined the count-based recipe. Raw counts were replaced with positive pointwise mutual information, `PPMI(w, c) = max(0, log P(w,c) / (P(w) P(c)))`, which downweights common-but-uninformative co-occurrences. PMI smoothing, context-window weighting, and dimensionality reduction are the three knobs that distinguish most pre-2013 vector-space models `[T2] Jurafsky & Martin, Speech and Language Processing 3rd ed. draft, ch. 6 "Vector Semantics and Embeddings", §6.6 (PPMI)`.

The neural prelude is Bengio, Ducharme, Vincent and Jauvin's "A Neural Probabilistic Language Model" (2003). Their feed-forward language model represented each word as a learned dense vector `C(w) ∈ ℝᵐ`, concatenated context vectors through a hidden layer, and predicted the next word with a softmax over the vocabulary `[T1] Bengio et al. 2003, JMLR 3, p. 1139, §2; pp. 1141–1142, §2.1`. The embedding matrix was a side-effect of language-model training, and the paper explicitly framed distributed representations as the answer to the curse of dimensionality in n-gram modelling `[T1] Bengio et al. 2003, JMLR 3, p. 1138, ¶3`. The architecture was too expensive to train at scale; the idea waited a decade.

## 2. word2vec: CBOW, skip-gram, and the math of negative sampling

Mikolov, Chen, Corrado and Dean's "Efficient Estimation of Word Representations in Vector Space" (arXiv 1301.3781, 2013) made two design moves whose only purpose was speed: drop the non-linear hidden layer of Bengio's NPLM, and train on much more data `[T1] Mikolov et al. 2013a, arXiv:1301.3781, p. 3, §3`. Two architectures emerged.

- **Continuous Bag-of-Words (CBOW)** averages the embeddings of the surrounding context words within a window and predicts the centre word. Order is discarded inside the window — hence "bag-of-words" `[T1] Mikolov et al. 2013a, arXiv:1301.3781, p. 4, §3.1`.
- **Skip-gram** does the opposite: from the centre word `w_t`, predict each context word `w_{t+j}` for `j ∈ [−c, c], j ≠ 0`, with the training objective `(1/T) Σ_t Σ_{−c ≤ j ≤ c, j ≠ 0} log p(w_{t+j} | w_t)` `[T1] Mikolov et al. 2013a, arXiv:1301.3781, p. 4, §3.2`.

The naïve `p(w_O | w_I) = exp(v'_{w_O}ᵀ v_{w_I}) / Σ_w exp(v'_wᵀ v_{w_I})` is intractable: `O(|V|)` per step for vocabularies in the millions. The companion paper "Distributed Representations of Words and Phrases and their Compositionality" (NIPS 2013) introduced two replacements `[T1] Mikolov et al. 2013b, NIPS 2013, p. 1, §1`.

The first is **hierarchical softmax**, which builds a binary tree (Huffman-coded by frequency) over the vocabulary; each word becomes a path of `O(log |V|)` binary decisions, each scored by a sigmoid against an internal-node vector. Probability of a word is the product of probabilities along its path `[T1] Mikolov et al. 2013b, NIPS 2013, p. 2, §2.1`.

The second, simpler and more popular, is **negative sampling (NEG)**. SGNS replaces the softmax with a per-pair binary classification: distinguish observed `(w, c)` pairs from `k` randomly drawn "noise" pairs. The objective per training pair is

```
J(θ; w_I, w_O) = log σ(v'_{w_O}ᵀ v_{w_I}) + Σ_{i=1..k} 𝔼_{w_i ~ P_n(w)} [ log σ(−v'_{w_i}ᵀ v_{w_I}) ]
```

with `σ(x) = 1/(1+e^{−x})` `[T1] Mikolov et al. 2013b, NIPS 2013, p. 4, eq. (4)`. Goldberg and Levy's clean derivation (arXiv 1402.3722, 2014) shows this is exactly the loss of a logistic regression that treats real `(w, c)` pairs as positives and `k` sampled context words as negatives, dropping the partition function entirely `[T1] Goldberg & Levy 2014, arXiv:1402.3722, p. 4, §3.2`.

The noise distribution is not the unigram distribution but `P_n(w) ∝ U(w)^{3/4}`, an empirical compromise that boosts the negative-sample probability of mid-frequency words without flooding the loss with stopwords `[T1] Mikolov et al. 2013b, NIPS 2013, p. 4, §2.2`. A separate **subsampling** trick discards each occurrence of a word `w_i` with probability `1 − √(t/f(w_i))` (Mikolov used `t ≈ 10⁻⁵`), effectively downsampling "the", "of", "and" while preserving the ranking of frequencies `[T1] Mikolov et al. 2013b, NIPS 2013, p. 4, §2.3`. With these tricks, skip-gram trains on a 1.6-billion-word corpus in under a day on commodity hardware `[T1] Mikolov et al. 2013a, arXiv:1301.3781, p. 1, abstract`.

Levy and Goldberg (NIPS 2014) supplied the analytical bridge to the count-based tradition: SGNS with `k` negatives is implicitly factorising the matrix whose `(w, c)` entry is `PMI(w, c) − log k`, the so-called "shifted PMI" matrix `[T1] Levy & Goldberg 2014, NIPS 2014, p. 1, §1; p. 4, Theorem 1`. SGNS was therefore not a new family of model; it was a stochastic, sparse-friendly factoriser of the same co-occurrence statistics LSA had been working with all along `[T3] Omer Levy, "Neural Word Embeddings as Implicit Matrix Factorization", https://levyomer.wordpress.com/2014/09/10/neural-word-embeddings-as-implicit-matrix-factorization/`.

## 3. GloVe and fastText: count-based revival and subword morphology

GloVe — Pennington, Socher and Manning, EMNLP 2014 — leaned into the matrix-factorisation interpretation directly. They argued that ratios of co-occurrence probabilities, not raw probabilities, are what encode meaning, and derived a weighted log-bilinear regression over the global co-occurrence matrix `X` `[T1] Pennington et al. 2014, EMNLP, pp. 1532–1533, §1; p. 1535, §3`. The objective is

```
J = Σ_{i,j} f(X_{ij}) ( w_iᵀ w̃_j + b_i + b̃_j − log X_{ij} )²
```

where `w_i` and `w̃_j` are the word and context vectors and `f` is a weighting function that suppresses the influence of very rare and very frequent pairs `[T2] Wikipedia, "GloVe", https://en.wikipedia.org/wiki/GloVe, sec. "Definition"`. The standard choice is

```
f(x) = (x / x_max)^α   if x < x_max
f(x) = 1                otherwise
```

with `x_max = 100` and `α = 3/4` — the same power used in word2vec's negative sampler, picked empirically `[T2] Wikipedia, "GloVe", sec. "Definition"`. GloVe's selling points were transparent statistics (you can write down what is being modelled in a single equation) and slightly better performance on the analogy benchmark — though Levy, Goldberg and Dagan's careful 2015 hyperparameter audit showed most apparent differences between GloVe and SGNS were down to design choices in pre-processing and tuning, not the underlying objective `[T1] Levy, Goldberg & Dagan 2015, TACL 3, pp. 211–225, §6`.

fastText (Bojanowski, Grave, Joulin, Mikolov, TACL 2017) attacked a different weakness of word2vec and GloVe: the assumption that each word type is atomic. Both treat `play`, `playing`, `played` as unrelated rows of the embedding matrix, which leaves morphologically rich languages and rare vocabulary badly served `[T1] Bojanowski et al. 2017, TACL 5, p. 135, §1`. fastText represents each word as the sum of its character-n-gram embeddings, with `n` typically in `{3, 4, 5, 6}`. The word `where` (delimited by `<` and `>`) yields tri-grams `<wh, whe, her, ere, re>` plus the special whole-word token `<where>`, and the word vector is the sum of those n-gram vectors `[T1] Bojanowski et al. 2017, TACL 5, p. 137, §3.2`. Two consequences fall out: morphologically related forms share parameters, and out-of-vocabulary words still receive a vector at inference time, computed from their n-grams. The objective is otherwise SGNS.

## 4. The analogy task: linear substructure, what works, what doesn't

The single demo that put word vectors on the map was the analogy task. Mikolov reported that vector arithmetic recovered semantic and syntactic relations: `vec("king") − vec("man") + vec("woman")` yielded a vector whose nearest neighbour was `vec("queen")` `[T1] Mikolov et al. 2013a, arXiv:1301.3781, p. 5, §4`. The standard scoring rule is **3CosAdd**: pick the word `b*` maximising `cos(b*, b − a + a*)`, excluding the three input words `a, a*, b` from candidates `[T1] Mikolov et al. 2013a, arXiv:1301.3781, p. 5, §4`.

Three findings dialled back the early enthusiasm.

First, Levy and Goldberg's "Linguistic Regularities in Sparse and Explicit Word Representations" (CoNLL 2014) showed the analogy effect was not unique to neural embeddings; explicit PPMI vectors do almost as well, suggesting the linear structure is a property of the underlying statistics, not of any particular learning algorithm `[T1] Levy & Goldberg 2014b, CoNLL, pp. 171–180, §4`.

Second, Drozd, Gladkova and Matsuoka's COLING 2016 paper "Beyond king − man + woman = queen" demonstrated that 3CosAdd is fragile: simple set-based methods (averaging multiple example pairs of the same relation, or a learned classifier) consistently beat vector offset on Google's analogy benchmark by 10–30 percentage points on hard relations `[T1] Drozd et al. 2016, COLING, pp. 3519–3530, §4`. Worse, the standard evaluation excludes the three input words from the candidate set; if you re-include them, the most popular "answer" for many analogies is one of the input words `[T1] Drozd et al. 2016, COLING, p. 3520, §2`.

Third, Linzen's NAACL 2016 RepEval paper "Issues in evaluating semantic spaces using word analogies" showed that for many relations, you can achieve high accuracy by simply taking the nearest neighbour of `b` (the word the model is being asked about), or of the two-word average of `b` and `a*`, while ignoring the offset `b* − a* + a` entirely `[T1] Linzen 2016, RepEval @ NAACL, pp. 13–18, §3`. The benchmark was, in part, measuring something other than analogical reasoning.

The fourth wrinkle is social. Bolukbasi, Chang, Zou, Saligrama and Kalai's "Man is to Computer Programmer as Woman is to Homemaker?" (NIPS 2016) found that the same vector arithmetic that produced `king − man + woman ≈ queen` also produced `computer programmer − man + woman ≈ homemaker` and similarly stereotyped completions across professions `[T1] Bolukbasi et al. 2016, NIPS 2016, arXiv:1607.06520, p. 1, abstract; p. 4, §4`. They identified a "gender direction" by averaging `he − she`, `man − woman`, etc., and proposed projecting gender-neutral words onto the orthogonal subspace. Subsequent work (Gonen and Goldberg 2019) showed the bias was not removed but hidden — clustering structure persisted — but the political and engineering point landed: word embeddings absorb whatever statistical regularities exist in the training corpus, including the ones we'd prefer they didn't `[T2] Jurafsky & Martin 3rd ed. draft, ch. 6, §6.11 "Bias and Embeddings"`.

## 5. From static to contextual: ELMo as the bridge to the Transformer era

The static word-vector paradigm has a structural ceiling: a single vector per word type cannot represent polysemy. `bank` (river / financial) and `play` (theatrical / verb) get one row each, averaging across senses. By 2017 the consensus problem in the field was how to make the embedding a function of the surrounding sentence rather than a lookup `[T2] Jurafsky & Martin 3rd ed. draft, ch. 11 "Transfer Learning with Pretrained Language Models", §11.1`.

ELMo — Peters, Neumann, Iyyer, Gardner, Clark, Lee and Zettlemoyer, NAACL 2018 — was the cleanest demonstration that this was both possible and practical. The architecture is a two-layer bidirectional LSTM language model trained on the 1B Word Benchmark; for each token, ELMo extracts the hidden states from both directions across all layers, and produces a token representation as a learned weighted sum

```
ELMo_k^task = γ^task Σ_{j=0..L} s_j^task h_{k,j}^LM
```

where `h_{k,j}^LM` is the LSTM state for token `k` at layer `j`, `s_j^task` are softmax-normalised mixing weights learned per downstream task, and `γ^task` is a scalar `[T1] Peters et al. 2018, NAACL, p. 2228, §3.2, eq. (1)`. The word representation is now a function of the entire input sentence — `bank` in *river bank* gets a different vector from `bank` in *Bank of England*.

ELMo set new state-of-the-art on six tasks in a single paper: SQuAD (question answering), SNLI (textual entailment), SRL, coref, NER, and SST-5 (sentiment), each by a few absolute points over strong task-specific architectures, simply by replacing GloVe inputs with ELMo and otherwise leaving the model unchanged `[T1] Peters et al. 2018, NAACL, p. 2227, §1; p. 2231, §4 (Table 1)`. The practical lesson — that transferring a deep pretrained language model into downstream tasks unlocks large gains across the board — was the proximate cause of the year that followed: ULMFiT (Howard & Ruder 2018), GPT-1 (Radford et al. 2018), and BERT (Devlin et al. 2019), all of which replaced the LSTM with the Transformer `[T2] Jurafsky & Martin 3rd ed. draft, ch. 11, §11.1`.

The phrase "word embedding" survived this transition only as legacy vocabulary. By 2019, what production systems used was the input-token embedding *table inside a Transformer*, learned jointly with the rest of the model and contextualised by self-attention layers above it. The static-embedding artefacts — the GloVe `300d` files, the word2vec binaries — remain useful as fast features for low-budget pipelines, classroom demonstrations, and interpretability probes `[T2] Jurafsky & Martin 3rd ed. draft, ch. 6, §6.10`. They are no longer the frontier.

## 6. What the embeddings era actually settled

Three things, broadly. First, the distributional hypothesis is right enough to power production systems: most of what we call lexical semantics can be approximated by neighbourhoods in a co-occurrence space `[T1] Boleda 2022, arXiv:2205.07750, p. 1, abstract`. Second, count-based and predict-based methods are not different paradigms; with appropriate hyperparameters they extract the same signal, because SGNS factorises shifted PMI and GloVe does weighted log-co-occurrence regression `[T1] Levy & Goldberg 2014, NIPS 2014, p. 4, Theorem 1; Levy, Goldberg & Dagan 2015, TACL 3, pp. 211–225, §6`. Third, the sharp limit of static embeddings — one vector per word type — was both a popularisation gift (the analogy demo) and a fatal weakness (polysemy, bias amplification, fragility on the analogy task itself), and resolving it required moving the unit of representation from the type to the token-in-context. ELMo did that with LSTMs; the Transformer did it better, and the field has not looked back `[T1] Peters et al. 2018, NAACL, p. 2227, §1`.

## Sources

- `[T1]` Harris, Z.S. (1954). "Distributional Structure." *Word* 10(2/3): 146–162. Available via JSTOR; reprinted in Harris, *Papers in Structural and Transformational Linguistics*, 1970.
- `[T1]` Deerwester, S.; Dumais, S.T.; Furnas, G.W.; Landauer, T.K.; Harshman, R. (1990). "Indexing by Latent Semantic Analysis." *Journal of the American Society for Information Science* 41(6): 391–407. https://asistdl.onlinelibrary.wiley.com/doi/10.1002/(SICI)1097-4571(199009)41:6%3C391::AID-ASI1%3E3.0.CO;2-9
- `[T1]` Bengio, Y.; Ducharme, R.; Vincent, P.; Jauvin, C. (2003). "A Neural Probabilistic Language Model." *Journal of Machine Learning Research* 3: 1137–1155. https://www.jmlr.org/papers/volume3/bengio03a/bengio03a.pdf
- `[T1]` Mikolov, T.; Chen, K.; Corrado, G.; Dean, J. (2013a). "Efficient Estimation of Word Representations in Vector Space." arXiv:1301.3781. Submitted 16 Jan 2013, last revised 7 Sep 2013. https://arxiv.org/abs/1301.3781
- `[T1]` Mikolov, T.; Sutskever, I.; Chen, K.; Corrado, G.; Dean, J. (2013b). "Distributed Representations of Words and Phrases and their Compositionality." *Advances in Neural Information Processing Systems* 26 (NIPS 2013). arXiv:1310.4546. https://papers.nips.cc/paper/5021-distributed-representations-of-words-and-phrases-and-their-compositionality
- `[T1]` Goldberg, Y.; Levy, O. (2014). "word2vec Explained: Deriving Mikolov et al.'s Negative-Sampling Word-Embedding Method." arXiv:1402.3722. Submitted 15 Feb 2014. https://arxiv.org/abs/1402.3722
- `[T1]` Pennington, J.; Socher, R.; Manning, C.D. (2014). "GloVe: Global Vectors for Word Representation." *Proceedings of the 2014 Conference on Empirical Methods in Natural Language Processing* (EMNLP 2014), pp. 1532–1543. https://aclanthology.org/D14-1162/
- `[T1]` Levy, O.; Goldberg, Y. (2014). "Neural Word Embedding as Implicit Matrix Factorization." *Advances in Neural Information Processing Systems* 27 (NIPS 2014), pp. 2177–2185. https://papers.nips.cc/paper/5477-neural-word-embedding-as-implicit-matrix-factorization
- `[T1]` Levy, O.; Goldberg, Y. (2014b). "Linguistic Regularities in Sparse and Explicit Word Representations." *Proceedings of CoNLL 2014*, pp. 171–180. https://aclanthology.org/W14-1618/
- `[T1]` Levy, O.; Goldberg, Y.; Dagan, I. (2015). "Improving Distributional Similarity with Lessons Learned from Word Embeddings." *Transactions of the ACL* 3: 211–225. https://aclanthology.org/Q15-1016/
- `[T1]` Linzen, T. (2016). "Issues in Evaluating Semantic Spaces Using Word Analogies." *Proceedings of the 1st Workshop on Evaluating Vector-Space Representations for NLP* (RepEval @ NAACL 2016), pp. 13–18. https://aclanthology.org/W16-2503/
- `[T1]` Drozd, A.; Gladkova, A.; Matsuoka, S. (2016). "Word Embeddings, Analogies, and Machine Learning: Beyond king − man + woman = queen." *Proceedings of COLING 2016*, pp. 3519–3530. https://aclanthology.org/C16-1332/
- `[T1]` Bolukbasi, T.; Chang, K.-W.; Zou, J.; Saligrama, V.; Kalai, A. (2016). "Man is to Computer Programmer as Woman is to Homemaker? Debiasing Word Embeddings." *NIPS 2016*. arXiv:1607.06520. https://arxiv.org/abs/1607.06520
- `[T1]` Bojanowski, P.; Grave, E.; Joulin, A.; Mikolov, T. (2017). "Enriching Word Vectors with Subword Information." *Transactions of the ACL* 5: 135–146. https://aclanthology.org/Q17-1010/
- `[T1]` Peters, M.E.; Neumann, M.; Iyyer, M.; Gardner, M.; Clark, C.; Lee, K.; Zettlemoyer, L. (2018). "Deep Contextualized Word Representations." *Proceedings of NAACL-HLT 2018*, pp. 2227–2237. https://aclanthology.org/N18-1202/
- `[T1]` Boleda, G. (2022). "What company do words keep? Revisiting the distributional semantics of J.R. Firth & Zellig Harris." arXiv:2205.07750. https://arxiv.org/abs/2205.07750
- `[T2]` Jurafsky, D.; Martin, J.H. *Speech and Language Processing*, 3rd ed. draft (chs. 6 "Vector Semantics and Embeddings" and 11 "Transfer Learning with Pretrained Language Models"). https://web.stanford.edu/~jurafsky/slp3/
- `[T2]` Wikipedia, "John Rupert Firth." https://en.wikipedia.org/wiki/John_Rupert_Firth
- `[T2]` Wikipedia, "Latent semantic analysis." https://en.wikipedia.org/wiki/Latent_semantic_analysis
- `[T2]` Wikipedia, "GloVe." https://en.wikipedia.org/wiki/GloVe
- `[T3]` Levy, O. (2014). "Neural Word Embeddings as Implicit Matrix Factorization" (blog post). https://levyomer.wordpress.com/2014/09/10/neural-word-embeddings-as-implicit-matrix-factorization/
