# Applied AI Case Studies

Tamás Nagy — full-stack and applied AI engineer, Budapest.

I build multimodal AI systems that run in production against real users, and I spend most of my effort on the part that decides whether they work: knowing what to measure when there is no correct answer, and knowing which ideas to kill. Three write-ups, each covering one system — what the gap was between the ideal and what models could actually do, what got discarded, how quality was measured without ground truth, and what it cost.

Written for an applied AI role. Clients are named with permission; figures are the ones I can defend, and where a number is a proxy or an estimate I say so.

---

### [01 — Encoding an Expert's Eye: Video Assessment for Hospitality Hiring](01-video-assessment.md)

Reproducing a senior hospitality professional's judgment of a candidate from a 3–5 minute video: Japanese and English proficiency, keigo, bow, posture. Go and Python, with Gemini, Claude and Whisper.

*Demonstrates:* extracting an unarticulated expert standard into a specification; validation against real external ground truth (JLPT certification); designing a system that is forbidden by law from producing a verdict, and is better for it.

### [02 — A Self-Improving Voice Agent With a Human Gate](02-voice-eval-system.md)

An autonomous voice agent handling ~1,000 live property enquiry calls a month, with a weekly loop that proposed its own prompt, rule and knowledge base changes for human approval. ElevenLabs, n8n, Supabase, Next.js and Python.

*Demonstrates:* building an evaluation harness rather than inheriting one; A/B testing conversational agents with a synthetic caller; treating model drift as a permanent condition to be managed rather than a bug to be fixed.

### [03 — When the Right Answer Isn't a Model: A Deterministic Image Pipeline](03-deterministic-image-pipeline.md)

Converting customer photographs into paintable images against a fixed palette of 806 real paints, selecting the paints that physically ship in the box. Pure Python, no model of any kind.

*Demonstrates:* recognising when a generative approach is the wrong tool; expert-knowledge extraction at scale (2,000+ reviewed images); reducing a six-touchpoint manual process to one.

### [04 — The Product Around the Model: OmotenashiJobs.jp End to End](04-omotenashijobs-product.md)

The full platform around the pipeline in study 01, built solo from zero: candidate upload flow, employer dashboard, admin, design and deployment. Next.js, Node, PostgreSQL.

*Demonstrates:* full-stack AI product engineering — the places where model output meets a user interface; rendering a legal constraint as a UI design; killing a revenue feature on legality grounds.

---

**Reading order.** They're numbered by relevance rather than chronology, and each stands alone. If you read one, read 01. If you read two, read 03 as well — it is the one where the answer was not to use a model. 01 and 04 are two halves of the same system: the model layer and the product built around it.
