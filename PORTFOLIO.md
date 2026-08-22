# Engineering Portfolio

**Scope:** 40 private repositories, 38 with shipped code, created between October 2025 and August 2026
**Role:** Solo on all but two of them — architect, engineer, and in most cases the person who deployed it
**Domains:** Multi-tenant SaaS, marketplaces, mobile apps and games, AI platforms, automation
**Stack:** TypeScript/Next.js, Flutter/Dart, Python, PHP/WordPress, PostgreSQL, Docker on self-managed VPS

An index of roughly the last year of building. The [four case studies](README.md) in this repository
go deep on single systems and are the better read if you have twenty minutes. This document is the
breadth behind them: what exists, what it was for, and what state it is actually in.

---

## What this document is, and what it isn't

It is an inventory with judgment applied, not a portfolio in the sense of selected highlights. Every
repository I own from the period is here, including the abandoned ones and the duplicates, because a
list that quietly omits its failures is not evidence of anything.

**It is not a career history.** Eleven months, October 2025 to August 2026. Earlier work isn't here.

**The case-study systems are not in this list.** The video assessment pipeline and the
OmotenashiJobs platform covered in studies 01 and 04, the voice agent in 02, and the image pipeline
in 03 are client-owned and live in repositories I don't hold. `jp-jobs` below is a *different*
Japanese hospitality product and should not be mistaken for them. Two of the projects here do have
their own case studies — Oldal and `sear` — and I've linked rather than repeated them.

**Status words mean specific things**, and I have tried to use them honestly:

| Word | What it actually means |
|---|---|
| **In use** | Real users or a paying client depend on it right now |
| **Shipped** | Published to an app store and downloadable |
| **Built** | Complete and verified against tests, but nobody outside has used it |
| **Blocked** | Code-complete; waiting on a licence, a credential or a client decision |
| **Research** | Deliberately unfinished — the point is the method, not a product |
| **Archived** | Superseded or abandoned; listed for provenance |

The distinction that matters most is **Built** versus **In use**. A lot of what follows is Built. A
system that passes its own tests has proven considerably less than a system that has survived
strangers, and I would rather say so than blur the two.

---

## 1. Things real people use

### Lumino Block Puzzle

**Repository:** `toms-blocks`
**Status:** In use — v4.52.2 (build 156), Google Play production at 100% rollout
**Role:** Solo
**Stack:** Flutter, Riverpod, Firebase (Auth, Firestore, Analytics, App Check), Google Mobile Ads, RevenueCat, Play Billing 8.3

The most commercially real thing here: a block-puzzle game with a live-ops content calendar, an
in-app economy and paying users, in 11 languages across 12 locales.

The engineering interest is in the parts that aren't the game. A `GameModeStrategy` abstraction
carries several modes including a complete hexagonal board — its own coordinate system, grid, clear
detection and renderer, sharing nothing with the square board but the interface. The puzzle campaign
ships a **solver**, used not to help players but to prove at build time that a hand-authored puzzle
is actually solvable, which is the kind of check that costs a day and saves a support inbox. The
economy keeps a purchase ledger with account-isolation tests, because the failure mode of getting
that wrong is refunding strangers.

43 test files cover the engine, hex geometry, scoring, serialisation, rotation edge cases,
challenges, achievements and purchases. The Play Billing 8 migration was promoted to 100% ahead of
Google's August 2026 deadline.

The honest caveat: I can tell you the code is well covered and the migration landed on time. I have
not put retention or ARPU figures in this document because I would rather omit a number than publish
one I can't source cleanly.

### Grundi — the live OpenCart marketplace

**Repository:** `grundi-opencart`
**Status:** In use — live at grundi.hu on a Hetzner VPS
**Role:** Solo, on an inherited codebase
**Stack:** PHP 7.4, MariaDB, OpenCart 3.0.3.8, Webkul Marketplace, Journal3, rootless Podman, Caddy

A multi-vendor marketplace I did not build and then had to make safe to change. It arrived with no
version control, no local environment, and — the reason the client called — no way to work out what
to pay its sellers.

Phase 0 was unglamorous and mattered most: the store under git with secrets and runtime artefacts
excluded, a dated security review with every finding triaged and remediated, and a containerised dev
stack so the whole thing boots from one command instead of existing only in production. You cannot
responsibly change a system you cannot reproduce.

