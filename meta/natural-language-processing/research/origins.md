# Origins of NLP (1950s–1960s)

The computational era of natural language processing did not begin with linguists. It began with cryptographers, electrical engineers, and a mathematician at the Rockefeller Foundation who thought Russian looked like a coded form of English. In the 17 years between Warren Weaver's 1949 *Translation* memorandum and the 1966 ALPAC report, NLP was invented, oversold, and — at least in its dominant funding stream — essentially shut down. Understanding that arc is understanding why every subsequent generation of language technology has had to argue against the ghost of an earlier failure.

This dossier traces five threads: Turing's framing of the problem, Shannon and Weaver's information-theoretic seeding of the field, the Georgetown–IBM demonstration and the optimism it manufactured, Chomsky's symbolic counter-revolution, and the Bar-Hillel / ALPAC reckoning that ended the era.

## 1. Turing 1950: The Problem Gets a Test

Alan Turing's *Computing Machinery and Intelligence*, published in *Mind* (vol. LIX, no. 236, October 1950, pp. 433–460), is where the question of machine language behaviour was given an operational shape [T1] Turing 1950, *Mind* 59, p. 433. Turing opens by arguing that the question "Can machines think?" is too ill-defined to attack, and proposes replacing it with the Imitation Game: a teletype-mediated conversation in which an interrogator must distinguish a human from a machine on the basis of textual responses alone [T1] Turing 1950, *Mind* 59, pp. 433–434, ¶1–2.

Two features of the paper matter for NLP specifically. First, it locates intelligence in *language behaviour* — the medium of the test is text — which made natural language the de facto benchmark for any future "thinking machine" [T2] Wikipedia, "Computing Machinery and Intelligence," https://en.wikipedia.org/wiki/Computing_Machinery_and_Intelligence. Second, the paper's final section on "learning machines" sketches a child-machine that is *trained* rather than programmed, with mutations, rewards, and selection — a recognisable ancestor of statistical and gradient-based learning, written 36 years before backpropagation became famous [T2] Turing 1950, *Mind* 59, §7 "Learning Machines," pp. 454–460.

Turing's specific 50-year prediction — that by ~2000, computers with ~10⁹ bits of storage would fool an average interrogator 30% of the time after five minutes — is often cited and was, on its narrow terms, plausibly met by chatbots well before the LLM era [T1] Turing 1950, *Mind* 59, p. 442. But the paper's lasting contribution was rhetorical: it gave the field a target. Every later NLP system would, implicitly or explicitly, be measured against the question "could this hold up in an Imitation Game?"

## 2. Shannon: Language as a Stochastic Source

While Turing was framing the philosophical question, Claude Shannon was quantising it. *A Mathematical Theory of Communication* (BSTJ vol. 27, July and October 1948) treated any source — including a writer of English — as a stochastic process emitting symbols, and introduced n-gram approximations to English as a worked example [T1] Shannon 1948, BSTJ 27, pp. 379–423 and 623–656. Shannon's increasingly higher-order approximations to English (zero-order random letters, then letter unigrams, bigrams, trigrams, then word unigrams and bigrams) directly anticipated the n-gram language models that dominated speech and translation work into the 2010s [T2] Wikipedia, "Entropy (information theory)," https://en.wikipedia.org/wiki/Entropy_(information_theory).

Shannon followed up with *Prediction and Entropy of Printed English* (BSTJ vol. 30, January 1951, pp. 50–64), which estimated the per-letter entropy of English by having human subjects guess the next character of a text — an early human-in-the-loop language model evaluation [T1] Shannon 1951, BSTJ 30, pp. 50–64. The headline number — that English is roughly 50% redundant when statistical structure is taken over runs of about eight letters — was a quantitative anchor that every subsequent compression and language-model paper had to reckon with [T2] Wikipedia, "Entropy (information theory)," https://en.wikipedia.org/wiki/Entropy_(information_theory).

Two things about Shannon's contribution are easy to miss. First, he did this work at Bell Labs in an electrical engineering frame, not a linguistic one — which planted the *stochastic paradigm* of NLP firmly outside linguistics departments [T2] Jurafsky & Martin, *Speech and Language Processing*, ch. 1 "Historical Notes." Second, his cryptographic work during World War II (declassified as *Communication Theory of Secrecy Systems*, 1949) was the bridge from message-passing into language: it suggested treating an unknown language as ciphertext over a known language, an idea Warren Weaver immediately seized on.

