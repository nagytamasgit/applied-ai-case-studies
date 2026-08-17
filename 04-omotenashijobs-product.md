# The Product Around the Model: OmotenashiJobs.jp End to End

**Companion piece to [case study 01](01-video-assessment.md), which covers the assessment pipeline. This one covers everything around it — the product that real candidates and employers actually touch.**

**Scope:** The entire platform, built solo from zero: candidate side, employer dashboard, admin, visual design, frontend, backend, deployment
**Stack:** Next.js, Node, PostgreSQL, Tailwind CSS; GitHub-based deployment to Coolify
**Status:** In daily production use for 10+ months; 1,600+ candidate videos through the full flow

---

## 1. Why the product layer is its own problem

An AI pipeline that assesses candidate videos is worth nothing if the video never arrives, arrives broken, or produces output an employer cannot responsibly act on. The pipeline in case study 01 sits in the middle of a chain of unglamorous product problems: a candidate on a phone, on a variable mobile connection, uploading a 3–5 minute video; minutes of processing the user cannot see into; and a results screen that is legally forbidden from telling the employer what to do.

Each of those is a design decision, and each one shaped the system more than any model choice did.

## 2. The three product decisions that mattered

**The upload path assumes failure.** Candidates record on phones, in whatever format their device produces, on connections that drop. Rather than validating strictly and rejecting, the platform asks for fixed formats but accepts what arrives and converts and optimizes media automatically. The candidate's job is to record themselves once; format compatibility is the system's job. On mobile-first flows, every re-upload you force is a candidate you lose.

**Nobody ever sees a partial state.** Processing takes real time, and the two audiences get two different answers to "what happens while we wait." The candidate sees an animated indicator — enough to know the system is working, nothing that invites refreshing or re-submitting. The employer sees nothing at all until the video is fully uploaded and processed: no half-finished entries, no "pending" rows that might be acted on prematurely. An employer who can see an incomplete assessment will read it as a complete one. The cleanest fix was to make intermediate states invisible.

**The results table is the legal constraint, rendered.** Case study 01 describes the rule: the system may surface evidence but must not produce a hiring verdict. On the employer dashboard, that abstraction had to become an actual screen. The design: a table, one row per candidate, columns per assessed dimension — language proficiency, keigo, bow, posture — each showing whether the result was positive, explicitly framed in the UI as suggestions. The framing is not decoration. The column layout invites comparison and human judgment; a single score would invite sorting and cutting. Employers received genuinely impactful structured data, and the interface itself is what kept the human as the decision-maker.

**And an escalation rule with no exceptions:** anything with any error in sight — a conversion anomaly, an uncertain analysis, anything short of clean — was routed to a human for full review. Over 85% of videos went through fully automatically; the rest were never silently guessed at. The automation rate you ship is the one you can stand behind on every single row of that employer table.

## 3. What was designed and then killed

**Fully automated personal-data handling — cut.** The original design automated more of the candidate data lifecycle. Handling job applicants' personal data, in a hiring context, under Japanese rules, is exactly the domain where automation risk concentrates, and the automated handling was removed in favor of more conservative flows. Slower, deliberately.

**The visa-partner marketplace — killed entirely.** The original product vision connected visa support partners directly with employers through the platform: an obvious revenue line for a service placing foreign candidates into Japanese jobs. On examination it was not clearly legal, and it was dropped. This was the single largest piece of the original concept to be removed, and the decision was not close. A platform whose entire value rests on operating credibly inside Japanese hiring rules does not fund itself by operating at the edge of them.

Both cuts share a shape with the constraint in case study 01: the regulatory boundary, taken seriously early, produced a smaller and better product than the unconstrained design would have.

## 4. How I knew the product worked

The pipeline's accuracy story lives in case study 01. The product's evidence is blunter: 1,600+ videos from real candidates on real devices have gone through the full record–upload–convert–process–present flow, in daily use for over ten months, with the client still operating it. A solo-built platform surviving ten months of live hiring traffic — candidates, employers, and an admin surface — without the developer on call behind it is the product-level test, and it is ongoing.

## 5. What I would do differently now

**Resumable uploads.** The convert-anything approach absorbs format failures, but connection failures on a multi-minute mobile upload still cost retries. Chunked, resumable upload is the single biggest robustness win available and I would build it first.

**Instrument the funnel.** I know videos that arrive get processed. I have weaker visibility into candidates who started recording and never finished uploading — the drop-off before the system ever sees them. That number is the product's real conversion metric and it deserves first-class measurement.

**Keep the table.** Given a redesign, I would still present results as comparative evidence rather than scores — now by conviction rather than legal necessity. It made employers better at their own decision, which is the only defensible goal for AI in hiring.
