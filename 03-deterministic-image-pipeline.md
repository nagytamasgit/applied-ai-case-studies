# When the Right Answer Isn't a Generative Model: A Deterministic Image Pipeline

**Client:** Festede.hu — custom paint-by-numbers kits generated from customer photographs
**Status:** In production; several thousand customer images processed
**Role:** Sole architect and engineer
**Stack:** Python. Two small discriminative models for perception. No generative model anywhere.

A system that converts an arbitrary customer photograph into a paintable, numbered template against a fixed palette of 806 real paints, selects the 30, 39 or 48 paints the customer will actually receive, and produces a print-ready PDF at exact canvas size.

---

## 1. The gap

Festede.hu sells custom paint-by-numbers kits made from photographs customers upload — a portrait, a wedding photo, a dog. Producing one order required six manual touchpoints across the whole team: a graphic designer running a Python pixelation step, then tracing by hand in CorelDRAW, then review, palette decisions, logistics. Roughly an hour of collective team time per order, which caps the business at whatever the designers can absorb.

The obvious framing is "automate the graphic designer." That framing is wrong, and getting it right was the important early decision.

**The output is not an image. It is a bill of materials.** Every region of the final picture must map to a specific jar of paint that physically ships to the customer. There is no blending, no dithering, no gradient. A colour either exists in the 806-paint recipe table and is one of the 30, 39 or 48 selected, or it cannot appear. The problem is not image generation under aesthetic judgment — it is constrained discrete optimisation where the constraints are physical objects in a cardboard box.

## 2. What I did not build

**No generative model, and no model training.** To be precise, because "no AI" would be inaccurate: the pipeline contains two small discriminative models, and both only *recognise* — neither draws anything. U²-Net separates the subject from the background; YuNet locates faces. Every decision that shapes the output is classical image processing. Nothing is generated, nothing is hallucinated, and the same photograph with the same settings produces byte-identical output every time. No GPU is required.

That was the architectural decision, and three constraints forced it. The output must be physically valid on every single run. When a customer says the skin looks wrong, I must be able to correct it rule by rule, which a learned colour mapping does not permit. And the business could not carry inference cost or a maintenance surface indefinitely.

**No general-purpose algorithm.** Faces need different treatment from fur: a face read wrong is uncanny, fur read wrong is merely flat. The pipeline carries explicit rules per content type, and a content profile decides which apply.

**No naive quantisation.** Choosing 30 colours from an image is trivial; choosing *which* 30, when the palette is a physical inventory, is not. Slots are allocated in priority order — eyes, then mouth, then skin, then shape, then background — and only then matched to the nearest real paint in CIELAB space. Each paint can be assigned once: matching independently let several colours land on the same pot, so a "30-colour kit" delivered about 22. The tiers also do not grow proportionally with box size, and none ever shrinks — skin runs 8 → 13 → 17 across the 30/39/48 boxes while background stays at 10. Measured on the face photos, a proportional 48-paint split put barely more tones on a cheek (9.77) than a 39-paint box (9.46); the chosen split gives 10.54. The extra budget goes where the eye looks.

**No difficulty levels.** The original design had easy/medium/hard. They were dropped: they moved only the smallest patch a painter had to fill, while the box is what the customer actually buys and sees. "Easy" also broke its own promise — it delivered 24 paints instead of thirty — and was erratic, scattering between 36% and 81% of medium's region count across five photos. The single ordering parameter became the paint count: 30, 39 or 48, all at medium's thresholds.

**No detail level chosen by the system.** Three versions are produced and the customer picks. Detail is a taste-and-budget trade-off belonging to the person buying the painting.

## 3. The problems that actually took the time

None of the hard problems were the ones I expected. All four below cost days and none of them were about model accuracy.

**Face detection where confidence is not monotonic in scale.** The first version used a Haar cascade, which found faces in foliage and grass while losing profiles, hats and sunglasses. Replacing it with YuNet fixed that and introduced a stranger failure: on a close-up two-person selfie at the 1300 px working width, the faces scored **0.58 confidence — lower than a cat scores (0.60)**. No threshold could separate people from animals, and the photo was processed as an animal through three review rounds. At 320 px the same faces score 0.95 while animals stay at 0.41–0.62. The fix was not a better model; it was a running strategy. The detector now runs at fixed widths (320 and 640) rather than at the working resolution, and the results are merged. Faces are also searched sideways and upside down, with per-orientation thresholds, because the numbers genuinely differ: an animal reaches 0.78 upright but only 0.68 sideways, while a person lying on a pillow reaches 0.82 sideways and 0.19 upright. A single threshold either admits the husky or leaves the sleeping couple faceless.

