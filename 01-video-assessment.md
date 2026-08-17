# Encoding an Expert's Eye: Video Assessment for Hospitality Hiring

**Client:** OmotenashiJobs.jp — recruitment platform placing candidates into Japanese hospitality roles
**Status:** In daily production use for 10+ months; 1,600+ candidate videos processed
**Role:** Sole architect and engineer
**Stack:** Go and Python (orchestration, analysis, pipeline), Gemini (video and multimodal), Whisper and ElevenLabs (transcription), Claude (Japanese and English language assessment)

A pipeline that watches a 3–5 minute self-introduction video from a candidate and produces structured evidence about their Japanese proficiency, English proficiency, keigo usage, bow, and posture — everything a senior hotel manager notices in the first minute, made explicit and consistent.

---

## 1. The gap

A senior Japanese hospitality professional can watch a candidate for sixty seconds and know. Not just whether the Japanese is good, but whether the keigo is natural or memorised, whether the bow is the right depth and held the right length, whether the posture reads as trained or improvised. This judgment is real, it is reliable, and it is almost entirely unarticulated — it lives in people who have done the work for twenty years and who cannot be cloned for every applicant on a recruitment platform.

What models could do was narrower than that. A vision-language model can describe a bow. It cannot, unprompted, tell you that this particular bow is fifteen degrees short for the context, because it has no idea what the context demands. The gap was never perception — it was **the standard**. Recognising the gesture is the easy half; knowing what correct looks like is the whole problem.

So the work divided cleanly: extract reliable observations from video and audio, and separately, encode the standard against which observations are judged.

## 2. The constraint that shaped the product, and what I killed

**The system was never allowed to rate a candidate.** Under the rules governing hiring in Japan, an automated system may not produce a score, ranking, or judgment that decides a person's application. We could analyse the video and surface structured observations. We could not output a verdict.

This is the constraint I would put first in any description of the project, because it turned out to be a better product than the one we would have built without it. A score invites a recruiter to stop thinking. Structured evidence — *here is the keigo used, here is the bow, here is the transcript with proficiency indicators* — makes the human faster at their own judgment instead of replacing it. Every candidate decision was made by a person, every time. The system was capable of identifying unqualified candidates reliably; it was not permitted to act on that, and shouldn't have been.

Beyond the legal boundary, most of what we designed did not ship. The discards fall into three kinds:

**Killed for unreliability.** Pure Python pose analysis, and cheap transcription models. Both were attractive on cost and neither produced output I trusted enough to put in front of a recruiter. A wrong observation is worse than no observation, because it consumes the human's attention and then has to be un-believed.

**Killed for cost, despite working.** These were the hard ones. A frame-by-frame mouth tracker that located the speaker's mouth and used it to disambiguate transcription against the audio — it worked beautifully, and measurably improved Japanese transcription accuracy. A frame-by-frame overall facial impression analyser, same story. Both were cut. Something working is not an argument for shipping it; the argument has to be that it earns its cost per video, and at 1,600 videos these did not.

**Killed for legal or policy reasons.** Several ideas that would have produced genuinely useful signal were straightforwardly not permissible, and were dropped without much debate.

What survived was 8–12 metrics: a pose analyser, voice and tone analysis, and Japanese proficiency assessment. That combination accounted for roughly 85% of cases where the system's evidence aligned with a qualified human's own read of the candidate. The two expensive frame-by-frame analysers would have improved that figure. They were not worth what they cost, and the honest version of "we focused on the highest-impact points" is that we found the smallest set of signals that carried most of the judgment, and deliberately left the rest on the table.

## 3. How I knew it was working

**Real ground truth, which is rare here.** Many candidates held actual JLPT certification — N1 through N5, externally awarded and verifiable. This gave the language-proficiency component something almost no subjective-assessment system has: a validated external label to check against. The system's proficiency estimate could be compared to a certificate rather than to an opinion. Any assessment system without this is measuring its own agreement with itself; this one wasn't.

**Expert calibration for everything JLPT couldn't cover.** Bow, stance, keigo appropriateness, and omotenashi bearing have no certificate. I wrote the initial rules — what counts as a bow, what the thresholds are — and then sat with a senior hotel worker and adjusted them against real examples until the system's observations matched what an experienced professional actually saw. That loop, my formalisation corrected by their tacit knowledge, is where the standard came from. It is also the part that could not have been done by prompting alone.

**Deployment discipline.** Every deployment went through an automated test pipeline — the critical paths of the pipeline covered by automated tests, with schema validation on the structured outputs each stage passes to the next. A multi-stage system whose stages exchange structured data fails loudest at the seams, so the seams are where the checks live.

**Production as the long test.** The system has run daily for over ten months without issues, on live candidate applications, at a client who continues to use it. Sustained production use on real applicants is a harder test than any offline benchmark, because every failure mode surfaces eventually.

**What I did not measure, and should have.** I never quantified agreement between human reviewers on the subjective metrics — how often two senior hospitality professionals would score the same bow the same way. That number is the ceiling on what any system can achieve against those metrics, and without it the 85% figure lacks a denominator I can fully defend.

## 4. Cost

Cost was the primary design constraint and the reason for most of the discards, but I no longer have access to the exact per-video figures. What I can state accurately: it was expensive at the time, expensive enough that two working components were removed on cost grounds alone, and the same pipeline could be built roughly 90% cheaper with current model pricing.

Latency was not a constraint. This was never real-time — the pipeline ran as a continuous automated queue, picking up the next video as it finished the last. That is a genuinely different engineering problem from live analysis, and it meant throughput and cost per video mattered while response time did not.

## 5. What I would do differently now

**Revisit the pose analysis.** Pure Python pose estimation was cut for unreliability. With current tooling I would expect roughly 95% accuracy and would ship it as a first pass with human review on the remainder — replacing an expensive model call with a cheap deterministic one for most cases.

**Reinstate the mouth tracker.** It was cut for cost, not for quality. At current prices that calculation reverses, and it was the single best improvement we had to Japanese transcription accuracy.

**Measure inter-rater agreement before anything else.** Before building, I would spend a week having multiple hospitality experts assess the same set of candidates independently. It would tell me which metrics are reliably judgeable at all, and which are simply contested — and I suspect at least one of the shipped metrics would not survive that test.

**Keep the constraint.** Given a free hand to output scores, I would still not do it. The evidence-not-verdict design made the product better, and I would apply it deliberately rather than because I was required to.
