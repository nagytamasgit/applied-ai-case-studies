# Oldal AI — AI Website Builder

> An AI-operated website builder that turns intent, a live URL, or a Figma file into
> an editable site — with click-to-edit direct manipulation and one-click export to
> a real WordPress theme.

| | |
|---|---|
| **Date** | 2026-07 – 2026-08 |
| **Domain** | Generative UI / design-to-code / agents |
| **Role** | Solo builder |
| **Status** | Shipped (production) |
| **Stack** | LLM-driven codegen, structured `SiteModel` IR, Figma API, WordPress FSE export, Docker on Hetzner, Coolify |

## Problem

Non-technical users want a real website without writing code, and existing
"AI site builders" tend to produce a locked-in black box you can't edit precisely
or take with you. The goal was a builder that (1) starts from whatever the user
already has — a description, an existing site, or a Figma design — and (2) produces
output that is both **directly editable** in the browser and **portable** out to a
standard platform (WordPress).

## Approach

Rather than have the model emit raw HTML on every turn, the system centers on a
structured intermediate representation — a **`SiteModel`** — that the AI reads and
mutates. This makes edits deterministic and reversible, and gives a stable target
for both the visual editor and the exporter.

Three ingestion paths feed the same model:

- **From intent** — the AI generates the site and a design system from a prompt.
- **From a URL** — clone-from-URL ingests an existing page into the `SiteModel`.
- **From Figma** — import a Figma file via the Figma API.

## Implementation

- **Click-to-edit inspector** — direct-manipulation editing: select any element on
  the canvas and edit it in place, backed by the `SiteModel`. Shipped across four
  milestones, verified end-to-end.
- **Clone-from-URL** — pull a live page into an editable model. Live in production.
- **Figma import** — bring designs in through the Figma API. Built and deployed;
  gated behind a Figma OAuth app (`FIGMA_CLIENT_ID` / `FIGMA_CLIENT_SECRET`) before
  it can be switched on.
- **WordPress export** — `SiteModel → FSE (Full-Site-Editing) theme`, so a generated
  site leaves as a standard, self-hostable WordPress theme rather than a lock-in.
  Phase 1 shipped to production.
- **Design system + onboarding** — a coherent design-system layer and first-run
  onboarding, deployed via the Coolify API.

### Infrastructure

Two environments run on Hetzner boxes. Deploys are done by syncing a git archive
and rebuilding the Docker compose stack (no `.git` on the server), and — where SSH
key auth was rejected — pushed through the Coolify API instead. The public app was
migrated off GCP to a Coolify-managed host, with daily backups enabled.

## Results

| Metric | Baseline | Result |
|--------|----------|--------|
| Ingestion paths to an editable site | 0 | 3 (intent / URL / Figma) |
| Editing model | regenerate-on-edit | direct click-to-edit on a structured IR |
| Portability of output | locked-in | exports to a standard WordPress FSE theme |
| Production status | — | live, daily backups, migrated off GCP |

## What worked / what didn't

- ✅ A structured `SiteModel` IR (instead of raw HTML per turn) made edits precise,
  reversible, and gave the exporter and visual editor one shared source of truth.
- ✅ Treating export as first-class (WordPress FSE) turned "AI builder" from a
  lock-in into something users can actually own and move.
- ⚠️ Deployment friction was real — SSH key auth was rejected on one host, so
  deploys had to route through the Coolify API instead of a git webhook.
- ❌ Figma import can't ship "on" yet: it needs a registered Figma OAuth app and
  client credentials, so the capability sits dark behind a flag.

## Lessons

For AI-generated software, invest in a **structured intermediate representation**
before investing in prettier generation. The IR is what makes editing deterministic,
export portable, and the whole system debuggable — the model becomes a mutator of
state rather than a one-shot generator you can't correct.