**Skin, in three stages, because the first two were not enough.** A fixed "skin colour range" fails across skin tones and lighting, so the first version used a raw ellipse around the face. That counted hair, ears, neck and the background above the forehead as skin, and the palette spent its skin budget on them — producing three tonal steps on a face instead of five or six, and bleeding brown into the surroundings. The second version samples each person's *own* face (the band between eyes and mouth) and admits pixels by **Mahalanobis distance in the (a\*, b\*) chromaticity plane**, not full Lab, so skin in shadow still matches while dark hair is excluded by a wide lightness band. That was better and still wrong: the mask called blonde and light brown hair skin, and because the skin layer learns its target colour *from that mask*, cluster centres yellowed by **+6.9° and +6.8°** on two blonde samples versus +1.2° where hair was dark. The third stage, `skin_core`, re-checks the mask against that person's own face sample. Contamination fell from **36.4° to 5.0°**.

**A filter one layer down silently invalidated the logic above it.** While tuning the face threshold, the YuNet instance was being constructed with its own minimum confidence set to the same value we were deciding with — so the detector itself filtered at 0.85 and a 0.82 hit never reached Python at all. The instance is now created with a low threshold and the real decision is made in our own code. This was one of three occurrences of the same pattern in the project: a symptom masked by oversizing a different layer. Thick ink hid the double outline; an area threshold hid that the thickness test was lying; a tone ceiling hid that the mono ladder was not monotonic in its request. Each surfaced only when the masking layer was refined for some unrelated reason.

**A silent colour-space bug.** Using CIELAB rather than RGB was a baseline decision, not a discovery — perceptual uniformity is the point of the space. The real error was quieter: the code used OpenCV's 8-bit LAB, which scales L by 2.55 and therefore weighted lightness *against* hue in every distance calculation. Months of colour decisions were subtly wrong before it surfaced. `palette.to_lab` now returns true CIELAB.

**The bug that got worse with a bigger box.** The client asked a precise question: would a larger palette reduce the greenish cast on skin, or was it caused by the 30-colour limit? Neither. The kit is decided in two places, and only one was disciplined: the snap step used hue-weighted distance and a skin lock, but the fill-in step that tops the kit up to exactly N used plain nearest-neighbour — where, at low chroma, hue is almost free. Added paints sat up to 30 hue-weighted units from the pixels handed to them: cold grey onto a warm arm, green beside a face. On one photo three of five added paints were neutral greys. Measured skin-green excess was **4.91 / 4.44 / 6.02 percentage points at 30 / 39 / 48 paints** — worse with more paints, because a bigger box runs the undisciplined path more often. That refuted the client's hypothesis that the 30 limit was at fault. With the fill-in using the same hue-weighted metric and refusing splits it has no paint for, the numbers became 0.64 / 0.53 / 1.80.

**Hue-weighted matching, because equal ΔE is not equal error.** Plain CIELAB distance treats a hue error and a lightness error as equally bad; the eye does not. Skin is read by its hue, and a dozen degrees turns a warm brown into olive. Measured case: a selected paint sat at **79° against the photograph's 63°**, with perfect lightness, chosen only because the two better paints were already assigned and the substitution looked cheap at 3.2 ΔE. A hue weight of 2.2 makes that trade honest. Skin additionally gets its own shelf: measured across every sample with faces, real skin sits between roughly 11° and 68° hue and never below chroma 10, so paints in that range form a sub-inventory and a skin-coloured cluster is restricted to it. The rule binds on the cluster's own colour rather than the mask, which also catches skin the mask missed — the grey patches on arms that were not in the photo. Skin painted off the scale fell from **19.3% to 0.0%** on the worst sample.

## 4. How I knew it worked

**The first decision was what to measure against, and it was not taste.** The obvious benchmark for "is this good enough" is a subjective judgment, or an invented quality target. I used neither. The shop already produces these canvases by hand and sells them — so their own delivered output is a measurable reference rather than an opinion. Normalised for resolution and counting only regions above our own paintability floor, their canvases average **1763 regions; ours started at 1142**. We were at 65% of a real, sold product. That number explained a review comment that had been returning for rounds ("the background could be more detailed") and located the cause precisely: our `MAX_REGIONS` ceiling of 2200 was an invented constraint that held us below the shipping product. Raised to 4500, our average became 1800 — **0.98× theirs**.