The substantive work is the **settlement engine**. Sellers collect cash on delivery; the marketplace
collects card payments; the two have to be netted per seller per period, with shortfalls billed
back. I built it as a pure calculator with unit tests over idempotent tables, then verified it
against real production data before it was allowed near a payout. On top of that sits an invoicing
abstraction over Számlázz.hu Agent-XML and Billingo v3 with a dry-run mode, per-seller payment
toggles enforced at checkout, XLSX catalogue import with an idempotent stock-decrement webhook, an
in-house support ticket queue, and Hungarian lifecycle email that is **at-most-once by construction**
— guard rows in the database rather than a hope that cron doesn't double-fire.

Ten phases delivered, two QA passes cleared. What remains is not code: carrier credentials,
production SMTP, a SimplePay contract and legal sign-off all sit with the client.

### Anfisa Beauty — replacing a rented platform

**Repository:** `getcourse`
**Status:** In use — VPS staging, migration pending
**Role:** Solo
**Stack:** WordPress, WooCommerce, WooCommerce Subscriptions, LearnDash, Amelia, Fluent Forms, TranslatePress, Podman, Caddy, Playwright

A subscription wellness school — face yoga, massage, meditation, nutrition — moving off GetCourse
onto infrastructure the client owns. The product requirements are unremarkable; what I would point
at is the harness.

`stack.sh` gives up, down, destroy, arbitrary WP-CLI passthrough, backup, and **`restore-drill`** —
a command that takes the latest backup and proves it restores by standing the whole site up on a
scratch stack on another port. An untested backup is a belief, not a backup, and for a business
whose entire course catalogue and membership base lives in one database that distinction is the
whole job. A Playwright smoke suite gates every deploy; each phase adds specs to it.

Phases 0–3 and 5–7 are done on staging: commerce, LMS, memberships and trial, booking, funnel forms,
EN/HU. Phase 4 video gating is code-complete. Everything still open is commercial rather than
technical — Stripe keys, three plugin licences, a Bunny account, Zoom credentials, and GetCourse
admin access before content can be migrated. I've kept those listed as blockers rather than quietly
calling the project finished.

### Oldal — AI website builder

**Repository:** `app.oldal.ai` · marketing site `oldal.ai`
**Status:** In use — production on Hetzner
**Role:** Solo
**Stack:** Next.js 15, PostgreSQL + Drizzle (20 migrations), Anthropic Claude, Hono, Docker + Caddy

**Covered in depth in [case study 0001](case-studies/0001-oldal-ai-website-builder/README.md)** — a
multi-tenant builder where agencies and clients produce hardened static sites by prompting, with a
structured `SiteModel` intermediate representation, click-to-edit direct manipulation, ingestion from
intent, a URL or Figma, and export to a real WordPress FSE theme.

What the case study doesn't dwell on, and belongs in an engineering index: the output is put through
a capability policy *after* generation — JavaScript off means no `<script>` tags or inline handlers
survive, generated JS goes through a static scanner, and a strict CSP is injected into every page.
There is a `verify-harden-bypass` suite whose only job is to attack the hardener. Billing meters the
real Claude API cost per generation and charges it on with a markup against a prepaid balance, which
means the cost accounting had to be correct before the product could be sold at all. Roughly 200
assertions across 15 verification suites run against PGlite and a mock LLM, so the platform can be
validated end to end without spending money.

### XPRIZE — an AI employee with an approval gate

**Repository:** `xprize`
**Status:** In use — production, Cloud Run then Coolify
**Role:** Solo
**Stack:** Next.js, PostgreSQL append-only event log, pg-boss, Google Gemini, Firebase Auth

A system a small business hires for outcomes rather than features: fill the calendar, issue and
chase invoices, become findable. Three loops, each ending in an action taken on the business's
behalf — which is precisely why none of them execute unsupervised.

The core is an append-only event log with projectors materialising read models. The `events` table
rejects UPDATE and DELETE **even for the service role**, projections are idempotent on replay, and
row-level security is proven by test to stop a workspace reading *or writing* another's rows. Every
agent action runs propose → approve/reject → execute with per-workspace autonomy policies, and the
whole approval history is the evidence trail.

The design decision I'd defend hardest is the **agent-addressable entity API** — live JSON of
services, prices, hours and real availability at a stable URL, built to be consumed by other AI
agents rather than by a browser. Onboarding works by archaeology: point it at an existing website
and it reconstructs a configured workspace.