## 3. Weaver's 1949 Memorandum: NLP Before NLP

In July 1949, Warren Weaver — director of the natural sciences division at the Rockefeller Foundation, and during the war a senior administrator of US scientific R&D — circulated a memorandum titled simply *Translation* to about 200 colleagues. It is the document most historians treat as the founding charter of machine translation, and by extension of NLP [T2] Hutchins, "Warren Weaver memorandum, July 1949," https://aclanthology.org/www.mt-archive.info/90/MTNI-1999-Hutchins.pdf.

The memo's most-quoted line — sent in a March 1947 letter to Norbert Wiener and reprised in 1949 — is the cryptographic conjecture: "When I look at an article in Russian, I say: 'This is really written in English, but it has been coded in some strange symbols. I will now proceed to decode'" [T2] Wikipedia, "Warren Weaver," https://en.wikipedia.org/wiki/Warren_Weaver. The memo proposed four lines of attack:

1. **Context windows** for word-sense disambiguation — look at neighbouring words to resolve ambiguity. (This is, in retrospect, an n-gram intuition.)
2. **Logical / formal-language methods**, drawing on McCulloch & Pitts's 1943 paper on neural nets and Carnap's logical syntax.
3. **Cryptographic methods**, leveraging Shannon's information-theoretic apparatus directly.
4. **Language universals** — a conjecture that all human languages share a "common basement" of structure, anticipating both Chomskyan universals and modern multilingual embeddings [T2] Hutchins 1999, ¶4–6; [T2] Wikipedia, "Warren Weaver."

Weaver's memo did three things at once: it framed translation as a tractable computational problem, it gave researchers a menu of methods, and — crucially, given Weaver's policy influence — it triggered funding. By 1952, MIT was hosting the first conference on machine translation, and US government money started flowing into Georgetown, MIT, RAND, the University of Washington, and Harvard [T2] Hutchins, "First public demonstration of machine translation," https://open.unive.it/hitrade/books/HutchinsFirst.pdf.

## 4. Georgetown–IBM 1954: The Demo That Set the Trap

On 7 January 1954, in IBM's New York headquarters, an IBM 701 mainframe translated more than 60 Russian sentences into English in front of journalists. The system had a vocabulary of 250 stems and endings, and six grammatical rules [T2] Wikipedia, "Georgetown–IBM experiment," https://en.wikipedia.org/wiki/Georgetown%E2%80%93IBM_experiment; [T2] Hutchins, "The Georgetown-IBM experiment demonstrated in January 1954," ACL Anthology AMTA-2004, https://aclanthology.org/2004.amta-papers.12/. The project was led by Léon Dostert at Georgetown — the linguist who had set up simultaneous interpretation at the Nuremberg trials — and Cuthbert Hurd at IBM [T2] Hutchins 2004, AMTA paper, §1.

The demo sentences were carefully chosen. Most were short statements about organic chemistry; a smaller set were "general interest" sentences. A widely reproduced example: *Mi pyeryedayem mislyi posryedstvom ryechyi* → "We transmit thoughts by means of speech" [T2] Wikipedia, "Georgetown–IBM experiment." The system was, in modern terms, a hand-engineered demo with a tiny lexicon and almost no grammar — closer to a stage-magic act than a translator.

The press took it at face value. IBM's own press release claimed that "within three or five years, machine translation could well be a solved problem," and the Georgetown team echoed the timeline [T2] Wikipedia, "Georgetown–IBM experiment"; [T2] *MultiLingual* magazine, "Seventy Years of Machine Translation," May 2024, https://multilingual.com/magazine/may-2024/seventy-years-of-machine-translation/. Soviet researchers, alarmed, started their own MT programme within months. The US Department of Defense, the CIA, and the National Science Foundation poured money into the field. By the mid-1950s, MT was the de facto first practical application of the new "electronic brain."

The trap, in hindsight, is obvious: Georgetown–IBM was an existence proof for *toy* MT, not a roadmap for *general* MT. The 250-word vocabulary and six rules were exhausted by the demo set; scaling required exponentially more linguistic engineering, which the field did not yet know how to do.