**The same discipline killed a plausible wrong theory.** The client's samples contain tens of thousands of unique colours, from which we first concluded that they paint with the full 806-colour inventory while we use thirty, and that this was the source of flatness. Measured, it was false: **95% of their pixels are covered by 19–28 colours**, and the dominant ones match 29–30 recipe paints almost exactly. The tens of thousands were antialiasing blends along region boundaries — an artifact that exists only on screen. So the fix was not more paint. It was `render.soften`: a light Gaussian on the assembled preview, rendered at double size. The template, the numbering and the 30-paint kit are unchanged; the richness happens on the screen, exactly as it does in their own product.

**Expert review at scale.** Over 2,000 outputs were reviewed by the graphic designers and the founder — the people whose manual work the system replaced. Every disagreement fed back into the rules. Where a category kept failing, it got its own rule set.

**The company's own rejection history as ground truth.** The quality standard was not the designers' opinion, and this took time to learn. Users had rejected technically accurate images because they disliked the shades, and the most common request sent to the designers was "improve my skin colour." The real standard belonged to the customers, and the most sensitive part of it was how they saw themselves. **The image rejection rate fell from about 30% to about 8%.**

**A test that guards a business decision.** The pipeline supports a fast mode that stops after quantisation. `test_fast_mode.py` proves the fast output is a *prefix* of the full run, with a byte-identical pixel image. That is not a performance test; it is the guarantee that what a reviewer approves is exactly what production reproduces. If it ever failed, the fast path would be worthless.

**The quality flag reports the worst substitution, not the average.** Sometimes the inventory has nothing near a colour the photo needs and the system must substitute something visibly wrong. It does not fail; it flags. The flag is driven by the single worst ΔE in the job, because one badly wrong colour on a face ruins a canvas even when the other twenty-nine are perfect — an average would hide exactly the case that matters. Above 10 units, a person looks before it prints.

**Regression protection as a measurement, not a promise.** The control panel exposes 86 tunable parameters so the client can question them without a developer, which creates an obvious hazard: approved output must not drift. A golden-file tool records a fingerprint of every artifact for all 19 sample photos at 30, 39 and 48 paints — 57 runs — and re-running must reproduce all 57 exactly. It passed **57/57 identical** after the panel was built. Presets store only what changed, so a preset can never shadow an approved value, and changing an approved default is deliberate work the panel cannot do.

**Honest limits.** Monochrome and near-monochrome photographs remain the weakest category — when the source has little chromatic range, a discrete palette has little to work with. They were brought to acceptable, not good.

## 5. Running it in production

The pipeline began as a CLI and now also runs as a service: a FastAPI application with a process pool, a file-based job store, and a control panel for tuning parameters. The API does not execute the pipeline inside the request — it validates, converts HEIC, applies EXIF rotation, opens a job directory and returns a job id immediately, because a 45–90 second HTTP request would time out at nginx and at the caller. A separate worker process runs the job; the caller polls.

**A job is a directory on disk.** Nothing lives in memory, so the API process can restart without loss, an operator can open a job, copy it, or hand it to support, and the whole store survives being moved with `scp`. Authentication sits in nginx with the app listening on localhost, so the reverse proxy is the only thing that can reach it.

**Workers are long-lived, deliberately.** A fresh process per job would guarantee isolation, but reloading the 168 MB model costs about 9 seconds — longer than an entire fast run. Instead each worker snapshots the pipeline constants at startup and restores them at the **start** of every job rather than the end, so a job that dies halfway cannot leave its settings behind for the next one.

**Cancelling a running job does not work the obvious way.** The first timeout implementation called `future.cancel()`, which only removes queued tasks — a running job ignores it entirely. I verified this rather than assuming it. The current implementation raises SIGALRM inside the worker, with the pool-level timeout kept only as a fallback for the case where the job is stuck inside a single long native OpenCV call that a Python-level signal cannot interrupt.