This one shares a shape with the voice agent in [case study 02](02-voice-eval-system.md) — a system
fully capable of running autonomously, with the switch deliberately left off.

### Cossy — social network for cosplayers

**Repository:** `cossy`
**Status:** Shipped — v1.0.0+21, Google Play internal track
**Role:** Solo, for RevenueCat Shipaton 2026
**Stack:** Flutter, Supabase (EU) with 30 migrations and 8 edge functions, RevenueCat, Cloudflare R2, ONNX moderation, OneSignal, Sentry, PostHog

A social network with the cosplay-making toolkit built in: feed, profiles, WIP posts, DMs and group
chat, a project tracker with nested checklists and budgets, conventions, and a coins economy where
users award each other.

Three things I'd single out. Moderation is self-hosted ONNX rather than an API, because a social app
that pays per image to a moderation vendor has its costs coupled to its growth in the wrong
direction. Account deletion is a first-class edge function, not a support email. And there is a
dated security audit in the repository alongside dedicated hardening migrations and an explicit RPC
surface migration — written when the app was small, which is the only time that work is cheap.

Honest position: it is on an internal track with seeded launch content, not a live community. The
engineering is real; the audience is not there yet.

### Ingatlanspanyol — Costa del Sol property search

**Repository:** `IngatlanSpanyol/ingatlanspanyol`
**Status:** In use — Coolify, scheduled sync
**Role:** Solo
**Stack:** Next.js 16, Drizzle + PostgreSQL, next-intl (EN/ES/HU), Google Maps, Playwright, Vitest

A trilingual property portal that syncs listings from an upstream Supabase feed into its own
database. Standard search-and-detail product, with the interesting part in the ingestion: an
admin-secret-protected sync endpoint, a `sync_runs` audit table with a history API, an optional IP
allowlist and an explicit `TRUST_PROXY_HEADERS` opt-in rather than trusting forwarded headers by
default.

One bug worth recording because the fix is in the Dockerfile as a comment: the scheduled sync called
the app through its own public URL and failed every time, because a container cannot reach itself
through the reverse proxy. It calls localhost now. Small, and the sort of thing that only shows up
in a real deployment.

A companion WordPress plugin lets the search embed into an existing WP site.

### jp-jobs — Japanese hospitality job board

**Repository:** `jp-jobs`
**Status:** In use — v3.6.49
**Role:** Solo
**Stack:** Next.js 16, Prisma + PostgreSQL (12 migrations), NextAuth v5, next-intl, Anthropic and OpenAI SDKs, Stripe, Playwright, gitleaks in CI

A bilingual marketplace connecting foreign candidates with Japanese hospitality employers. Three
role surfaces — candidate, employer, admin — with the workflows that make this market specific
rather than generic: visa case tracking with assignment to a lawyer role, and generation,
translation and PDF export of the **shokumu keirekisho**, the Japanese-format career history
document, with usage metering on the AI calls.

Beyond that it is a large, ordinary, well-worn CRUD platform: job posting and moderation, applicant
pipelines, a candidate discovery marketplace, Stripe checkout, a translated blog, map search,
audit logs, rate-limit event tracking and a performance budget checker in CI.

To repeat the earlier warning, because the confusion is easy: this is **not** the OmotenashiJobs
platform from case studies 01 and 04. Different product, different codebase, no video assessment
in it.

---

## 2. Built and verified, but nobody outside has used it

This is the largest group, and the distinction from section 1 is the point of separating them.
Everything here passes its own tests. None of it has met a stranger.

### Sentinel — WordPress security and operations platform

**Repository:** `sentinel`
**Status:** Built — MVP complete, not deployed to customers
**Stack:** Next.js 15, pg-boss worker, PostgreSQL 16 + Drizzle, zero-dependency PHP plugin

A console for agencies maintaining fleets of WordPress sites, plus an agent plugin that runs on each
site. The plugin has **no dependencies at all**, which is deliberate: anything you install on
someone else's production site is something you are now responsible for patching.

The wire protocol is the part I'd show someone. Every request carries
`HMAC-SHA256(secret, "{timestamp}.{nonce}.{raw_body}")` with a ±300 s window and single-use
per-site nonces pruned by the worker. The per-site secret is stored AES-256-GCM encrypted server
side and derived from `AUTH_KEY` on the WordPress side, so neither end holds the other's plaintext.
Pairing hands back the key material exactly once. TOTP is implemented by hand against RFC 6238 and
unit-tested rather than pulled from a library — a decision I'd defend for a security product, since
it is a small enough spec to own and verify.

