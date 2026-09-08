# Applied AI Case Studies

Tamás Nagy — full-stack and applied AI engineer, Budapest.

I build multimodal AI systems that run in production against real users, and I spend most of my effort on the part that decides whether they work: knowing what to measure when there is no correct answer, and knowing which ideas to kill. Four write-ups, each covering one system — what the gap was between the ideal and what models could actually do, what got discarded, how quality was measured without ground truth, and what it cost.

Written for an applied AI role. Clients are named with permission; figures are the ones I can defend, and where a number is a proxy or an estimate I say so.

---

### [01 — Encoding an Expert's Eye: Video Assessment for Hospitality Hiring](01-video-assessment.md)

Reproducing a senior hospitality professional's judgment of a candidate from a 3–5 minute video: Japanese and English proficiency, keigo, bow, posture. Go and Python, with Gemini, Claude and Whisper.

*Demonstrates:* extracting an unarticulated expert standard into a specification; validation against real external ground truth (JLPT certification); designing a system that is forbidden by law from producing a verdict, and is better for it.

### [02 — A Self-Improving Voice Agent With a Human Gate](02-voice-eval-system.md)

An autonomous voice agent handling ~1,000 live property enquiry calls a month, with a weekly loop that proposed its own prompt, rule and knowledge base changes for human approval. ElevenLabs, n8n, Supabase, Next.js and Python.

*Demonstrates:* building an evaluation harness rather than inheriting one; A/B testing conversational agents with a synthetic caller; treating model drift as a permanent condition to be managed rather than a bug to be fixed.

### [03 — When the Right Answer Isn't a Generative Model: A Deterministic Image Pipeline](03-deterministic-image-pipeline.md)

Converting customer photographs into paintable, numbered templates against a fixed palette of 806 real paints, selecting the paints that physically ship in the box. Classical Python image processing with two small discriminative models for perception — no generative model anywhere.

*Demonstrates:* recognising when a generative approach is the wrong tool; expert-knowledge extraction at scale (2,000+ reviewed images); reducing a six-touchpoint manual process to one; thirteen decisions in sequence, most of them reversals forced by a measurement.

### [04 — The Product Around the Model: OmotenashiJobs.jp End to End](04-omotenashijobs-product.md)

The full platform around the pipeline in study 01, built solo from zero: candidate upload flow, employer dashboard, admin, design and deployment. Next.js, Node, PostgreSQL.

*Demonstrates:* full-stack AI product engineering — the places where model output meets a user interface; rendering a legal constraint as a UI design; killing a revenue feature on legality grounds.

### [05 — Retrieval That Works in English and Fails in Hungarian and Japanese: A Multilingual RAG Benchmark](05-polyglot-retrieval-eval.md)

A controlled retrieval benchmark: the same legal text in English, Hungarian and Japanese, run through six retrievers, measuring how far accuracy drops off English and which component causes each drop. English-tuned dense retrieval falls from recall@5 0.71 to 0.35 (HU) and 0.21 (JA); a multilingual model, a language-aware tokenizer and a reranker bring all three back near parity. Python, sentence-transformers, PyTorch, SudachiPy; BM25 and RRF from scratch.

*Demonstrates:* designing a retrieval evaluation from corpus to metrics with no inherited harness; decomposing a failure into the component responsible rather than a single score; reporting the result that contradicts common advice (RRF fusion lost here), and the limitations, plainly. The one study whose code and every number a reader can clone and reproduce.

---

**Reading order.** They're numbered by relevance rather than chronology, and each stands alone. If you read one, read 01. If you read two, read 03 as well — it is the one where the answer was not to use a generative model, and it is deliberately the deep one: the other three are written to the length a hiring reader has; 03 goes to the depth a technical reviewer would want, including the decisions that were reversed and what reversed them. 01 and 04 are two halves of the same system: the model layer and the product built around it. 05 is the shortest path to running my code yourself: it is open source and reproduces end to end.

---

## Wider engineering portfolio

The four studies above go deep on single systems. [**PORTFOLIO.md**](PORTFOLIO.md) is the breadth behind them: an annotated index of 40 private repositories — 38 with shipped code — covering roughly the last year of work, from October 2025 onward.

Multi-tenant SaaS, marketplaces, mobile apps published to Google Play, an AI website builder, a WordPress security platform, and the automation and content pipelines around them. Each entry states what the project is, the problem it solves, its stack and a real feature list, and says plainly whether it is in production, shipped to a store, built but undeployed, or archived.

Those repositories are private because they contain client and commercial work. I'm happy to arrange a walkthrough or read-only access on request — client-owned code only with the client's consent.
