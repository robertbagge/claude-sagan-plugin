# Tokenization & Subword Units

> *"Tokenization is at the heart of much weirdness of LLMs. Do not brush it off."* — Andrej Karpathy [T3]

A neural language model never sees text. It sees a sequence of integers. The function that converts a UTF-8 string into those integers — and back — is the tokenizer. For most of NLP's deep-learning era, this preprocessing step has been treated as plumbing. The last decade has shown that it is not plumbing: it is a structural choice that bounds what models can spell, count, and compose. This dossier traces the arc from word-level vocabularies to byte-level patches, with attention to the algorithms, their pathologies, and the current push to delete the tokenizer altogether.

## 1. Word-Level Tokenization and the OOV Wall

Classical statistical NLP treated the *word* as the atomic unit. A vocabulary `V` was extracted from training data, anything outside `V` collapsed into a single `<UNK>` token, and downstream models worked over `|V|`-dimensional one-hot or embedding vectors. This worked tolerably for n-gram language models and feature-engineered classifiers but broke down for two reasons that became existential once neural sequence-to-sequence models arrived in 2014.

First, the **out-of-vocabulary (OOV) problem**. Natural language is Zipfian: the long tail of named entities, morphological variants, typos, code, URLs, and loanwords means that any fixed `V` will lose information at inference time. Sennrich, Haddow & Birch open their 2016 ACL paper with exactly this observation, noting that NMT systems "typically operate with a fixed vocabulary, but translation is an open-vocabulary problem" [T1] Sennrich et al. 2016, p. 1, ¶1, https://aclanthology.org/P16-1162/.

Second, the **softmax cost**. The output layer of a neural LM is a `|V|`-way classifier. Pushing `|V|` to 500k–1M words to reduce OOV blew up parameter counts and training time. Hierarchical softmax and sampled softmax were band-aids, not cures.

Two limit cases bracketed the design space. **Character-level models** (Sutskever, Martens & Hinton 2011 RNNs; Kim et al. 2016 character-CNN inputs) eliminate OOV entirely — the alphabet is finite — but pay for it with sequence lengths 4–6× longer than word sequences and a heavier burden on the model to learn morphology from scratch [T1] Kim et al. 2016, *Character-Aware Neural Language Models*, https://arxiv.org/abs/1508.06615. **Word-level** models go the other way and lose the tail. The interesting design space is in the middle, and it is the space subword tokenization occupies.

## 2. Byte Pair Encoding: A 1994 Compression Trick Reborn for NMT

The algorithm that became the workhorse of modern LLM tokenization was originally a data-compression scheme. Philip Gage published "A New Algorithm for Data Compression" in the February 1994 *C Users Journal*, describing a routine that "compresses data by finding the most frequently occurring pairs of adjacent bytes in the data and replacing all instances of the pair with a byte that was not in the original data," repeating until no profitable pair remains [T2] Gage 1994, *C Users Journal* Vol 12 Issue 2, pp. 23–38, http://www.pennelynn.com/Documents/CUJ/HTML/94HTML/19940045.HTM. Gage's pitch was modest: BPE achieved compression close to LZW with a "small, fast expansion routine, ideal for applications with limited memory." For 22 years it sat in the compression-folklore bin.

Sennrich, Haddow & Birch (Edinburgh, ACL 2016) repurposed it. Instead of compressing bytes, they applied the merge loop to *characters of words in a training corpus*, building a vocabulary of variable-length subword units that could "represent an open vocabulary through a fixed-size vocabulary" [T1] Sennrich et al. 2016, p. 1, ¶3, https://aclanthology.org/P16-1162/. The training procedure:

1. Initialize the vocabulary with the character set plus an end-of-word marker `</w>`.
2. Count all adjacent symbol pairs across the training corpus.
3. Merge the most frequent pair into a new symbol; add it to the vocabulary.
4. Repeat for `k` merges. Final vocabulary size = base alphabet + `k`.