What is missing is the commercially important half: uptime and SSL monitoring, Wordfence feed
import, and notifications. The security foundation is finished; the monitoring product on top of it
is not.

### SoforApp — ride-hailing on a bidding model

**Repository:** `soforapp`
**Status:** Built — full flow works locally against seeded accounts
**Stack:** NestJS + Prisma + PostgreSQL, Socket.IO, Redis, two Flutter apps, Codemagic

Passengers request a ride and transport *companies bid* for it, instead of an algorithm dispatching.
The whole lifecycle is state-machine validated — request, broadcast, bid, accept, assign driver,
en route, in progress, complete, rate — with realtime room-based events per company and per ride.

The rule I like best is a business constraint expressed as a guardrail: a company cannot bid unless
its wallet holds at least `bid + 2000`, so nobody wins work they can't be charged commission for.
Around that sits invoicing through Billingo with a retry service and reconciliation, a
country-specific billing policy module, team and driver management with OTP activation, and ride
conversations.

Honest status: nearby-company matching is mocked as "first N companies" and the payment provider is
mocked at the wallet layer. It is a complete demonstration of the model, not a deployed service.

### Grundi — the greenfield rebuild

**Repository:** `grundi-`
**Status:** Built
**Stack:** Next.js storefront, NestJS + Fastify API with OpenAPI, BullMQ workers, Drizzle, Zod contracts, OpenSearch, MinIO, Redis

A from-scratch replacement for the OpenCart marketplace in section 1 — same business, contract-first
architecture, six packages sharing typed Zod contracts.

The design decision worth noting is the dual data store: an in-memory implementation for fast local
work and tests, and PostgreSQL as canonical, with both sharing order numbering, moderation gating,
settlement rules, session handling and SKU constraints. Where they diverge, Postgres wins by
declaration in the README. That is a rule that has to be written down, because the alternative is
discovering the divergence in production.

Running two systems for one business is a real cost, and the honest read is that the live OpenCart
store is what earns money while this is what a maintainable version would look like.

### Eutory — European brand directory

**Repositories:** `eutory` (platform) · `eutory-collector` (content pipeline)
**Status:** Built
**Stack:** Next.js 16, Prisma 7 + PostgreSQL, NextAuth, Playwright — and Python, Typer, trafilatura, OpenAI-compatible LLM for the collector

A directory, deals and comparison site for European brands, with a company self-service portal, a
scoped-API-key public API, first-party analytics, an uptime monitor and an SEO audit script.

The collector is the half I'd defend. It researches a brand from public sources, drafts a content
document against a versioned contract, then **validates it** — required fields, enums, ISO dates,
URL shapes, locale codes, canonical slugs, GEO objects — and explicitly **refuses to invent database
IDs**. Foreign keys stay as placeholders until resolved from a local mapping file in a separate
step. An AI content pipeline that fabricates a plausible-looking foreign key produces data that
imports cleanly and is wrong, which is the worst available outcome.

### EuroSzaki — skilled-trades job board

**Repository:** `euroszaki`
**Status:** Built
**Stack:** Laravel, Blade, Filament admin, Laravel Cashier, MySQL

A job board placing skilled tradespeople into Germany, Austria and the Netherlands. Listings with
tags, pricing tiers and upsells; candidate profiles and applications; employer accounts;
conversations.

The half that took the work is acquisition rather than the board itself: an Apify/Indeed import
service with auto-import and expiration-check console commands and an import statistics dashboard,
a résumé parser for uploaded CVs, an AI translation service backing a fully localised interface,
click tracking with attribution on outbound listing clicks, and marketing automation — newsletter,
Filament-managed email campaigns, a weekly digest command and automated Facebook Page posting. A
job board with no listings and no traffic is a schema, so most of the engineering went there.

`jobscore` is a byte-identical duplicate of this repository.


### Two Lies and a Truth

**Repository:** `liesandtruth`
**Status:** Built
**Stack:** Flutter, Riverpod, Freezed, Firebase (Auth, Firestore, Functions, Messaging, Remote Config), AdMob, RevenueCat

A social deduction game in five modes — realtime multiplayer, async challenges, daily, solo trivia
ladder, and a couples mode. Rules are server-authoritative in Cloud Functions with hard invariants:
exactly three statements, one valid true index, no vote after the reveal deadline, no round
finalised without votes. Eleven callable functions, and room lifecycle, presence, recovery,
matchmaking, rate limiting and blocking each with their own test file.

