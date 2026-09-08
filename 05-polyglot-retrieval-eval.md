# Retrieval That Works in English and Fails in Hungarian and Japanese

**Project:** polyglot-retrieval-eval — a reproducible retrieval benchmark (private repo; source, corpus and results available on request)
**Status:** Complete; corpus, 141 labels, raw JSON results and generated tables all committed and reproducible from a clone
**Role:** Sole author
**Stack:** Python 3.11. `sentence-transformers` (`intfloat/e5-base-v2`, `intfloat/multilingual-e5-base`), reranker `BAAI/bge-reranker-v2-m3`, PyTorch (CPU), SudachiPy, Snowball stemmers. BM25, Reciprocal Rank Fusion and every metric implemented from scratch on the standard library.

A controlled experiment that runs one retrieval pipeline over the *same* text in English, Hungarian and Japanese, measures how far accuracy drops off English, and isolates which component causes each drop — so the fix is a targeted swap rather than a guess.

*Every number below comes from the committed results. Where a judgement is mine rather than a measurement, it says so.*

## 1. The gap

Retrieval stacks are built and tuned in English, then pointed at multilingual content on the quiet assumption that behaviour transfers. "Retrieval is worse in other languages" is folklore that gets repeated and rarely measured on parallel text, and almost never decomposed into *which* part fails. That decomposition is the whole value: "Japanese retrieval is worse" is not actionable, whereas "naive BM25 scores zero on Japanese because it cannot find word boundaries, and a Japanese tokenizer recovers it" tells you exactly what to change.

This is a personal project rather than client work, and it is built to be reproduced end to end: given the repository, a cold clone regenerates every number in an afternoon. The source is kept private for now — I'm happy to arrange a walkthrough or read-only access on request (nagytmas@gmail.com).

## 2. The instrument

The comparison had to be content-controlled, so English and Hungarian use one parallel document: the EU AI Act (Regulation (EU) 2024/1689), fetched from the EU Publications Office Cellar in both official language versions. Japanese uses Japan's privacy statute (APPI) from e-Gov — topically related, but not parallel, and I flag every Japanese comparison as topical rather than content-controlled because of it. Chunking is character-based and identical across all three languages, so the corpus never favours English.

141 labels, hand-authored across six failure categories (definition, exact-string, synonym, split-answer, cross-lingual, unanswerable): 55 English, 48 Hungarian, 38 Japanese. Six retrievers, each run against every language: naive-whitespace BM25, tokenizer-aware BM25, an English-centric dense model, a multilingual dense model, Reciprocal Rank Fusion, and a cross-encoder reranker over the fused candidates. Metrics are recall@{1,3,5,10} and MRR@10, per language and per category.

## 3. What I found

Headline metric is recall@5 — how often the correct passage lands in the top five.

| retriever | EN | HU | JA |
|---|---|---|---|
| bm25_naive (whitespace) | 0.33 | 0.27 | **none — 38/38 zero-signal** |
| bm25_tok (language tokenizer) | 0.51 | 0.53 | 0.41 |
| dense_en (e5-base-v2, English) | 0.71 | **0.35** | **0.21** |
| dense_multi (multilingual-e5-base) | 0.79 | **0.75** | **0.67** |
| rrf (BM25_tok + dense_multi) | 0.67 | 0.65 | 0.67 |
| rerank (bge-reranker-v2-m3) | 0.89 | 0.81 | 0.80 |

Three findings stand out.

**Naive keyword search is not weak on Japanese, it is dead.** All 38 Japanese queries score zero signal — Japanese writes without spaces between words, so a whitespace tokenizer cannot even split the text into terms to match. Swapping in a Japanese segmenter (SudachiPy) lifts recall@5 from unmeasurable to 0.41.

**The English dense model falls off a cliff outside English.** recall@5 goes 0.71 → 0.35 → 0.21 across English, Hungarian and Japanese, while the *same-architecture* multilingual model holds at 0.79, 0.75 and 0.67. The two dense models differ only in language coverage, so coverage is the cause, not the architecture.

**Lexical retrieval is blind across languages.** On cross-lingual queries both BM25 variants score 0.00–0.17, while multilingual dense reaches 0.50–0.88 and the reranker 0.88–1.00. Keyword overlap simply does not exist when the query and the answer are in different languages.

## 4. What fixed it

Every fix is a component swap, which is the point of building the matrix rather than a single number.

- **Multilingual embedding model instead of the English one.** Hungarian more than doubles (0.35 → 0.75); Japanese roughly triples (0.21 → 0.67). Largest single lever.
- **A language-aware tokenizer for lexical retrieval.** Japanese BM25 goes from dead to 0.41; Hungarian benefits from Snowball stemming too (0.27 → 0.53).
- **A cross-encoder reranker on top.** All three languages land near English parity (0.89 / 0.81 / 0.80), and it is the only retriever whose score separates answerable from unanswerable queries cleanly — the mean top-1 gap is 0.38 / 0.38 / 0.70, versus near-zero for every dense and BM25 model. A confidence threshold to decline unanswerable queries is viable *after* reranking, not before.

## 5. What did not work

Fusing weak BM25 with strong dense retrieval (Reciprocal Rank Fusion) is treated as a free win in a lot of RAG advice. It was not one here. RRF never beat the multilingual dense model alone: it dragged it down on English (0.67 vs 0.79) and Hungarian (0.65 vs 0.75) and only tied it on Japanese. Fusing a strong retriever with a weak one pulls the strong one toward the weak one. Reported plainly because it contradicts the common recommendation.

## 6. How I kept it honest

Queries where every chunk scores about zero are flagged and never counted as a pass, so a retriever cannot score by accident on text it did not actually match; ties are broken by chunk id so nothing passes by luck. The Japanese synonym queries were a trap I caught myself in: the first versions shared kanji with the gold answer and flattered BM25 to a perfect 1.00 on that cell, so I rewrote them with everyday and katakana paraphrases, which dropped tokenized BM25 to 0.20 there while multilingual dense held at 0.80 — the disjoint-in-intent version is the one that ships.

The honest limits, stated before a reader finds them: the labels are drafted by a model reading the corpus and **not verified by a native legal expert** — this is the biggest limitation, and definition and exact-string golds were grep-checked but the synonym and split judgements are mine. APPI is topically related to the AI Act, not parallel, so only the English-versus-Hungarian comparison is truly content-controlled. The corpus is small (about 200–250 chunks per language) and per-category cells run as few as 5–12 queries, so they read as direction, not precision.

## 7. What this is and is not

It is a diagnostic benchmark, not a product: retrieval only, no generation, no UI, no agent. Its value is the decomposition — pinning each failure to a component and a fix — and that every number is reproducible from the committed corpus and results. The one takeaway I would hand someone building RAG for non-English content: the English defaults degrade quietly and your offline metrics will not warn you unless you evaluate in the target language, with a multilingual model and language-aware tokenization, before you ship.

---

**Request a demo or read-only access to the source:** [nagytmas@gmail.com](mailto:nagytmas@gmail.com)
