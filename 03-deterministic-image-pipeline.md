# When the Right Answer Isn't a Model: A Deterministic Image Pipeline

**Client:** Festede.hu — custom paint-by-numbers paintings generated from customer photographs
**Status:** Recently released to production; several thousand customer images processed
**Role:** Sole architect and engineer
**Stack:** Python. No LLM, no ML model, anywhere in the pipeline.

A system that converts an arbitrary customer photograph into a paintable image using a fixed palette of 806 real paints, selects the 30, 39, or 48 paints the customer will actually receive, and updates the order so logistics knows what to put in the box.

---

## 1. The gap

Festede.hu sells custom paint-by-numbers kits made from photographs customers upload — a portrait, a wedding photo, a dog. Producing one required six manual touchpoints across the whole team: graphic designers reworking the image, review, palette decisions, logistics. Roughly an hour of collective team time per order, which caps the business at whatever the designers can absorb.

The obvious framing is "automate the graphic designer." That framing is wrong, and getting it right was the important early decision.

**The output of this system is not an image. It is a bill of materials.** Every region of the final picture must map to a specific jar of paint that physically ships to the customer. There is no blending, no dithering, no approximation, no gradient. A colour either exists in the 806-paint palette and is one of the 30, 39, or 48 selected, or it cannot appear. The problem is not image generation under aesthetic judgment — it is constrained discrete optimisation where the constraints are physical objects in a cardboard box.

Once framed that way, the tooling follows. A generative model produces plausible pixels; this problem needs provably valid ones.

## 2. What I did not build

**No ML, no LLM.** Not because I couldn't, but because nothing about the problem asked for it. The transformation is deterministic and specifiable: given this photograph and this palette, there is a correct set of colour assignments, and the difficulty is in the rules, not in the perception. A model here would introduce inference cost per image, non-determinism in a process that ships physical goods, and a maintenance surface that needs monitoring and eventually drifts.

That last point was also a hard client requirement: the business wanted no ongoing model cost and no maintenance burden. It is worth being plain that this constraint was given to me rather than chosen by me — but it pointed the same direction I would have gone anyway, and it clarified the target. A deterministic Python pipeline has no inference bill, produces identical output for identical input, and will still run unchanged in three years.

**No automated choice of detail level.** The system produces three versions of every image — 30, 39, and 48 colours — priced separately, and the customer picks. This looks like a product decision because it is one. Detail level is a taste-and-budget tradeoff belonging to the person buying the painting, and there was no case for the system guessing on their behalf.

**No general-purpose solution.** Rather than one algorithm handling all images, the pipeline carries explicit rules for the categories that actually occur: faces, eyes, lips, skin tones, hair highlights, beards, animal fur, backgrounds. Faces need different treatment from fur, because a face read wrong is uncanny and fur read wrong is merely flat. Chasing generality would have produced something uniformly mediocre.

## 3. How I knew it was working

**Expert review at scale.** Over 2,000 images were reviewed by the graphic designers and the founder — the people whose manual work the system was replacing, and the people best positioned to see where it fell short. Every disagreement fed back into the rules. Where a category of image kept failing, it got its own rule set. The review continued until they approved the output, and the rules for the most frequent image categories were built directly from that loop.

This is expert-knowledge extraction rather than metric optimisation. There is no loss function for "this portrait looks like her." What exists is a designer saying the skin has gone muddy, and my turning that into a threshold.

**Comparison against the work it replaced.** The result eventually exceeded the designers' own manual output — with an honest caveat I would state in any interview: the designers were working under production time pressure, one order at a time, and the system was not. It doesn't beat an unhurried expert. It beats what an expert can produce in the time the business can afford, on every order, consistently. That is the comparison that matters commercially, and inflating it into "better than human designers" would be a misrepresentation.

**Tests that determinism makes possible.** A deterministic pipeline is trivially testable — fixed input, expected output, no tolerance bands — and it was tested that way: automated tests over the critical paths, run on every deployment. Simple tests, but the kind a model-based approach could never have this cheaply, which is itself part of the argument for the architecture.

**Rejection rate as the outside check.** The previous process already had a Python colour-matching component, so there was a real baseline. Customer rejections fell substantially against it — an outcome measured on people spending money, not on internal review.

**The honest limits.** Monochrome and near-monochrome images are the hardest case, and were brought to acceptable rather than good; when the source has little chromatic range, a discrete palette has little to work with. And the system has only recently entered production — several thousand images with no issues so far is encouraging, not yet proven. The ten-month track record I'd want doesn't exist yet.

## 4. Cost

- **Team time per order: ~1 hour → ~5 minutes.** Six manual touchpoints across designers, review and logistics reduced to one — logistics assembling the order.
- **Marginal cost per image: effectively zero.** No inference, no API calls. The pipeline is CPU time.
- **15–30 seconds per image**, fast enough that customers now upload a photo and see the actual final product live before ordering — which the manual process could never offer at any price.

That last point is the one I'd emphasise. The automation removed cost, but the latency is what changed the product: a customer seeing their own dog rendered as a real paintable image, before paying, is a different purchase from one made on trust.

## 5. What I would do differently now

**Instrument the rules.** The category rules were tuned by me, by hand, against expert feedback. They work, but I cannot currently answer "which rule contributed most to the improvement" or "which rule is now doing nothing." A per-rule evaluation harness over a frozen image set would make the system improvable by someone other than me — and the fact that it currently isn't is the real weakness of this project.

**Attack monochrome deliberately.** It was brought to acceptable and then left. It's the one known-weak category and it deserves its own treatment rather than being handled by the general path.

**Keep the no-model decision.** With a year's hindsight and cheaper models available, I would make the same call. The reasons weren't about cost — they were determinism, zero maintenance, and the fact that the output has to be physically valid. Those haven't changed.