Two details I'd point out. There is a **Crashlytics PII scrubber with its own unit test** — crash
reports from a game where users type personal statements about themselves will contain those
statements unless something removes them. And the analytics layer ships BigQuery SQL for retention,
conversion and monetisation rather than leaving that to be written later under pressure.

### Country Guide

**Repository:** `country-guide`
**Status:** Research / partially built
**Stack:** Flutter app, Node/TS API, git-versioned JSON packs, zero-dependency schema validator

An offline app telling travellers how to behave and what will get them arrested, built on one
versioned data layer intended to serve three products — the app, a licensable API, and white-label
builds.

The app is a stub. The reason it's here is the **editorial gate**, which is the actual product. Each
content field carries severity, sources, confidence, a hedge string, `ai_generated` and
`human_reviewed`. A dependency-free validator refuses to publish if any field at
`severity >= caution` has empty sources or is still un-reviewed, if a low-confidence field lacks a
hedge, or if the pack's stored overall risk disagrees with the value derived from its own severity
spread. Publish mode makes those hard errors that block CI; staging mode downgrades them to warnings
so drafts can move.

The first pack, Japan, is committed in the deliberately blocked state — AI-drafted, un-reviewed, and
correctly refused by its own gate. Given the content is legal advice, a pipeline that *cannot*
ship an unsourced claim seemed more valuable than one that ships faster.

### NexusPress — AI WordPress block generation

**Repositories:** `aiwp2` (platform) · `aiwp-plugin` (companion plugin v1.0.0)
**Status:** Built
**Stack:** Next.js 14, Prisma + MySQL, Google Gemini, Stripe, JWT, WordPress plugin (PHP 7.4+, GPL-2.0)

Generates brand-consistent Gutenberg blocks from a prompt and injects them into a connected
WordPress site in one click. A per-account "brand DNA" — colours, fonts, voice, industry —
conditions every generation; the plugin exposes a JWT-authenticated REST endpoint and creates the
result as a **draft**, never publishing directly to someone's live site.

Credits and subscriptions run through Stripe with full webhook handling. Unit suites cover JWT,
password hashing, sanitisation and validation schemas.

### Webautomatizáció — productised automation shop

**Repository:** `webautomatizacio`
**Status:** Built
**Stack:** Next.js storefront, Medusa with a custom module, PostgreSQL, Puppeteer, Docker + Caddy

A Hungarian agency storefront selling n8n workflows as catalogue items, with a custom Medusa module
carrying automation requests, offers, customers and knowledge articles through seven migrations. The
pipeline runs request → review → offer generated and rendered to PDF → sent → approved → invoice →
tokenised onboarding form.

Notable for what it refuses to do: **no card payments and no Stripe**, by design. Payment is by
invoice, there is no card form anywhere, and that is documented as a deliberate security decision
for an MVP rather than a missing feature.

---

## 3. Research, tooling and infrastructure

### sear — a single-moment cooking coach

**Repository:** `cook-solutions`
**Status:** Research — blocked on kitchen time, not on code
**Stack:** Python, numpy (hand-rolled STFT, no librosa), OpenCV, uv, pytest

**Covered in [case study 0002](case-studies/0002-cook-solutions-audio-detection/README.md).** One
mistake — the pan wasn't hot enough when the food went in — detected within seconds and corrected
while it still can be. A cold pan is an audio problem rather than a vision one, which makes the
whole thing orders of magnitude cheaper and immune to a hand occluding the pan.

The part that belongs in an engineering index is the discipline around the measurement. Splits are
session-level and **frozen after first run**; the held-out test split is not examined until day 11
of a 14-day plan. Throwaway clips used to exercise the ingest → label → split → harness plumbing go
into `SMOKE` sessions that are structurally excluded from splits and eval runs, so they cannot
contaminate a real number. `DECISIONS.md` records every idea that was cut *with the condition that
would reopen it*, and the README states the limits — one kitchen, one cook, one camera angle, no
real users, no evidence it changes how anyone learns.

`cookai` is the earlier weekend-scoped attempt at the same problem, kept for provenance.

### Automated trading pipeline

**Repository:** `trading`
**Status:** Research — smoke backtest green, no live capital
**Stack:** Python, NautilusTrader 1.231, Interactive Brokers as target broker