## 5. Chomsky 1956–1957: The Symbolic Turn

While statisticians at Bell Labs and IBM were treating language as a Markov source, a 28-year-old MIT linguist was about to argue that this approach was, on principled grounds, doomed. Noam Chomsky's *Three Models for the Description of Language* appeared in IRE Transactions on Information Theory IT-2 (September 1956), pp. 113–124 — pointedly, in an information-theory journal [T1] Chomsky 1956, IRE Trans. Info. Theory IT-2(3), pp. 113–124, https://chomsky.info/wp-content/uploads/195609-.pdf.

The paper's core result: no finite-state Markov process can serve as an English grammar, and n-gram approximations do not converge to grammaticality as n grows [T1] Chomsky 1956, p. 113, ¶1; p. 115, ¶3. The argument used long-distance dependencies (e.g. *if-then* and *either-or* nested across arbitrary numbers of intervening words) that finite-state machines cannot track. Chomsky proposed phrase-structure grammars — context-free rewriting systems — as more adequate, and transformational grammars as more illuminating still [T1] Chomsky 1956, §3–§4.

A year later, *Syntactic Structures* (Mouton, 1957) made the case to a wider audience and introduced what became the Chomsky hierarchy: regular ⊂ context-free ⊂ context-sensitive ⊂ unrestricted [T2] Chomsky, *Syntactic Structures*, 1957, ch. 3, https://www.ling.upenn.edu/courses/ling5700/Chomsky1957.pdf. The book's "colourless green ideas sleep furiously" — grammatical but meaningless — and "furiously sleep ideas green colourless" — neither — made the point that grammaticality is structural, not statistical [T2] Chomsky 1957, ch. 2, p. 15.

The effect on NLP was bipolar. On one hand, Chomsky's hierarchy became the bedrock of *programming language* parsing; Donald Knuth credited *Syntactic Structures* with shaping his thinking on compilers [T2] Wikipedia, "Syntactic Structures," https://en.wikipedia.org/wiki/Syntactic_Structures. Context-free grammars and parsing algorithms (CKY, Earley) flowed directly out of this lineage. On the other hand, Chomsky's strong claim that statistical methods were inadequate in principle pushed mainstream linguistic NLP into a *symbolic* paradigm — hand-written grammars and rules — that would dominate for the next 30 years. As Jurafsky & Martin put it, by the early 1960s the field had "split very cleanly into two paradigms: symbolic and stochastic" [T2] Jurafsky & Martin, *Speech and Language Processing* (2nd ed.), ch. 1 "Historical Notes," https://web.stanford.edu/~jurafsky/slp3/.

This split is the original sin of NLP's first 50 years. The symbolic camp had Chomsky's prestige and the elegance of formal grammar; the statistical camp had the engineers, the data, and — eventually — the results. They would not really reconcile until the IBM Candide MT system in the late 1980s and the deep-learning revolution in the 2010s.

## 6. Bar-Hillel 1960: The Insider Calls It Off

Yehoshua Bar-Hillel — the first full-time MT researcher in the world, hired by MIT in 1951 — published *The Present Status of Automatic Translation of Languages* in *Advances in Computers*, vol. 1 (1960), pp. 91–163 [T1] Bar-Hillel 1960, *Advances in Computers* 1, pp. 91–163, https://aclanthology.org/www.mt-archive.info/Bar-Hillel-1960.pdf. It was an obituary written by a believer.

Bar-Hillel's central claim: the goal of "fully automatic high quality translation" (FAHQT) — translation indistinguishable from a competent human translator — was not merely hard but *impossible in principle* with the methods then available, because translation requires real-world knowledge that no MT system possessed or had a path to acquiring [T1] Bar-Hillel 1960, §"The Unreasonableness of Aiming at Fully Automatic High Quality Translation," p. 91 onward; [T2] *The Future of MT is Now and Bar-Hillel was (almost entirely) Right*, BISFAI-95, https://cdn.aaai.org/BISFAI/1995/BISFAI95-013.pdf.

The illustration that survived him is a tiny passage:

> Little John was looking for his toy box. Finally he found it. The box was in the pen.