**Memory is dominated by the ONNX arena, not by image size.** Measured stepwise in one process: 16 MB at interpreter start, 136 MB after imaging libraries, 167 MB after the project modules, **2179 MB once detection has run**, 2937 MB for a full pipeline. A 1.4 MP photo peaks at 2.76 GB and a 16.8 MP photo at 2.88 GB — essentially independent of input size. So the sizing rule is roughly 3 GB per worker regardless of what customers upload.

**Throughput does not scale linearly, and measuring it corrected my own estimate.** Same 16 MP photo, 48 colours, increasing concurrency:

| Concurrent jobs | Time per job | Effective throughput |
|---|---|---|
| 1 | 42.9 s | 1.00× |
| 2 | 46.5 s | 1.85× |
| 3 | 57.3 s | 2.25× |
| 4 | 65.0 s | 2.64× |
| 8 | 106.1 s | 3.24× |

Eight concurrent jobs on ten cores returned 3.2× throughput, not 8×; going from four workers to eight bought only 23%. Two causes are mixed here and this machine cannot separate them: the test machine has 4 performance and 6 efficiency cores, which an x86 VPS does not, and memory bandwidth saturation, which it does — the 34% slowdown already visible at three concurrent jobs occurs while everything still fits on the four fast cores. Each job runs large array operations over a 3 GB working set.

The practical consequence: **do not set the worker count equal to the core count.** On a 4 vCPU / 16 GB machine, three to four workers is realistic. I also revised my own capacity figure down — from a theoretical 160 jobs/hour to about 105 on the four-core machine. Still far above the business's actual volume, but the proposal deserves the real number.

*These measurements were taken on a development machine with heterogeneous cores. The curve will differ on x86 and must be re-measured on the first production host before the worker count is fixed.*

## 6. Cost

- **Team time per order: ~1 hour → ~5 minutes.** Six manual touchpoints reduced to one — logistics assembling the order.
- **Marginal cost per image: effectively zero.** No inference billing, no API calls. The pipeline is CPU time on a VPS.
- **Two real speedups, both by-products of quality fixes rather than performance work.** The region-merging loop went from 171 s to 7 s on its worst case by operating on per-component bounding boxes instead of the whole image, and boundary tracing became roughly 25× faster (to 0.12 s) by following boundaries on the inter-pixel grid in a single pass instead of scanning the image once per palette entry. The second was a side effect of eliminating double lines where two colours meet.
- **Honest note: this project was never profiled.** There is no cProfile, line_profiler or pyinstrument anywhere in the repository, and no commit in its history is about speed. The only timing recorded during development is total elapsed time per job. The breakdowns above were measured afterwards, deliberately, when sizing the service.

## 7. What I would do differently

**Profile before optimising, not after.** Both speedups above were accidents of quality work. That they were large suggests there is more, and I currently could not say where without measuring.

**Split the pipeline into two phases.** The fast path (to quantisation) costs 4.3 s and produces exactly the preview a customer needs; the full run to print-ready PDF costs 44.8 s. Only paying customers need the second. At 500 previews a day and 10% conversion, that is 6.3 CPU-hours versus 1.2 — a fivefold reduction, and eightfold at 3% conversion. The mechanism already exists and the byte-identity test already guards it; the business flow around it (order webhook, designer approval) is designed but not yet built. Two things I have not solved: a public preview endpoint is an internet-facing CPU-burning surface that needs rate limiting beyond the current 40 MB upload cap, and non-converting previews need a retention policy.

**Run detection once for three colour counts.** Of the 4.3 s fast path, 2.56 s is detection, which is entirely independent of colour count. Today `process_image` re-detects on every call, so showing a customer all three versions costs 12.9 s instead of a possible ~7.4 s.

**The mono branch taught me the most about deleting work.** Concentric rings appeared on smooth faces in black-and-white photos. The first diagnosis was structural — segment by shape first, then assign tone — so a SLIC superpixel stage was built. It was wrong: the rings came from evenly spaced tone steps crossing smooth gradients, and once the steps followed each image's own tone distribution the superpixels were unnecessary. Their cost was severe (every cell fell below the paintability floor, so every small facial feature merged into its neighbour), so the stage and five constants were deleted. Three other suspects were cleared by measurement in the same round.

**Attack monochrome deliberately.** It was brought to acceptable and left there. It is the one known-weak category and deserves its own treatment.

**Keep the deterministic decision.** With cheaper models available now, I would make the same call. The reasons were never cost — they were physical validity, correctability rule by rule, and zero maintenance. None of those have changed.