For English-German WMT15 they reported a +1.1 BLEU gain over a back-off-dictionary baseline; for English-Russian +1.3 BLEU [T1] Sennrich et al. 2016, p. 1, abstract. The Edinburgh group released `subword-nmt` on GitHub [T2] https://github.com/rsennrich/subword-nmt; within two years the technique was the default for NMT and was being adopted by the early Transformer LMs.

A worked example. Take the corpus `low low low low low lowest newer newer newer wider wider`. Initial pre-tokenized form (with `</w>` end-of-word markers): `l o w </w>` × 5, `l o w e s t </w>` × 1, `n e w e r </w>` × 3, `w i d e r </w>` × 2. The most frequent adjacent pair across the corpus is `(e, r</w>)` at 5 occurrences; merge to `er</w>`. Next: `(l, o)` at 6; merge to `lo`. Next: `(lo, w)` at 6; merge to `low`. After enough merges, common words like `low</w>` collapse to a single token while a rare word like `lowest</w>` remains a sequence (`low`, `est</w>`). New unseen words like `lowering</w>` become `low + er + ing</w>` — never `<UNK>`.

## 3. WordPiece, Unigram LM, and SentencePiece

Three variations on the subword theme followed, each fixing a different limitation.

**WordPiece** predates the BPE NMT paper. Schuster & Nakajima introduced it for Google's Japanese and Korean voice-search system at ICASSP 2012, building a ~22k-unit inventory from basic Unicode characters and growing it via a likelihood-greedy merge criterion rather than raw frequency [T1] Schuster & Nakajima 2012, *Japanese and Korean Voice Search*, ICASSP, https://research.google/pubs/japanese-and-korean-voice-search/. Where BPE merges the *most frequent* pair, WordPiece merges the pair that *most increases the training-data likelihood under a unigram LM* — `score(a,b) = freq(ab) / (freq(a) · freq(b))`. WordPiece was adopted by BERT (Devlin et al. 2019) with a 30k vocabulary and is the reason BERT tokens carry the iconic `##` continuation prefix [T1] Devlin et al. 2019, *BERT*, NAACL, https://aclanthology.org/N19-1423/.

**Unigram LM tokenization** (Kudo 2018) inverts the bottom-up merge approach. Start with a *large* candidate vocabulary (often built via suffix array or BPE), assign each token a probability under a unigram LM trained by EM, then iteratively *prune* low-probability tokens until a target size is reached. The Viterbi algorithm decodes the most probable segmentation [T1] Kudo 2018, *Subword Regularization*, ACL, p. 2, ¶3, https://aclanthology.org/P18-1007/. The big payoff is **sampling**: because the unigram LM defines a distribution over segmentations, training can stochastically resample tokenizations of the same input each epoch, acting as a regularizer. Kudo reports +1–2 BLEU on low-resource and out-of-domain pairs.