To translate "pen" correctly into a language that distinguishes a writing instrument from an enclosure, the system has to know that toy boxes don't fit inside writing instruments — which is to say, it needs an encyclopaedic store of facts about the physical world [T1] Bar-Hillel 1960, appendix; [T2] Wikipedia, "Yehoshua Bar-Hillel," https://en.wikipedia.org/wiki/Yehoshua_Bar-Hillel. Bar-Hillel argued no such store existed, none was on the horizon, and hand-coding one was infeasible. Decades later, AAAI papers were still observing that Google Translate failed on this exact sentence [T2] BISFAI-95, ¶6.

Bar-Hillel was not telling the field to give up on language technology — he was telling it to give up on FAHQT specifically and pursue *partly mechanised* high-quality translation with humans in the loop. The funders, however, drew a simpler conclusion.

## 7. ALPAC 1966: The Funding Cliff

In April 1964, the Automatic Language Processing Advisory Committee (ALPAC) was set up by the National Academy of Sciences at the request of the principal MT funders — the Department of Defense, the CIA, and the National Science Foundation. It was chaired by John R. Pierce of Bell Telephone Laboratories. Its seven members included psychologist John B. Carroll, linguists Eric Hamp and Charles Hockett, MT researchers David Hays and Anthony Oettinger, and AI pioneer Alan Perlis [T2] Wikipedia, "ALPAC," https://en.wikipedia.org/wiki/ALPAC; [T2] Hutchins, "ALPAC: The (In)Famous Report," MTNI 1996, https://aclanthology.org/www.mt-archive.info/90/MTNI-1996-Hutchins.pdf.

The committee's November 1966 report, *Languages and Machines: Computers in Translation and Linguistics*, ran 124 pages and concluded — on the basis of evaluation experiments comparing MT output to human translation — that machine translation was "slower, less accurate, and twice as expensive as human translation" and that "there is no immediate or predictable prospect of useful machine translation" [T2] Wikipedia, "ALPAC"; [T2] Hutchins MTNI 1996. By the committee's estimate, the US government had spent roughly $20 million on MT research over the preceding decade with no operational system to show for it [T2] Pangeanic, "The ALPAC report," https://blog.pangeanic.com/alpac-report; [T2] Hutchins MTNI 1996, ¶ "Conclusions and recommendations."

The recommendations were striking for what they did and did not endorse. ALPAC recommended that funding be redirected from MT toward:

- basic research in computational linguistics;
- improving human translation tools (glossaries, terminology databases, dictation systems);
- evaluating the *quality and speed of human translators*, not machines [T2] Wikipedia, "ALPAC," "Recommendations" section; [T2] Hutchins MTNI 1996.

The effect was immediate and harsh. NRC ended all support; DARPA wound down its MT contracts; major US programmes at IBM, RAND, Berkeley, and Harvard closed within a few years [T2] Hutchins MTNI 1996. Some work survived — Wayne State (Josselson) until 1972, Texas (Lehmann), and the Mormon Church-funded systems — but the era of *ambitious, well-funded, government-backed* MT in the United States ended in late 1966 and did not really resume for nearly two decades [T2] Wikipedia, "ALPAC."

ALPAC was not solely about MT. By cooling enthusiasm for the most prominent applied AI programme of the era, it foreshadowed the broader 1973–80 "first AI winter," intensified in the UK by the 1973 Lighthill report's parallel critique of AI's "combinatorial explosion" problem on real-world tasks [T2] Wikipedia, "Lighthill report," https://en.wikipedia.org/wiki/Lighthill_report; [T2] Wikipedia, "AI winter," https://en.wikipedia.org/wiki/AI_winter.

## 8. ELIZA 1966: A Coda, and a Warning

In the same year as ALPAC, Joseph Weizenbaum at MIT published "ELIZA — A Computer Program for the Study of Natural Language Communication Between Man and Machine" in *Communications of the ACM* (vol. 9, no. 1, January 1966, pp. 36–45), running on CTSS in MAD-Slip on an IBM 7094 [T1] Weizenbaum 1966, CACM 9(1), pp. 36–45; [T2] Wikipedia, "ELIZA," https://en.wikipedia.org/wiki/ELIZA. Its most famous script, DOCTOR, mimicked a Rogerian psychotherapist by reflecting users' inputs back as questions using simple keyword-spotting and template substitution.