Research → backtest → paper → small live, for a small account. The repository is written primarily
as a defence against self-deception, and the code enforces what the research concluded.

No backtest may run without the real IBKR fee model, because gross P&L is fiction. The validation
ladder — in-sample, walk-forward, untouched holdout, four weeks of paper, then tiny live — has no
skippable rungs, justified in `RESEARCH.md` by the observed R² below 0.025 between backtest Sharpe
and live results. A cash account means long-only. The kill switch lives at the broker, not in code,
because a kill switch that depends on your code still running is not one.

The smoke backtest is **designed to lose money**: an EMA cross over synthetic random-walk data whose
expected output is a small net loss, most of it fees, demonstrating what thirty trades a day costs a
$5k account. I would rather the first thing a reader runs be the cost of trading than a curve.

### n8n workflow library

**Repository:** `n8n-workflows`
**Status:** In use — 21 automations
**Stack:** JavaScript, n8n via MCP

Twenty-one productised automations, each packaged with a README, intake script, runner,
status-event handling, ops test and exported workflow JSON: invoice extraction from Gmail and
Outlook, incoming-invoice AI, payment reminders, lead automation with follow-up, customer follow-up
with a reply monitor, CRM to Sheets, support ticket routing, WooCommerce admin, a WordPress security
monitor with IMAP intake and a daily digest, Meta Ads assistant, webshop chatbot, and more.

Authoring goes through an MCP connection that first asserts the server advertises the builder tools
it needs. Drafts stay inactive until publishing is explicitly approved — a workflow that starts
itself on creation is a workflow that emails your customers during a test.

### Hermes — agent operations with a human gate

**Repositories:** `hermes-vps-agent/hermes-agent` · `hermes-vps-agent/hermes-dashboard`
**Status:** Built — dashboard deployed, Tailscale-only
**Stack:** Next.js 14, PostgreSQL, Docker, OpenRouter, GitHub API, n8n

An autonomous coding agent on a VPS with approval in front of every merge and deploy, reviewed from
a phone. Each queue item shows task, project, colour-coded risk, branch and links to PR, staging,
screenshots and test report; approving merges the PR and records a deployment row.

The intake design is the interesting constraint. Describing work in plain language does not queue
anything — an LLM distinguishes chit-chat from build requests, asks *one* clarifying question when
vague, then renders a task preview that must be explicitly tapped. A raw sentence can never launch
the runner on the wrong thing. The dashboard has no authentication at all, which is a deliberate
choice paired with network-layer restriction rather than an omission.

The agent repository also carries delivered artefacts with evidence bundles — a WordPress FSE theme
generator with `theme.json` v3 schema validation, and a site builder whose QA scorer is calibrated
against a deliberately-bad control site.

### Question Collector

**Repository:** `lie2me-collector`
**Status:** Built
**Stack:** Python, OpenAI-compatible providers plus a keyless mock, Docker, n8n integration

The content factory behind the trivia modes of *Two Lies and a Truth*, built for repeatable drops of
60, 600 or 10,000+ questions. Nine validation checks — schema, exact and windowed near-duplicate
detection, option quality, difficulty and category fit, profanity — with CI-grade gates that fail
the build on rate ceilings such as `near_duplicate_prompt=0.02`.

Scaling is chunked: generate in batches, dedupe each against previous bundles with `--seed-bundle`,
merge with dedupe. Internet research mode builds questions from facts extracted from Wikipedia and
Wikidata **with source URLs attached**, rather than from model recall. Output includes a reviewer
CSV, because the human pass was assumed from the start rather than added when quality slipped.

### Smaller tools

**`amelia-szamlazzhu`** — a commercial WordPress plugin bridging Amelia Booking to Számlázz.hu
invoicing, with a custom field manager, a **retry queue** for failed invoice attempts, an invoice
log and a licence manager. The retry queue is the whole value: an invoice that silently fails to
issue is a compliance problem, not a UI problem.

**`resumemaker`** — generates job-targeted CV variants from one canonical JSON Resume, with a
path-based **field-lock config** so the AI may tailor a summary but cannot touch personal facts, and
a validator that fails if a locked field drifted.

**`nstudio`** — the agency site, and the app-compliance surface for everything else here: privacy
policies, terms, refund policy and GDPR deletion pages and endpoints for Lumino, Lie to Me and
SoforPlus, which the app stores require and which have to exist somewhere.

