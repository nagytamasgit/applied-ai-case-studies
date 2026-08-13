# A Self-Improving Voice Agent With a Human Gate

**Client:** Horváth Estates, Marbella (real estate) — built under AI Business Automation
**Period:** 6 months, from summer 2025
**Role:** Sole architect and engineer
**Stack:** ElevenLabs (voice), GoHighLevel (CRM), n8n (orchestration), Supabase (real-time state), Next.js + Python services

An autonomous voice agent that placed and received calls with real prospective property buyers, and a weekly evaluation loop that proposed its own improvements — prompts, rules, knowledge base — for human approval before anything shipped.

---

## 1. The gap

An estate agency in Marbella loses enquiries in the hours nobody is working. Buyers browse listings at 22:00 in another timezone, call once, and if nothing answers they call the next agency. The ideal is an agent that answers every call instantly, knows the current state of every property, and asks the qualifying questions a good human agent would — at 03:00, in parallel, on a Sunday.

What models could actually do at the time was narrower. Speech recognition mishears numbers. Conversational agents drift from their instructions over weeks of prompt edits. And an agent that sounds fluent while quoting a property that sold last Tuesday is worse than no agent at all — it damages trust in a market where a single transaction is six figures.

So the real problem was never "can it hold a conversation." It was **can it hold a conversation that stays correct as the business changes underneath it**, without a human rewriting the prompt every week.

## 2. What I deliberately did not build

The agent was forbidden from three things, and the boundaries were the design:

- **It could not negotiate.** The agent sent real property information, including listing prices — that is public, verifiable, and exactly what a caller rings to ask. What it could not do was issue an offer at any price other than the listing price, or move on price in conversation. The line is between *stating a published fact* and *creating a commitment the agency is bound by*. The first is what the agent is for; the second is where one wrong figure costs more than the automation saves.
- **It could not evaluate itself unsupervised.** An LLM scoring its own transcripts optimises for transcripts that score well. Reviews came from humans.
- **It could not ship its own changes.** The system was capable of running fully autonomously — that switch existed and I left it off.

Two things ran without a human in the loop, and only two: the agent proposing improvements, and the knowledge base expanding from what customers actually asked. Proposals are cheap and reversible; knowledge additions are additive and verifiable. Everything with a blast radius kept a gate.

Also cut: multilingual support beyond English and Spanish. Scandinavian and German buyers were a real segment, but each language multiplies the evaluation surface — every regression test, every review pass, every drift check, again. It was postponed rather than half-done.

## 3. How I knew it was working

There is no ground truth for a good sales call. Two calls can be identical in transcript and opposite in outcome, and conversion is a lagging, noisy signal contaminated by everything else the business does. So the evaluation system was the substance of the project, not an afterthought to it.

**Per-call human review.** Reviewers scored calls on rating, transcript quality, conversation handling, and tagged errors by category. Error tagging mattered more than the rating: a 3/5 tells you little, "misheard a phone number twice" tells you what to fix.

**Weekly self-review with proposed diffs.** Every week the system reviewed its own performance against review scores, error tags, internal reports, and what did and didn't convert — then generated concrete proposed changes to prompts, rules, and knowledge base entries, and notified the responsible human to review them. The LLM proposed; it did not decide.

**A/B harness with a synthetic caller.** This was the piece that made the loop trustworthy. I built a system that ran multiple agent variants simultaneously against the same set of customer questions, driven by a second AI voice acting as the caller. That gave controlled, repeatable comparison: same questions, same conditions, different prompt — so a human could listen to both full conversations and judge the change directly rather than inferring from aggregate numbers. Without this, "the new prompt seems better" is an opinion. With it, it's a comparison.

**Multiple gates on the critical paths.** Not because the process was bureaucratic, but because of what I observed: **the agent drifted and regressed constantly.** Prompt changes that improved one behaviour degraded another; knowledge additions shifted tone; small edits compounded. This is the honest finding of the project. A self-improving system is not a system that gets monotonically better — it is a system that moves, and the human layer is what keeps the movement directional. The parts that mattered most got more than one reviewer.

**Grounding instead of trust.** Rather than evaluating whether the agent hallucinated property details, I removed the opportunity: it had live data access with a hard sub-3-second freshness requirement, backed by real-time Supabase sync. Across the project there were no hallucinated property facts — not because the model was reliable, but because it was never asked to remember anything it could look up.

**What the loop actually changed.** Before it existed, improvement meant a human deciding to sit down and edit a few prompt values, roughly twice a month, driven by whoever happened to notice a problem. After, the system surfaced a reasoned set of proposed changes every week, sourced from tagged review data rather than recollection. The iteration rate roughly doubled, but the more important change was in what the human was doing: no longer hunting for what was wrong, only judging whether a proposed fix was right. The evaluation and self-improvement tooling itself — scoring pipelines, review aggregation, proposal generation — was Python, developed continuously alongside the system rather than built once.

**The metric that mattered: did the call survive.** Review scores are useful for diagnosis but hard to trust in aggregate. The signal I trusted most was blunter — whether a caller stayed in the conversation. Early on, a large share of calls ended within seconds, or with the caller asking to be put through to a human. That is an unambiguous verdict, and unlike a rating it cannot be inflated by a generous reviewer. By the end of the six months, calls that ran to a proper conclusion as a real back-and-forth conversation were up 82%.

It is a proxy, and worth naming as one: it measures whether the agent was tolerable, not whether it was good. A caller who stays on the line may still be poorly served. But it is a proxy that is cheap to compute, impossible to game from inside the system, and directly tied to the failure the client actually cared about — the enquiry that evaporates.

**Known weakness, not fixed:** the dominant error was mishearing numbers and words, which forced the agent to re-ask and made the exchange briefly awkward. Most callers absorbed it, understanding they were calling outside business hours. Latency also varied. I judged both acceptable against the alternative — an unanswered phone — and spent the effort elsewhere. That was a deliberate trade, and it was the right one at the volume involved, but it is the first thing I would revisit.

## 4. Cost

- **~1,000 calls per month**, sustained over six months — roughly 6,000 live conversations with real buyers.
- **€0.05–€2.00 per call** depending on length — below the equivalent Spanish labour rate.
- **Customer acquisition cost fell roughly 40%.**
- **Conversations increased**, from two compounding causes: continuous availability, and concurrency — the agent handled three or more simultaneous calls, where a human handles one and the fourth caller hears a busy tone.

The agency's revenue grew substantially over this period. The automation system was one contributing factor among several, and I would not claim the figure as a result of this work alone; the numbers above are the ones I can attribute directly.

**Reproducibility.** The system has since been redeployed for other businesses through the client's GoHighLevel offering. The conversational logic, evaluation loop, and review tooling transfer intact — what changes per deployment is the business goal and the knowledge base. The honest limit is infrastructure: each deployment still needs a VPS provisioned and configured by hand, which is the part I would automate next.

## 5. What I would do differently now

**Native speech-to-speech.** The mishearing problem was largely an artifact of the transcription boundary. Current end-to-end speech models remove that seam and would address both the dominant error mode and the latency variance at once.

**Replay-based regression testing.** The A/B harness compared variants going forward. I would now maintain a frozen set of recorded real calls and replay every proposed change against it before it reaches a human reviewer — catching regressions automatically and reserving human attention for genuine judgment calls rather than obvious breakage.

**Measure the reviewers.** I trusted human scores without measuring agreement between reviewers or of a reviewer against themselves over time. That number is the ceiling on the reliability of every other number in the system, and I did not know it.

**Revisit multilingual.** The cost of adding a language has fallen sharply. The reason to postpone it was evaluation load, and that reason is weaker now than it was.