**SentencePiece** (Kudo & Richardson, EMNLP 2018) is not really a new algorithm — it implements both BPE and Unigram LM — but it is the engineering deliverable that made tokenization end-to-end. Prior tools assumed pre-tokenized whitespace-segmented input (and therefore couldn't tokenize Chinese, Japanese, Thai cleanly). SentencePiece "trains subword models directly from raw sentences, which allows for a purely end-to-end and language-independent system" [T1] Kudo & Richardson 2018, *SentencePiece*, EMNLP, p. 66, ¶1, https://aclanthology.org/D18-2012/. The trick: treat whitespace as just another character, replaced by `▁` (U+2581) so that detokenization is reversible by joining tokens and replacing `▁` with space. SentencePiece is the tokenizer behind T5, ALBERT, XLNet, mBART, LLaMA, and most non-OpenAI open models [T2] https://github.com/google/sentencepiece.

## 4. Byte-Level BPE and the GPT Lineage

Even with subwords, two pre-tokenization decisions still leak: how to handle bytes outside the training character set (Cyrillic in an English-trained model, emoji, control codes), and what counts as a "word boundary" before BPE runs. GPT-2 (Radford et al. 2019) cut the knot by running BPE *over bytes* rather than Unicode characters [T1] Radford et al. 2019, *Language Models are Unsupervised Multitask Learners*, OpenAI tech report, https://cdn.openai.com/better-language-models/language_models_are_unsupervised_multitask_learners.pdf. The base alphabet becomes the 256 bytes; any UTF-8 string is representable; nothing is OOV. The resulting `gpt2` tokenizer has 50,257 entries (256 byte units + 50,000 merges + 1 `<|endoftext|>`) [T2] OpenAI tiktoken, https://github.com/openai/tiktoken.

GPT-2 also introduced the now-famous pre-tokenization regex that splits on contractions, letter runs, digit runs, and whitespace before BPE merges run, ensuring that `don't` tokenizes as `don` + `'t` rather than learning a `don'` merge:

```
's|'t|'re|'ve|'m|'ll|'d| ?\p{L}+| ?\p{N}+| ?[^\s\p{L}\p{N}]+|\s+(?!\S)|\s+
```

GPT-3.5/GPT-4 use `cl100k_base` (~100k vocabulary); GPT-4o uses `o200k_base` (~200k) [T2] Modal, *What is o200k Harmony*, https://modal.com/blog/what-is-o200k-harmony. The doubling halved tokens-per-text on many languages, directly cutting per-call cost.

**The same string across schemes.** Take the text `Tokenization isn't trivial.` (28 bytes):

| Scheme | Tokens | Notes |
|---|---|---|
| Char-level | `T`,`o`,`k`,`e`,`n`,`i`,`z`,`a`,`t`,`i`,`o`,`n`,` `,`i`,`s`,`n`,`'`,`t`,` `,`t`,`r`,`i`,`v`,`i`,`a`,`l`,`.` (27) | OOV-free, sequence is long. |
| BERT WordPiece (uncased, 30k) | `token`, `##ization`, `isn`, `'`, `t`, `trivial`, `.` (7) | `##` marks intra-word continuation. |
| GPT-2 BPE (byte-level, 50k) | `Token`, `ization`, ` isn`, `'t`, ` trivial`, `.` (6) | Leading-space variants are distinct tokens. |
| GPT-4 cl100k_base (~100k) | `Token`, `ization`, ` isn`, `'t`, ` trivial`, `.` (6) | Same boundaries; richer merges show on longer inputs. |
| GPT-4o o200k_base (~200k) | `Tokenization`, ` isn`, `'t`, ` trivial`, `.` (5) | Larger vocab swallows the whole word. |
| ByT5 (UTF-8 bytes, ~256+special) | 28 byte tokens | No tokenizer; raw bytes. |

The leading-space behavior — where ` trivial` and `trivial` are *different* tokens — is a frequent source of subtle prompting bugs and explains why "stop sequences" sometimes need a leading space.

## 5. Tokenizer Pathologies

Once trillions of tokens of training data flow through these schemes, the cracks show.

**Glitch tokens / SolidGoldMagikarp.** In January 2023 Jessica Rumbelow and Matthew Watkins, working in the SERI-MATS alignment program, were clustering GPT-2/3 token embeddings and noticed a band of tokens whose embedding vectors clustered near the *centroid* of the embedding space — i.e., tokens the model had seen in training but had effectively never updated weights for. Asking text-davinci-003 to repeat the string `SolidGoldMagikarp` produced `Distribute`; asking it to repeat ` petertodd` produced apocalyptic gibberish; many tokens triggered refusal or evasive output [T3] Rumbelow & Watkins 2023, *SolidGoldMagikarp (plus, prompt generation)*, Alignment Forum, https://www.alignmentforum.org/posts/aPeJE8bSo6rAFoLqg/solidgoldmagikarp-plus-prompt-generation. Their proposed mechanism: the BPE tokenizer was trained on a *different* (larger, scraped) corpus than the model itself. Tokens like `SolidGoldMagikarp` (a Reddit username), `BuyableInstoreAndOnline` (an e-commerce template field), or `DeliveryDate` (log-file scaffolding) earned merges during tokenizer training but were filtered out before LM training, leaving "ghost" embeddings the model never grounded [T3] Wikipedia, *Glitch token*, https://en.wikipedia.org/wiki/Glitch_token. Most of these glitches dissolved when newer tokenizers (cl100k_base, o200k_base) re-learned merges from cleaner corpora, but the pattern recurs: any tokenizer trained independently of the model will accumulate untrained tokens [T3] LessWrong, *SolidGoldMagikarp III: Glitch token archaeology*, https://www.lesswrong.com/posts/8viQEp8KBg2QSW4Yc/solidgoldmagikarp-iii-glitch-token-archaeology.

**Numbers and arithmetic.** BPE has no notion of place value. The number `2249` tokenizes (cl100k_base) as `2`,`249`; `2250` as `22`,`50`; `2251` as `225`,`1` [T3] Beren Millidge, *Integer tokenization is insane*, https://www.beren.io/2023-02-04-Integer-tokenization-is-insane/. The model must learn arithmetic across these arbitrary chunkings, and Singh & Strouse (2024) show that forcing right-to-left digit grouping (e.g., adding commas) measurably improves GPT-3.5/GPT-4 addition accuracy, with errors spiking exactly when tokenizer alignment between operands and answer breaks [T1] Singh & Strouse 2024, *Tokenization counts*, arXiv 2402.14903, p. 1, ¶1, https://arxiv.org/abs/2402.14903. LLaMA and PaLM sidestep this by hard-coding single-digit tokenization; modern OpenAI tokenizers split numbers into 1–3 digit chunks, which is better than nothing but still inconsistent.

**Other failure modes** documented in the wild: case sensitivity inflating vocabulary (`The` and `the` as separate tokens), whitespace boundary collisions (` Hello` vs `Hello`), code tokenization that splits identifiers awkwardly, and locale-specific failures where multilingual text fragments into byte-level sequences and balloons context length. Karpathy's 2024 video "Let's build the GPT Tokenizer" enumerates these and is the canonical educational reference [T3] Karpathy, https://github.com/karpathy/minbpe.

## 6. The Tokenizer-Free Counter-Movement

The accumulating pathologies have driven a serious push to remove the tokenizer.

**ByT5** (Xue et al., TACL 2022) is the cleanest demonstration that a vanilla Transformer can operate on raw UTF-8 bytes with a vocabulary of ~256 + a few special tokens, no SentencePiece in sight [T1] Xue et al. 2022, *ByT5*, TACL Vol 10, pp. 291–306, https://aclanthology.org/2022.tacl-1.17/. The trade-off is brutal: byte sequences are 4–5× longer than subword sequences, so attention cost goes up quadratically and training/inference both slow. ByT5 compensates by re-balancing parameters into the encoder. The wins are real — substantial robustness gains on noisy/spelling-perturbed inputs and on tasks sensitive to character-level form (transliteration, morphological inflection) — but the compute cost kept it in research-curio territory.

**Charformer** (Tay et al. 2022) and **CANINE** (Clark et al. 2022) tried to have it both ways: ingest characters/bytes but learn dynamic, soft pooling into "subword-like" units inside the model. They worked but didn't unseat SentencePiece in practice [T1] Tay et al. 2022, *Charformer*, ICLR, https://arxiv.org/abs/2106.12672.

**MEGABYTE** (Yu et al., NeurIPS 2023) reframed the problem: don't fight the long sequence, *patch it*. MEGABYTE chunks bytes into fixed-size patches (e.g., 8 bytes), runs a heavy global Transformer over patch embeddings, and a small local Transformer over bytes within each patch. This gives sub-quadratic attention, fatter feedforwards per FLOP, and parallel decoding [T1] Yu et al. 2023, *MEGABYTE*, NeurIPS, p. 1, abstract, https://arxiv.org/abs/2305.07185. They showed byte-level models could compete with subword models on long-context language modeling and set SOTA on ImageNet density estimation and raw-audio modeling.

**MambaByte** (Wang et al., COLM 2024) replaced the patch trick with a state-space model (Mamba/S6), whose fixed-size hidden state handles long byte sequences with linear-time decode. They report MambaByte "is competitive with, and even outperforms, state-of-the-art subword Transformers on language modeling" while keeping the byte-level robustness benefits, plus a 2.6× speedup via tokenized speculative drafting + byte-level verification [T1] Wang et al. 2024, *MambaByte*, COLM, p. 1, abstract, https://arxiv.org/abs/2401.13660.

**Byte Latent Transformer (BLT)** (Pagnoni et al., Meta, December 2024 → ACL 2025) is the most recent and arguably most credible candidate to actually replace SentencePiece in production. The key idea: don't use *fixed* patches; **dynamically segment bytes by next-byte entropy**. A small auxiliary byte-LM scores every position; high-entropy bytes (typically the start of a word, where prediction is hard) start new patches, low-entropy bytes (continuations of predictable strings) get folded into ongoing patches [T1] Pagnoni et al. 2024, *Byte Latent Transformer*, arXiv 2412.09871, p. 2, ¶2, https://arxiv.org/abs/2412.09871. Compute is allocated where the data is hard. Pagnoni et al. ran the first FLOP-controlled byte-level scaling study up to 8B parameters / 4T training bytes and showed BLT *matches* tokenization-based LLMs at scale while improving inference efficiency and robustness. Meta open-sourced training/inference code [T2] https://github.com/facebookresearch/blt.

If BLT-style entropy patching pans out, the long-promised end of the tokenizer may finally arrive — not by deleting it, but by making it a learned, adaptive, model-internal component.

## 7. Synthesis

The tokenization story has the shape of many engineering arcs in ML: a workable hack (word-level + UNK) → a clever borrow from another field (BPE from compression) → a generation of refinements (WordPiece, Unigram LM, SentencePiece, byte-level BPE) → diagnosis of accumulated pathologies (glitch tokens, arithmetic failures) → an attempt to delete the abstraction entirely (ByT5, MEGABYTE, MambaByte, BLT). The current state of practice is split: production LLMs (GPT-4o, Claude, Gemini, LLaMA) all still ship with BPE/SentencePiece tokenizers because the FLOP-efficiency of pre-tokenized inputs is hard to beat, while the research frontier has demonstrated — for the first time at 8B scale with BLT — that byte-level models can match them.

The likely 2026–2028 trajectory: hybrid systems where a learned, dynamic byte-grouping front-end replaces the static BPE merges, models become natively multilingual without per-language vocabulary engineering, and the long tail of glitch tokens and arithmetic miscounts attributable to fixed tokenizers becomes a historical curiosity — replaced, no doubt, by a fresh set of pathologies attributable to whatever the new abstraction is.

## Sources

- [T1] Sennrich, R., Haddow, B., & Birch, A. (2016). *Neural Machine Translation of Rare Words with Subword Units.* Proceedings of ACL 2016, pp. 1715–1725. https://aclanthology.org/P16-1162/ (arXiv: https://arxiv.org/abs/1508.07909)
- [T1] Kim, Y., Jernite, Y., Sontag, D., & Rush, A. M. (2016). *Character-Aware Neural Language Models.* AAAI 2016. https://arxiv.org/abs/1508.06615
- [T1] Schuster, M., & Nakajima, K. (2012). *Japanese and Korean Voice Search.* ICASSP 2012. https://research.google/pubs/japanese-and-korean-voice-search/
- [T1] Devlin, J., Chang, M.-W., Lee, K., & Toutanova, K. (2019). *BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding.* NAACL 2019. https://aclanthology.org/N19-1423/
- [T1] Kudo, T. (2018). *Subword Regularization: Improving Neural Network Translation Models with Multiple Subword Candidates.* ACL 2018. https://aclanthology.org/P18-1007/
- [T1] Kudo, T., & Richardson, J. (2018). *SentencePiece: A simple and language independent subword tokenizer and detokenizer for Neural Text Processing.* EMNLP 2018 System Demonstrations, pp. 66–71. https://aclanthology.org/D18-2012/
- [T1] Radford, A., Wu, J., Child, R., Luan, D., Amodei, D., & Sutskever, I. (2019). *Language Models are Unsupervised Multitask Learners.* OpenAI technical report. https://cdn.openai.com/better-language-models/language_models_are_unsupervised_multitask_learners.pdf
- [T1] Xue, L., Barua, A., Constant, N., Al-Rfou, R., Narang, S., Kale, M., Roberts, A., & Raffel, C. (2022). *ByT5: Towards a Token-Free Future with Pre-trained Byte-to-Byte Models.* Transactions of ACL, Vol. 10, pp. 291–306. https://aclanthology.org/2022.tacl-1.17/
- [T1] Tay, Y. et al. (2022). *Charformer: Fast Character Transformers via Gradient-based Subword Tokenization.* ICLR 2022. https://arxiv.org/abs/2106.12672
- [T1] Yu, L., Simig, D., Flaherty, C., Aghajanyan, A., Zettlemoyer, L., & Lewis, M. (2023). *MEGABYTE: Predicting Million-byte Sequences with Multiscale Transformers.* NeurIPS 2023. https://arxiv.org/abs/2305.07185
- [T1] Wang, J., Gangavarapu, T., Yan, J. N., & Rush, A. M. (2024). *MambaByte: Token-free Selective State Space Model.* COLM 2024. https://arxiv.org/abs/2401.13660
- [T1] Pagnoni, A. et al. (2024). *Byte Latent Transformer: Patches Scale Better Than Tokens.* arXiv 2412.09871; ACL 2025. https://arxiv.org/abs/2412.09871
- [T1] Singh, A. K., & Strouse, D. J. (2024). *Tokenization counts: the impact of tokenization on arithmetic in frontier LLMs.* arXiv 2402.14903. https://arxiv.org/abs/2402.14903
- [T2] Gage, P. (February 1994). *A New Algorithm for Data Compression.* The C Users Journal, Vol. 12 Issue 2, pp. 23–38. http://www.pennelynn.com/Documents/CUJ/HTML/94HTML/19940045.HTM
- [T2] OpenAI. *tiktoken (cl100k_base, o200k_base).* GitHub repository. https://github.com/openai/tiktoken
- [T2] Sennrich, R. *subword-nmt reference implementation.* GitHub. https://github.com/rsennrich/subword-nmt
- [T2] Google. *SentencePiece reference implementation.* GitHub. https://github.com/google/sentencepiece
- [T2] Meta AI Research. *Byte Latent Transformer (BLT) code release.* GitHub. https://github.com/facebookresearch/blt
- [T2] Modal. *What is o200k Harmony? OpenAI's latest tiktoken edition.* https://modal.com/blog/what-is-o200k-harmony
- [T3] Rumbelow, J., & Watkins, M. (2023). *SolidGoldMagikarp (plus, prompt generation).* Alignment Forum, Feb 2023. https://www.alignmentforum.org/posts/aPeJE8bSo6rAFoLqg/solidgoldmagikarp-plus-prompt-generation
- [T3] *SolidGoldMagikarp III: Glitch token archaeology.* LessWrong. https://www.lesswrong.com/posts/8viQEp8KBg2QSW4Yc/solidgoldmagikarp-iii-glitch-token-archaeology
- [T3] *Glitch token.* Wikipedia. https://en.wikipedia.org/wiki/Glitch_token
- [T3] Millidge, B. (2023). *Integer tokenization is insane.* https://www.beren.io/2023-02-04-Integer-tokenization-is-insane/
- [T3] Karpathy, A. (2024). *Let's build the GPT Tokenizer* (video) and minbpe repository. https://github.com/karpathy/minbpe
- [T3] fast.ai (2025). *Let's Build the GPT Tokenizer: A Complete Guide to Tokenization in LLMs* (book chapter from Karpathy video). https://www.fast.ai/posts/2025-10-16-karpathy-tokenizers