---

## 4. Archived

Listed for completeness rather than as evidence.

**`block-puzzle`** — the Unity/C# predecessor to Lumino, with AdMob, Play Games Services, RevenueCat
and editor tooling for batch iOS/Android builds. Superseded by the Flutter rewrite.

**`hostjobs`** and **`jp-hospitality-import-jobs`** — the WordPress implementation of the Japanese
job board and its PHP feed importer, both superseded by `jp-jobs`.

**`crypto-telegram-bot`** — a Telegram bot with 40+ commands: charts with technical indicators,
portfolio P/L, price and DEX alerts, multi-chain wallet tracking, and a subscription tier system
billed either through Stripe or on-chain USDT/USDC on Polygon with transaction verification.

**`tripora`** — a React Native trip-plan marketplace with Stripe subscriptions via Supabase edge
functions, offline caching and seven languages. Complete, unlaunched.

**`travelplanning`** — an early multi-agent trip planner. Superseded, and useful now mainly as a
marker of how much the agent tooling changed in a year.

**`cookai`** and **`nagytamasgit/ingatlanspanyol`** — earlier snapshots of projects above. Two further
duplicates exist and are discussed under *What I would do differently* rather than listed here.

---

## Full index

| Repository | What it is | Stack | State |
|---|---|---|---|
| `toms-blocks` | Lumino Block Puzzle — published game with live-ops economy | Flutter, Firebase, RevenueCat | In use |
| `grundi-opencart` | Live grundi.hu marketplace: settlement engine, invoicing, lifecycle email | PHP, OpenCart 3, Webkul | In use |
| `getcourse` | Anfisa Beauty — GetCourse replacement (LMS, memberships, booking) | WordPress, LearnDash, Woo | In use |
| `app.oldal.ai` | AI website builder, multi-tenant, credit-billed — case study 0001 | Next.js 15, Drizzle, Claude | In use |
| `oldal.ai` | Marketing site for the above | Next.js 16, MDX | In use |
| `xprize` | AI "business employee" — booking, invoicing, discoverability | Next.js, PG event log, Gemini | In use |
| `IngatlanSpanyol/ingatlanspanyol` | Costa del Sol property search (EN/ES/HU) | Next.js 16, Drizzle | In use |
| `jp-jobs` | Japanese hospitality job board with visa + shokumu keirekisho flows | Next.js 16, Prisma, NextAuth | In use |
| `n8n-workflows` | 21 productised automations | JS, n8n MCP | In use |
| `nstudio` | Agency site + app compliance surface | Next.js 16, next-intl | In use |
| `cossy` | Cosplay social network with creator economy | Flutter, Supabase, RevenueCat | Shipped |
| `lumino-rush` | Arcade spin-off with a rhythm mode | Flutter, RevenueCat | Shipped |
| `sentinel` | WordPress security platform + zero-dependency agent plugin | Next.js 15, Drizzle, PHP | Built |
| `soforapp` | Ride-hailing with partner bidding | NestJS, Prisma, Flutter ×2 | Built |
| `grundi-` | Greenfield marketplace rebuild | Next.js, NestJS, Drizzle | Built |
| `eutory` | European brand directory, deals and comparisons | Next.js 16, Prisma | Built |
| `eutory-collector` | AI research → contract-validated content drafts | Python, Typer, LLM | Built |
| `liesandtruth` | Social deduction game, server-authoritative rules | Flutter, Firebase Functions | Built |
| `aiwp2` | NexusPress — AI WordPress block SaaS | Next.js 14, Prisma, Gemini | Built |
| `aiwp-plugin` | NexusPress WordPress plugin v1.0.0 | PHP, JWT | Built |
| `webautomatizacio` | Productised automation shop, invoice-only by design | Next.js, Medusa | Built |
| `euroszaki` | Skilled-trades job board with feed import and marketing automation | Laravel, Filament | Built |
| `lie2me-collector` | Question pipeline with CI-grade quality gates | Python, Docker | Built |
| `hermes-vps-agent/hermes-agent` | Agent task protocol + delivered artefacts | JS, WP theming, n8n | Built |
| `hermes-vps-agent/hermes-dashboard` | Phone-first approval dashboard | Next.js 14, PG, OpenRouter | Built |
| `amelia-szamlazzhu` | Amelia ↔ Számlázz.hu invoicing bridge with retry queue | PHP, WordPress | Built |
| `resumemaker` | CV variant generator with field locks | Node.js | Built |
| `cook-solutions` | `sear` — single-moment cooking coach — case study 0002 | Python, numpy DSP | Research |
| `country-guide` | Travel etiquette app with a machine-enforced editorial gate | Flutter, Node/TS, JSON Schema | Research |
| `trading` | Quant pipeline; fees mandatory, validation ladder | Python, NautilusTrader | Research |
| `block-puzzle` | Unity predecessor of Lumino | Unity, C#, Firebase | Archived |
| `hostjobs` | WordPress predecessor of the JP job board | WordPress, Blocksy, Fluent | Archived |
| `jp-hospitality-import-jobs` | PHP job-feed importer for the above | PHP, SQLite | Archived |
| `crypto-telegram-bot` | Crypto bot, Stripe + on-chain subscriptions | Python, Web3 | Archived |
| `tripora` | Trip-plan marketplace app, unlaunched | React Native, Supabase | Archived |
| `travelplanning` | Early multi-agent trip planner | Python, Anthropic | Archived |
| `cookai` | Earlier attempt at `sear` | Python | Archived |
| `nagytamasgit/ingatlanspanyol` | Earlier snapshot of the property platform | Next.js 16 | Archived |
| `soforplus` · `hermes-vps-agent/hermes` | Reserved names, no code | — | Empty |