ELIZA had no understanding of language in any reasonable sense. It also fooled people. Weizenbaum reported that his own secretary asked him to leave the room so she could talk to the program privately [T2] Wikipedia, "ELIZA"; [T3] Smithsonian Magazine, "Why Joseph Weizenbaum Invented the ELIZA Chatbot," https://www.smithsonianmag.com/history/why-the-computer-scientist-behind-the-worlds-first-chatbot-dedicated-his-life-to-publicizing-the-threat-posed-by-ai-180987971/. The "ELIZA effect" — humans projecting understanding onto systems that don't have it — was christened then and is still the relevant frame for evaluating modern chat interfaces. Weizenbaum himself spent the next decade arguing that the ease with which people anthropomorphised ELIZA was a moral problem, culminating in *Computer Power and Human Reason* (1976) [T2] Wikipedia, "ELIZA," "Later work" section.

The pairing is worth holding in mind: 1966 produced both the report that ended the optimism about *machines understanding language* and the demo that proved how little understanding was needed for *humans to feel understood by machines*. Every NLP era since has navigated some version of that gap.

## 9. Why the Era's Promises Outran Its Methods

Looking back, four structural reasons explain the 1950s–60s collapse:

1. **No data, no compute, no learning.** All MT systems of the era were hand-engineered: lexicons, transfer rules, morphological tables. There was no corpus infrastructure (the Brown Corpus, the first million-word balanced English corpus, only appeared in 1964) and no computational budget for the kind of statistical approaches Shannon and Weaver had sketched [T2] Jurafsky & Martin, ch. 1.
2. **The semantic problem was invisible from the demo level.** Georgetown–IBM's chemistry sentences hid the world-knowledge requirement. Bar-Hillel's pen example made it explicit, but the field had no representation of world knowledge to deploy.
3. **Chomsky's prestige redirected linguists away from statistics.** *Three Models* gave intellectual cover for a generation of computational linguists to dismiss n-gram and Markov approaches as in-principle inadequate, slowing the recombination of statistics and linguistics until the late 1980s [T2] Jurafsky & Martin, ch. 1.
4. **Funders bought the timeline, not the science.** The Georgetown demo's "three to five years" became the implicit deliverable schedule. When ALPAC measured against that schedule, the field looked like a failure — even though, on a longer timescale, the foundational ideas (Shannon's n-grams, Weaver's cryptographic frame, Turing's learning-machine sketch) were exactly the ones that worked when compute caught up.

The first generation of NLP researchers was, in other words, not wrong about the destination. They were wrong about how long the journey would take, and the institutional cost of being wrong about timelines is what produced the field's first winter.

## Sources

