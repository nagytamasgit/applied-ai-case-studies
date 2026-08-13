# cook-solutions / "sear" — Cooking Detection from Audio

> A dependency-light audio pipeline that detects cooking events from sound alone,
> using an STFT-based heuristic (numpy only, no `librosa`) as a fast Tier-2 stage.

| | |
|---|---|
| **Date** | 2026-08 |
| **Domain** | Audio signal processing / event detection |
| **Role** | Solo builder |
| **Status** | Prototype — code-complete, pending real-kitchen data |
| **Stack** | Python, numpy (STFT), pytest harness; deliberately no `librosa` |

## Problem

Detect what's happening on a stove from **audio alone** — the sear, the sizzle, the
state changes of cooking — reliably enough to drive a downstream decision. The
constraints that shaped the design: keep the dependency footprint minimal (no heavy
DSP/ML libraries), keep it testable without a live kitchen, and reach a defensible
accuracy bar before committing to a heavier approach.

This is a 14-day rebuild that supersedes an earlier prototype (`cookai`), with the
canonical home at [`nagytamasgit/cook-solutions`](https://github.com/nagytamasgit/cook-solutions).

## Approach

A **tiered detection** design, so cheap stages handle the easy cases and only
ambiguous audio escalates:

- **Tier 1 / SMOKE ingest** — get audio in and through the harness end-to-end first,
  proving the plumbing before any clever detection.
- **Tier 2 heuristic** — a hand-built detector computing a **short-time Fourier
  transform (STFT) with numpy** and thresholding spectral features. No `librosa`:
  the STFT is implemented directly to keep the dependency surface tiny and the
  behavior fully inspectable.

Thresholds are intentionally left as tunables to be set against real recordings
rather than guessed up front.

## Implementation

- **Test harness + tooling** — built first, so every detection change runs against a
  repeatable suite (31 tests green at the Day 1–7 checkpoint).
- **SMOKE ingest** — the ingestion path, validated before detection logic.
- **Audio Tier-2 heuristic** — numpy STFT + spectral thresholding, no external DSP
  library.
- **Reproducible checkpoint** — Days 1–7 code-complete at commit `7277ac1`.

### What's deliberately deferred

The next phase is **KITCHEN**: record 30–40 real takes, tune the Tier-2 thresholds
against them, and make the **Day-7 go/no-go decision at ≥85% accuracy**. That gate
is intentionally *not* pre-judged in code — the heuristic exists, but the numbers
that decide whether it's good enough only come from real audio.

## Results

| Metric | Baseline | Result |
|--------|----------|--------|
| Ingest → detect pipeline | none | end-to-end, code-complete |
| Test suite | 0 | 31 tests green |
| DSP dependencies | `librosa` (heavy) | numpy only |
| Detection accuracy | — | *pending* — gated on ≥85% against kitchen data |

## What worked / what didn't

- ✅ Building the test harness before the detector made the audio logic safe to
  iterate on and kept the Day-7 checkpoint honest.
- ✅ Reimplementing the STFT on numpy dropped a heavy dependency and made the whole
  detector inspectable and portable.
- ⚠️ The real bottleneck is **data, not code** — thresholds can't be trusted until
  they're tuned against 30–40 real recordings.
- ❌ The ≥85% accuracy bar is genuinely unproven until KITCHEN runs; the pipeline is
  ready but the decision it exists to make is still open.

## Lessons

For a perception system, **stand up the harness and the ingest path before the model**,
and don't let a threshold masquerade as "done" until it's been measured against real
data. A minimal, self-implemented signal path (numpy STFT vs. a heavyweight library)
buys inspectability and portability that pay off exactly when you're debugging why a
real recording didn't trigger.