---

## What I would do differently

The case studies each end this way and it would be dishonest to write an index of forty
repositories without it. These are the criticisms I would make of this body of work if someone
handed it to me.

**Too much of it is Built rather than In use.** Ten projects have real users; fifteen are complete,
tested, and have never been used by anyone outside. Passing your own tests proves the code does what you thought;
it proves nothing about whether the thing was worth building. The ratio is the single most honest
criticism available here, and the fix is not more projects — it is finishing fewer of them
*through* to users.

**I cannot give you outcome numbers for most of it.** I can tell you about coverage, architecture
and deadlines met. I largely cannot tell you retention, conversion or revenue, because for most of
these I did not instrument the thing that would have told me. In the case studies I had JLPT
certificates, a rejection rate, and an 82% figure for calls surviving to a real conversation — the
projects in this list mostly lack an equivalent, and where I have not published a number here it is
because I could not source one I would defend, not because it was unflattering.

**Four duplicates and two abandoned snapshots.** `jobscore` is byte-identical to `euroszaki`.
`aiwp` and `aiwp2` are the same product, and there were two copies of the property platform at
different versions. That is what shipping quickly across many projects costs when repository
hygiene is nobody's priority, and it makes the list look larger than the work is.

**Secrets in six repositories.** An audit across everything found live credentials committed and
still in history: two Android signing keystores, an Anthropic key, a database URL, WordPress salts
and several API keys. All in private repositories, so none of it leaked — but "the repository was
private" is a mitigation, not a practice. The right habit is a pre-commit secret scanner from the
first commit, which I now run and did not then. One of these was purged properly, with the history
rewritten, when the repository was published.

**Several READMEs are still `create-next-app` boilerplate.** For a body of work whose main artefact
is the repository, that is a real gap, and it is visible in the ones I could not describe well here
without reading the source.

**Breadth was partly a choice and partly a habit.** Forty repositories in eleven months means some
of them are one good week each. The projects I would actually point at — the settlement engine,
Sentinel's wire protocol, `sear`'s evaluation discipline, Country Guide's editorial gate — are the
ones where I stayed long enough to get the hard part right. The rest is competent, and competent is
not the interesting part of anyone's work.

**What I would keep.** The habit of writing decisions down, including what was cut and what would
reopen it. Building the harness before the feature — the restore drill, the frozen splits, the
verification suites, the fee model that cannot be skipped. And enforcing correctness in a machine
rather than in my own discipline, because my discipline is the thing that fails at 2am under
deadline and a validator is not.

---

## Access

These repositories are private because they contain client and commercial work. I'm glad to arrange
a walkthrough, a read-only collaborator invite, or a sanitised code sample for anything here —
client-owned code only with the client's consent, which I'll ask for before sharing.

**[nagytmas@gmail.com](mailto:nagytmas@gmail.com)** — tell me which project and what you want to
see. If you'd rather read than meet, start with [case study 01](01-video-assessment.md); it is the
best single thing I have written about my own work.

*Compiled 2026-08-22 from repository contents rather than memory.*