- [T1] Turing, A. M. (1950). "Computing Machinery and Intelligence." *Mind* 59(236), pp. 433–460. https://www.cs.ox.ac.uk/activities/ieg/e-library/sources/t_article.pdf — peer-reviewed paper.
- [T1] Shannon, C. E. (1948). "A Mathematical Theory of Communication." *Bell System Technical Journal* 27, pp. 379–423 and 623–656. https://people.math.harvard.edu/~ctm/home/text/others/shannon/entropy/entropy.pdf — peer-reviewed paper.
- [T1] Shannon, C. E. (1951). "Prediction and Entropy of Printed English." *Bell System Technical Journal* 30(1), pp. 50–64. https://www.princeton.edu/~wbialek/rome/refs/shannon_51.pdf — peer-reviewed paper.
- [T1] Chomsky, N. (1956). "Three Models for the Description of Language." *IRE Transactions on Information Theory* IT-2(3), pp. 113–124. https://chomsky.info/wp-content/uploads/195609-.pdf — peer-reviewed paper.
- [T1] Bar-Hillel, Y. (1960). "The Present Status of Automatic Translation of Languages." *Advances in Computers* 1, pp. 91–163. https://aclanthology.org/www.mt-archive.info/Bar-Hillel-1960.pdf — peer-reviewed survey paper.
- [T1] Weizenbaum, J. (1966). "ELIZA — A Computer Program for the Study of Natural Language Communication Between Man and Machine." *Communications of the ACM* 9(1), pp. 36–45 — peer-reviewed paper.
- [T2] Chomsky, N. (1957). *Syntactic Structures*. Mouton, The Hague. https://www.ling.upenn.edu/courses/ling5700/Chomsky1957.pdf — landmark monograph.
- [T2] Weaver, W. (1949). "Translation" memorandum, July 1949 (Rockefeller Foundation). Reproduced in W. J. Hutchins, "Warren Weaver memorandum, July 1949," MTNI 1999. https://aclanthology.org/www.mt-archive.info/90/MTNI-1999-Hutchins.pdf — landmark white paper.
- [T2] ALPAC (1966). *Languages and Machines: Computers in Translation and Linguistics*. National Research Council, Washington DC. https://www.mt-archive.net/50/ALPAC-1966.pdf — official government report.
- [T2] Hutchins, W. J. (1996). "ALPAC: The (In)Famous Report." *MT News International* 14, pp. 9–12. https://aclanthology.org/www.mt-archive.info/90/MTNI-1996-Hutchins.pdf — domain expert reference.
- [T2] Hutchins, W. J. (2004). "The Georgetown-IBM experiment demonstrated in January 1954." Proceedings of AMTA 2004. https://aclanthology.org/2004.amta-papers.12/ — domain expert reference.
- [T2] Hutchins, W. J. (1999). "Warren Weaver memorandum, July 1949." MT News International. https://aclanthology.org/www.mt-archive.info/90/MTNI-1999-Hutchins.pdf — domain expert reference.
- [T2] Jurafsky, D. & Martin, J. H. *Speech and Language Processing* (3rd edition draft, ch. 1 "Historical Notes"). https://web.stanford.edu/~jurafsky/slp3/ — canonical NLP textbook.
- [T2] Wikipedia, "Computing Machinery and Intelligence." Accessed May 2026. https://en.wikipedia.org/wiki/Computing_Machinery_and_Intelligence — reference encyclopaedia.
- [T2] Wikipedia, "Georgetown–IBM experiment." Accessed May 2026. https://en.wikipedia.org/wiki/Georgetown%E2%80%93IBM_experiment — reference encyclopaedia.
- [T2] Wikipedia, "Warren Weaver." Accessed May 2026. https://en.wikipedia.org/wiki/Warren_Weaver — reference encyclopaedia.
- [T2] Wikipedia, "Syntactic Structures." Accessed May 2026. https://en.wikipedia.org/wiki/Syntactic_Structures — reference encyclopaedia.
- [T2] Wikipedia, "Yehoshua Bar-Hillel." Accessed May 2026. https://en.wikipedia.org/wiki/Yehoshua_Bar-Hillel — reference encyclopaedia.
- [T2] Wikipedia, "ALPAC." Accessed May 2026. https://en.wikipedia.org/wiki/ALPAC — reference encyclopaedia.
- [T2] Wikipedia, "ELIZA." Accessed May 2026. https://en.wikipedia.org/wiki/ELIZA — reference encyclopaedia.
- [T2] Wikipedia, "Lighthill report." Accessed May 2026. https://en.wikipedia.org/wiki/Lighthill_report — reference encyclopaedia.
- [T2] Wikipedia, "AI winter." Accessed May 2026. https://en.wikipedia.org/wiki/AI_winter — reference encyclopaedia.
- [T2] Wikipedia, "Entropy (information theory)." Accessed May 2026. https://en.wikipedia.org/wiki/Entropy_(information_theory) — reference encyclopaedia.
- [T2] *MultiLingual* magazine, "Seventy Years of Machine Translation." May 2024. https://multilingual.com/magazine/may-2024/seventy-years-of-machine-translation/ — domain industry publication.
- [T2] Pangeanic, "The ALPAC report." https://blog.pangeanic.com/alpac-report — domain industry publication.
- [T3] Smithsonian Magazine, "Why Joseph Weizenbaum Invented the ELIZA Chatbot." https://www.smithsonianmag.com/history/why-the-computer-scientist-behind-the-worlds-first-chatbot-dedicated-his-life-to-publicizing-the-threat-posed-by-ai-180987971/ — long-form magazine.
- [T3] *The Future of MT is Now and Bar-Hillel was (almost entirely) Right.* BISFAI-95 conference paper. https://cdn.aaai.org/BISFAI/1995/BISFAI95-013.pdf — conference talk / position paper.
