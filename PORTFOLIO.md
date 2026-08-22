# Engineering Portfolio — Tamás Nagy

**40 private repositories · 38 with shipped code · full-stack product engineering**

**Scope: this is roughly the last year of work.** Every repository listed here was created between
**October 2025 and August 2026** — an 11-month window. It is not a complete career history; it is
what I have built recently, and mostly solo. Earlier work is not included.

I build and ship complete products end to end: product definition, architecture, backend, frontend,
mobile, infrastructure, deployment and store release. Most of the work below is **solo-built** and
production-deployed, spanning multi-tenant SaaS, marketplaces, AI platforms, mobile apps and
published games.

The repositories are private because they contain client work and commercial products.
**I'm happy to walk any company through a live demo or a read-only code review of any project here.**

> **Request a demo or code access** — [nagytmas@gmail.com](mailto:nagytmas@gmail.com)
> Tell me which project interests you and I'll arrange a screen-share walkthrough, a read-only
> collaborator invite, or a sanitised code sample. Client-owned repositories require the client's
> consent, which I'll obtain before sharing.

---

## At a glance

| | |
|---|---|
| **Repositories** | 40 private (38 with code, 2 empty placeholders) |
| **Languages** | TypeScript (13), Python (7), Dart/Flutter (5), PHP (5), JavaScript (3), HTML (2), Shell, Blade, C# |
| **Deployed to production** | 10+ (Hetzner, Coolify, GCP Cloud Run, Vercel, Render, Google Play, self-managed VPS) |
| **Published to app stores** | 2 (Google Play production + internal testing tracks) |
| **Largest single codebase** | 389 files, 20 DB migrations (Oldal AI website builder) |

### Capability matrix

| Area | Evidence in this portfolio |
|---|---|
| **Multi-tenant SaaS** | Oldal (agency→client→project RBAC), Sentinel (org scoping), XPRIZE (workspace RLS), NexusPress |
| **AI / LLM engineering** | Anthropic Claude (streaming, vision, prompt caching, tool calls), Google Gemini, OpenAI, OpenRouter, local Ollama, output hardening, LLM-as-judge, cost metering & markup billing |
| **Applied ML (non-LLM)** | `sear` — audio DSP cascade (log-mel STFT, onset detection, spectral features) with a frozen train/test split and an honest eval harness |
| **Mobile** | Flutter (5 apps, iOS+Android, published), React Native/Expo, Unity/C# |
| **Backend** | Next.js App Router, NestJS + Fastify, Hono, Laravel, Medusa, Firebase Cloud Functions, Supabase Edge Functions |
| **Data** | PostgreSQL (Drizzle, Prisma), MySQL, SQLite, Firestore, Supabase, OpenSearch, Redis |
| **Realtime** | Socket.IO rooms, Supabase Realtime, Firestore listeners |
| **Payments & billing** | Stripe, RevenueCat, Google Play Billing 8, Számlázz.hu, Billingo, prepaid credit metering, on-chain USDT/USDC |
| **Security** | HMAC request signing, AES-256-GCM at rest, scrypt/bcrypt, TOTP (RFC 6238), JWT, RLS tenant isolation, CSP injection, static JS scanners, rate limiting, append-only audit logs |
| **Infrastructure** | Docker Compose, Caddy auto-TLS, Coolify, GCP Cloud Run + Cloud SQL, systemd, pg-boss/BullMQ workers, cron |
| **E-commerce operations** | OpenCart 3 + Webkul Marketplace (seller settlement engine, commission netting, COD vs card reconciliation, invoice provider abstraction), Medusa, WooCommerce + Subscriptions |
| **WordPress engineering** | Plugin development (agent, invoicing bridge, block injector), FSE theme generation, LearnDash LMS builds, containerised WP staging with backup/restore drills |
| **Quality** | Playwright E2E, Vitest/Jest, Flutter widget+integration tests, custom verification harnesses, CI on GitHub Actions |
| **Release engineering** | Codemagic, Fastlane-style Play publishing, staged rollouts, store listing automation |

---

## Table of contents

1. [AI platforms & SaaS](#1-ai-platforms--saas)
2. [Marketplaces & multi-sided platforms](#2-marketplaces--multi-sided-platforms)
3. [Mobile apps & games](#3-mobile-apps--games)
4. [Automation, pipelines & developer tooling](#4-automation-pipelines--developer-tooling)
5. [Websites & client work](#5-websites--client-work)
6. [Earlier work & experiments](#6-earlier-work--experiments)
7. [Full repository index](#7-full-repository-index)

---

## 1. AI platforms & SaaS

### Oldal — AI website builder (multi-tenant SaaS)
`app.oldal.ai` · TypeScript · pnpm monorepo · **the largest and most complete project here**

**What it is.** A SaaS where agencies and their clients build production websites by prompting AI.
Output is flat-file, host-anywhere HTML/CSS — deliberately *not* a CMS, so there is no WordPress
attack surface to maintain.

**The problem it solves.** Agencies spend the bulk of a small-site budget on build labour, then
inherit a WordPress installation they must patch forever. Oldal collapses the build into a prompt
and ships static output the client can host anywhere.

**Stack.** Next.js 15 (App Router) · TypeScript · PostgreSQL + Drizzle (20 migrations) ·
Anthropic Claude SDK (streaming, vision, prompt caching) · Hono form relay · Resend ·
Tailwind 4 · Docker Compose + Caddy auto-TLS on a single VPS.

**Features**

- **Two generation modes sharing one set of rails.** *Agentic*: describe the site (optionally paste
  a reference image) and Claude generates a bespoke multi-page site, refined by chat, exported as
  a ZIP. *Template*: a structured `SiteModel` edited through validated AI tool calls and rendered
  by a deterministic renderer — the AI never writes raw code, so preview == export by construction.
- **Security hardening pipeline.** A capability policy (HTML+CSS / +JS / +PHP) is enforced *after*
  generation: JS off means no `<script>` tags or inline handlers survive; generated JS is run
  through a static scanner; a strict CSP is injected into every page. There is a dedicated
  `verify-harden-bypass` suite that attacks its own hardener.
- **Multi-tenant RBAC** — agency → client organisations → projects, with delegation, invites,
  password reset, scrypt password hashing, session security and login rate limiting.
- **Prepaid credit billing** — 1 credit = $0.01. Generation charges the *real* Claude API cost ×
  a configurable markup; a finished site charges a one-time site fee on first export; generation
  is gated on balance. Real-USD usage metering per client plus invoice totals in the billing UI.
- **Background generation jobs** with a live progress UI and a generation watcher.
- **Figma import** (per-org OAuth) and **clone-from-URL** — extract a design system from an existing
  site or a Figma frame and rebuild it as a hardened static site.
- **Hosted form relay** — exported static sites post to one hardened endpoint with spam filtering,
  transactional email and a lead inbox back in the dashboard.
- **WordPress export path** — render a `SiteModel` as a full-site-editing WP theme, plus a
  WP orchestrator service for AI-operated managed hosting.
- **Version history, media manager, design-token controls, block picker, canvas editor,
  changelog feed, newsletter with double opt-in, admin console, public REST API (`/api/v1/sites`).**
- **~200 assertions across 15 verification suites** (PGlite + mock LLM), so the whole platform can
  be validated without touching a paid API.

**Status.** Feature-complete for closed beta, fully verified, deployed to production on Hetzner.

---

### XPRIZE — "AI business employee"
`xprize` · TypeScript · npm workspaces · **live in production**

**What it is.** An AI employee a small business hires for *outcomes* rather than for software
features: **Booked** (fills the calendar), **Paid** (issues and chases invoices), **Found** (gets
the business into AI answers and search).

**The problem it solves.** Small service businesses don't want another dashboard to operate. They
want the outcome. This system takes actions on their behalf, with every action gated by a human
approval step and recorded in an immutable evidence trail.

**Stack.** Next.js App Router · TypeScript · PostgreSQL (append-only event log + RLS) ·
pg-boss worker · Google Gemini · Firebase Auth · GCP Cloud Run + Cloud SQL (europe-west4),
later migrated to Coolify · Resend.

**Features**

- **Append-only event-log core.** Events are the source of truth; projectors materialise
  bookings/leads/contacts read models. The `events` table rejects UPDATE and DELETE *even for the
  service role*. Projections are idempotent on replay.
- **Row-level-security tenant isolation** — a workspace can never read *or write* another
  workspace's rows, proven by test.
- **Approval queue with progressive autonomy** — every agent action runs propose → approve/reject →
  execute, with per-workspace autonomy policies and idempotent execution.
- **Loop A (Booked)** — a Gemini chat agent that reads owner constraints, time off and slot holds,
  then books.
- **Loop B (Paid)** — completed work → deterministic invoice → gated approval → connector issues it
  (Számlázz.hu / Stripe / mock).
- **Loop C (Found)** — grounded HU/EN content generation → LLM judge → gated publish → public site
  with hreflang and FAQ/LocalBusiness structured data.
- **Owner Assistant** — configure services, prices, packages, schedule, holidays and booking mode
  purely by chat, all routed through Approvals.
- **Agent-addressable entity API** (`/api/entity/<workspace>`) — live JSON of services, prices,
  hours and real availability, designed to be consumed by *other* AI agents.
- **Onboarding by archaeology** — point it at an existing website URL and it reconstructs a live
  configured workspace.
- **Owner email channel** with approval magic links and a daily brief; custom domain support;
  embeddable chat widget; evidence export endpoint.

**Status.** Live in production. All three loops working; inbox, Google Business Profile ingestion
and the Stripe connector are the next increments.

---

### Sentinel — WordPress security & operations platform
`sentinel` · TypeScript · pnpm monorepo + PHP plugin

**What it is.** A security and uptime platform for agencies that maintain fleets of WordPress
sites — a dashboard plus a zero-dependency WordPress agent plugin.

**The problem it solves.** Maintenance agencies manage dozens of client sites with no unified view
of what changed, what's vulnerable or what's down. Sentinel gives one console over the fleet with a
cryptographically authenticated channel to each site.

**Stack.** Next.js 15 (App Router) · pg-boss worker · PostgreSQL 16 + Drizzle ·
Zod · WordPress plugin in PHP 7.4+ with **zero dependencies** · Docker Compose / Coolify.

**Features**

- **Hardened wire protocol.** `HMAC-SHA256(secret, "{timestamp}.{nonce}.{raw_body}")` on every
  request, ±300 s timestamp tolerance, single-use per-site nonces with worker-side pruning.
- **Secrets at rest** — the per-site HMAC secret is stored AES-256-GCM encrypted server-side and
  derived from `AUTH_KEY` on the WordPress side.
- **One-shot pairing flow** — dashboard issues a single-use `pt_…` token; the pair response returns
  the `site_key` + `hmac_secret` exactly once.
- **Auth** — organisation signup, scrypt + database sessions, **TOTP two-factor (RFC 6238,
  hand-implemented and unit-tested)**, org-level scoping throughout.
- **Plugin agent** — pairing, heartbeat, event collection, command executor, file-integrity module,
  secrets handling, admin UI.
- **Command channel** — signed commands with a pending/result lifecycle.
- **Credit accounting** and an audit log; E2E smoke test against a running server + database.

**Status.** MVP complete and green — monorepo, data model, pairing, HMAC, heartbeat, event
reporting, auth/TOTP, site detail views, E2E smoke. Uptime/SSL monitoring, Wordfence feed import
and notification channels are the mapped next steps.

---

### Hermes — autonomous agent operations
`hermes-agent` + `hermes-dashboard` (org: `hermes-vps-agent`) · JavaScript / TypeScript

**What it is.** A two-repository system for running an autonomous coding/ops agent on a VPS with a
**human approval gate** in front of every merge and deploy.

**The problem it solves.** Letting an agent act on real infrastructure is only safe if a human can
see what it proposes, in context, and approve or reject it in one tap — from a phone.

**Stack.** Next.js 14 · PostgreSQL · Docker · Tailscale-only network access ·
OpenRouter LLM for intake parsing · GitHub API · n8n.

**Features**

- **Review queue** — every pending item shows task, project, colour-coded risk level, branch and
  links to PR / staging / screenshots / test report, plus the model audit. Approve merges the PR,
  labels it and records a deployment row; Reject closes it unmerged; Request changes posts a PR
  comment.
- **Conversational intake** — describe work in plain language; an LLM acting as an intake assistant
  distinguishes chit-chat from build requests, asks *one* clarifying question when vague, then shows
  a **task preview** that must be explicitly tapped to queue. A raw sentence can never launch the
  runner on the wrong thing.
- **Multi-chat sidebar** with rename/delete, per-chat persisted transcripts keyed by session.
- **Task list + history** with status/priority filters; new tasks open a templated GitHub issue.
- **Live health dots** for the WordPress sandbox and n8n, checked server-side.
- **Agent side (`hermes-agent`)** — a task protocol driven by GitHub issues/labels, plus real
  delivered work products: a WordPress FSE theme generator with a JSON design-spec translator and
  `theme.json` v3 schema validation, an AI site builder with mechanical/visual/rubric QA scoring
  calibrated against a deliberately-bad control site, and an n8n workflow factory that ships
  evidence bundles (workflow sent → created → test results → execution records → LLM cost) for
  every automation.

**Status.** Working system with deployed dashboard and an evidence trail of completed agent runs.

---

### NexusPress — AI WordPress block generation SaaS
`aiwp2` · `aiwp-plugin` (companion plugin)

**What it is.** A SaaS that generates brand-consistent WordPress Gutenberg blocks from a text
prompt and injects them into a connected site in one click.

**Stack.** Next.js 14 · TypeScript · Prisma + MySQL · Google Gemini · Stripe · JWT ·
WordPress plugin (PHP 7.4+, GPL-2.0).

**Features**

- **Brand DNA** — per-account colours, fonts, voice tone and industry, applied to every generation
  so output stays on-brand.
- **Block types** — hero, features grid, CTA, pricing table and more.
- **One-click injection** — the plugin exposes `POST /wp-json/ai-design/v1/inject`, authenticated
  by JWT bearer token, and creates the page/post as a draft, returning edit and preview URLs.
- **Multi-site support** — connect and manage several WordPress installations, with per-site secret
  keys, regeneration and connection ping/testing.
- **Billing** — pay-as-you-go credits or subscriptions via Stripe checkout, customer portal and
  webhook handling (`checkout.session.completed`, subscription updated/deleted, invoice
  succeeded/failed).
- **Auth** — register/login/me, password reset request + confirm, JWT sessions.
- **Generation history**, injection logs, onboarding flow, admin dashboard.
- **Tested** — unit suites for JWT, password hashing, input sanitisation and validation schemas,
  with coverage reporting.

**Status.** Feature-complete build with a published companion plugin (v1.0.0) and a deployment
pipeline for Hostinger.

---

### sear — single-moment cooking coach (applied ML research)
`cook-solutions` · Python · MIT licensed

**What it is.** A deliberately narrow system that detects **one** cooking mistake — the pan wasn't
hot enough when the food went in — within seconds, and coaches the correction while it can still
be fixed.

**Why it's interesting engineering.** It is a study in scoping and in honest evaluation. The core
insight is that a cold pan is an **audio** problem, not a vision problem: a proper sear produces
immediate broadband noise with a sharp onset, an under-temperature pan a weaker hiss that ramps.
Audio is orders of magnitude cheaper than video, works when the cook's hand occludes the pan, and
doesn't care about steam on the lens.

**Stack.** Python 3.11 · numpy (hand-rolled STFT — no librosa) · OpenCV · `uv` · pytest · ruff.

**Features**

- **Three-tier cascade** where expensive components run rarely: Tier 1 always-on audio onset
  detection (log-mel over a 1 s window) plus 2 fps frame differencing on the pan ROI; Tier 2
  triggered ~10 ms classifier (`good_sear` / `pan_too_cold` / `ambiguous`) using band-energy ratio,
  spectral flatness and attack slope; Tier 3 rare single LLM call producing one coaching utterance
  under 20 words.
- **Evaluation harness** that scores any classifier, with session-level 80/20 splits **frozen after
  first run** and a held-out test split not examined until day 11 of the build.
- **`SMOKE` session isolation** — throwaway clips used to test the ingest→label→split→harness
  plumbing are structurally excluded from splits and eval runs so they can never contaminate real
  numbers.
- **A written `DECISIONS.md`** recording every idea that was *cut*, each with the condition that
  would reopen it, and an `EVAL.md` with targets and an ablation against a naive
  video-language-model baseline.
- **White-balance drift experiment**, labelling tool, split generator, synthetic-signal test suite.

**Status.** Days 1–7 code-complete and fully tested on synthetic signals, with the Tier 3 coach
(provider, contract, cost accounting, event log) added since; blocked on kitchen recording time,
not on code. Honest limits are documented in the README rather than hidden. `cookai` is the earlier
weekend-scoped attempt at the same problem, kept for provenance.

---

## 2. Marketplaces & multi-sided platforms

### Grundi — greenfield marketplace
`grundi-` · TypeScript · pnpm monorepo

**What it is.** A full multi-vendor marketplace rebuilt from scratch: storefront, buyer account,
seller portal and admin, on a typed contract-first backend.

**Stack.** Next.js App Router (storefront/admin) · **NestJS + Fastify** REST API with OpenAPI docs ·
BullMQ workers · Drizzle + PostgreSQL · Zod shared contracts · OpenSearch · MinIO · Redis · Mailpit.

**Features**

- **Six-package monorepo** — `apps/web`, `apps/api`, `apps/worker`, `packages/contracts` (shared Zod
  contracts + event names), `packages/db`, `packages/ui`.
- **Dual data store** — an in-memory store for fast local iteration and tests, and PostgreSQL as the
  canonical production store; both share order numbering, moderation gating, payment/settlement
  rules, session handling and SKU/slug constraints, with Postgres authoritative where they diverge.
- **Seller lifecycle** — seller registration, product creation, moderation gating, settlement
  generation.
- **Buyer lifecycle** — catalogue search (OpenSearch-backed), cart, checkout, order pages, account.
- **Workers** for imports, feeds, search indexing, invoices and shipping labels.
- **RBAC + rate-limit guards + token service + Zod validation pipe** across the API surface.
- **Tested** — auth core, checkout ordering, import validation, product workflow, settlement
  generation, catalogue search and contract tests.

**Status.** Web wired to the API; backlog and status tracked in-repo.

---

### Ingatlanspanyol — Costa del Sol property search
`IngatlanSpanyol/ingatlanspanyol` (canonical, v1.6.4) · TypeScript

**What it is.** A trilingual (EN/ES/HU) property search platform for the Spanish Costa del Sol
market, syncing listings from an upstream Supabase feed into its own PostgreSQL.

**Stack.** Next.js 16 App Router · next-intl · Drizzle + PostgreSQL · Supabase client (source feed) ·
Google Maps JS API · Playwright + Vitest · Docker multi-stage + Coolify · Plausible.

**Features**

- **Localised routing** for three languages with locale-aware sitemaps, robots, manifest, hreflang
  alternates and legal pages (`aviso-legal`, privacy, terms).
- **Property search** — filter form with location autocomplete, type dropdown, beds/baths dropdown,
  results toolbar, pagination, skeleton loading states and reveal-on-scroll.
- **Property detail** — image gallery, map, sticky header, share buttons, scroll-to-contact and a
  contact form.
- **Ingestion pipeline** — an admin-secret-protected `POST /api/admin/sync` plus a `sync_runs` audit
  table and `GET /api/admin/sync-runs` history; scheduled post-deploy sync runner that calls the app
  on localhost to avoid the reverse-proxy hairpin problem.
- **Admin auth hardening** — shared-secret header, optional IP allowlist, explicit
  `TRUST_PROXY_HEADERS` opt-in.
- **A companion WordPress plugin** so the search can be embedded into an existing WP site.
- **Tested** — Vitest units for admin auth, property queries and the sync runner; Playwright smoke.

**Status.** Deployed via Coolify with scheduled sync. *(`nagytamasgit/ingatlanspanyol` is an earlier
snapshot of the same codebase at v0.4.7.)*

---

### Eutory — European brand directory & comparison platform
`eutory` (platform) + `eutory-collector` (content pipeline)

**What it is.** A directory, deals and comparison site for European brands and services, with a
company self-service portal and a separate AI research pipeline that drafts its content.

**Stack (platform).** Next.js 16 · Prisma 7 + PostgreSQL · NextAuth · Playwright E2E ·
Docker multi-stage with healthcheck · nodemailer.
**Stack (collector).** Python 3.11 · Typer CLI · trafilatura + DuckDuckGo search · OpenAI-compatible
LLM client (incl. local Ollama) · Docker / Coolify.

**Platform features**

- **Content model** — brands, categories, countries, deals, posts, comparisons and comparison
  features, driven by a versioned content contract (v2) with localised fields.
- **Public surface** — directory (by category / country / tag / type), deals with redirect tracking,
  blog, comparison pages including head-to-head `compare/vs/[brand]`, calculators workbench,
  "why European" and verification pages.
- **Company dashboard** — self-service brand editing, deal management, profile and password.
- **Brand submission wizard** for unclaimed brands, with a draft-approval queue.
- **Admin CMS** — generic resource editor over every content type, analytics (traffic, SEO,
  incidents overview), API-key manager with scopes, calculator studio.
- **Public REST API** (`/api/v1/*`) with scoped API keys, bulk import and media import.
- **SEO** — JSON-LD, dynamic OG image route, alternates/hreflang, sitemap, robots, 404 tracking.
- **Analytics & monitoring** — first-party page-view and not-found tracking, uptime monitor tick
  with alerting, scheduled SEO audit script.
- **Security** — email verification with hashed tokens, rate limiting, request-security and
  env-security modules, audit log, dedicated DB-role SQL, a documented hardening pass.
- **Playwright E2E** across auth, brand flows, listings, navigation and API smoke.

**Collector features**

- Gathers research for a brand/category/country/deal/post/comparison, biased toward official sites,
  public docs, pricing and legal pages, Wikipedia and EU public pages.
- Synthesises a draft content JSON document against the canonical contract, then **validates** it —
  required fields, enums, ISO dates, URL shapes, locale codes, canonical category slugs, GEO objects
  and unresolved placeholders.
- **Never invents database IDs**: foreign keys stay placeholders until resolved from a local mapping
  file, with a separate `resolve-ids` post-processing step.
- Emits a full artefact bundle per run (`research.json`, `job.json`, `generated.json`,
  `validation.json`, `source-notes.json`).
- Ships n8n workflow definitions for discover / enrich / sync-pull / sync-push, plus an HTTP
  smoke-test mode with optional API-key hardening.

---

### SoforApp — ride-hailing with a partner-bidding model
`soforapp` · NestJS + Flutter × 2

**What it is.** A three-part ride-hailing MVP where passengers request a ride and **transport
companies bid** for it, rather than a single algorithmic dispatch.

**Stack.** NestJS + Prisma + PostgreSQL · Socket.IO · Redis · Flutter passenger app ·
Flutter partner/driver app · Docker Compose · Codemagic CI.

**Features**

- **Full ride lifecycle** — request → broadcast to partners → bids → passenger accepts → partner
  assigns driver → en-route → in progress → completed → rating, with state-machine validation at
  each transition.
- **Bidding engine** — one bid per company per ride, rejection of bids on closed rides, and a
  wallet guardrail requiring a company to hold at least `bid + 2000` before bidding.
- **Wallet system** — balance, deposits, commission withdrawal, full transaction history.
- **Realtime** — room-based Socket.IO events per company (`company_<id>`) and per ride
  (`ride_<id>`): new request, new bid, bid accepted, driver assigned, ride started/completed and
  live driver location.
- **Team & fleet management** — teams, driver activation flow with OTP, RBAC, driver documents,
  fleet map, dispatch messaging, ride conversations, company ride history and stats.
- **Payments & invoicing** — Stripe, company payment methods, Billingo invoicing provider with a
  retry service, reconciliation service and a country-specific billing policy module.
- **Passenger app** — destination selection, weather preview for the destination (Open-Meteo), live
  bid list, ride tracking with driver location, completion rating.
- **Tested** — unit specs across auth, wallet, invoicing, payments and the events gateway, plus an
  end-to-end simulation spec.

**Status.** End-to-end flow working locally with seeded test accounts; deployment, Adyen and release
prep documented in-repo.

---

### Japan hospitality job board
`jp-jobs` (v3.6.49, the live platform) · `jp-hospitality-import-jobs` (PHP importer) ·
`hostjobs` (WordPress predecessor)

**What it is.** A bilingual job marketplace connecting foreign candidates with Japanese hospitality
employers, including the visa and Japanese-résumé workflows that make that market specific.

**Stack.** Next.js 16 · TypeScript · Prisma + PostgreSQL (12 migrations) · NextAuth v5 ·
next-intl · Anthropic + OpenAI SDKs · Stripe · Tiptap editor · Leaflet maps · Playwright E2E ·
Vitest · Docker · gitleaks in CI.

**Features**

- **Three role surfaces** — candidate, employer and admin, each with its own layout and dashboard.
- **Candidate** — profile, applications, saved jobs, documents (with private-storage migration),
  notifications, visa details, discoverability controls, linguistic profile and talent-intent.
- **Shokumu keirekisho** — generation, translation and PDF export of the Japanese-format career
  history document, with usage metering.
- **Employer** — onboarding, job posting with validation, applicants table, application detail and
  status actions, candidate discovery marketplace, Stripe checkout with success/cancel flows,
  notifications, company profile and branding (brand assets, slogans, follow model).
- **Admin** — job moderation and approval, payment confirmation, job sync and self-import, user and
  role management, visa cases with assignment, lawyer role, blog posts with translations,
  subscribers and waiting-list export, audit logs.
- **Job search** — full search, map view (Leaflet), company directory, expired-jobs handling and an
  indexed jobs schema tuned for query performance.
- **Content** — multilingual blog with per-locale translations, industry insights, FAQ (candidate and
  employer), guides.
- **Ops** — security rate-limit event tracking, performance budget checker, translation script,
  production migration runner, cron setup, custom `server.js`.
- **Tested** — Playwright specs for job search, application flow, candidate dashboard, employer
  experience and company directory.

**Related repos.** `jp-hospitality-import-jobs` is the standalone PHP job-feed importer (fetch,
enrich, bulk import, logo repair, WordPress integration) that seeded the board.
`hostjobs` is the earlier full WordPress implementation (Blocksy + FluentCRM/Forms/Cart/Boards,
ACF Pro, iThemes Security Pro) with a custom `cron_import.php` feed importer.

---

### EuroSzaki — skilled-trades job board for Western Europe
`euroszaki` · PHP / Laravel + Blade

**What it is.** A job board connecting skilled tradespeople with employers in Germany, Austria and
the Netherlands.

**Stack.** Laravel · Blade · Filament admin · Laravel Cashier (subscriptions) · MySQL.

**Features**

- **Listings** with tags, filters, pricing tiers, upsells, checkout fields and payment fields.
- **Candidate profiles** and applications; company profiles; conversations and messages.
- **Automated job import** — Apify/Indeed service, generic import service, auto-import and
  expiration-check console commands, import statistics dashboard, a documented skilled-trades import
  guide.
- **AI translation service** plus a general translation service and a translation-migration command,
  for a fully localised interface.
- **Résumé parsing service** for uploaded CVs.
- **Marketing automation** — newsletter subscribers, email campaigns (Filament-managed), weekly job
  digest command, and a Facebook Page posting command.
- **Click tracking** with tracking fields, for attribution on outbound listing clicks.
- **Social auth** alongside standard Laravel auth (login, registration, password reset, confirm).
- **Filament admin resources** for listings, posts, users, campaigns and subscribers.

---

### Webautomatizáció — productised automation agency shop
`webautomatizacio` · TypeScript · pnpm monorepo

**What it is.** A Hungarian automation agency storefront that sells **productised n8n workflows** as
catalogue items, with a deliberately manual invoice flow instead of card payments.

**Stack.** Next.js App Router (Hungarian storefront) · **Medusa** commerce backend with a custom
module · PostgreSQL · Puppeteer (offer PDFs) · Docker Compose + Caddy + systemd · Coolify.

**Features**

- **Custom Medusa module** (`automation-request`) with models for automation requests, customers,
  descriptions, offers and knowledge articles, plus seven migrations.
- **Quote-to-onboarding pipeline** — visitor submits an audit or order request → admin reviews →
  offer is generated, previewed, rendered to PDF and sent → approved → invoice marked → onboarding
  form issued to the customer at a tokenised URL.
- **Admin surfaces inside Medusa** for requests (with read-status and status transitions), customers,
  automation descriptions and knowledge articles, plus a product widget for public sections.
- **SEO landing pages** per automation category — n8n automation, CRM, lead, invoicing, reporting,
  webshop, AI chatbot for webshops — with a knowledge base, ROI calculator and pricing pages.
- **Deliberate payment stance** — "payment by invoice", no card form and no Stripe in the MVP;
  documented as a security decision, with honeypot + server-side validation + rate limiting on all
  public forms.
- **Legal pages** (ÁSZF, privacy, cookie policy, impressum) and cookie consent.

---

### Grundi (OpenCart) — the live marketplace, rebuilt and hardened
`grundi-opencart` · PHP · OpenCart 3.0.3.8 + Webkul Marketplace + Journal3 · **live at grundi.hu**

**What it is.** The production grundi.hu store: a working copy of the real multi-vendor OpenCart
installation, brought under version control, security-reviewed, containerised for local
development and extended across ten delivery phases. (Distinct from `grundi-`, the greenfield
Next.js/NestJS rebuild of the same business.)

**The problem it solves.** An inherited production marketplace with no version control, no local
environment and no seller-payout logic. The work turned it into a reproducible, auditable system a
team can safely change.

**Stack.** PHP 7.4 · MariaDB · OpenCart 3.0.3.8 · Webkul Marketplace · Journal3 theme ·
rootless Podman (PHP+Apache+MariaDB) · Caddy · Docker Compose deploy with runbook, backup script,
healthcheck and cron.

**Features delivered**

- **Phase 0 — reproducibility and security.** Repo brought under git with secrets and runtime junk
  gitignored; a full security pass with every finding triaged and remediated, documented in a dated
  in-repo review; full containerised dev stack so the store boots from `bash dev/setup.sh`.
- **Phase 1 — settlement engine.** Per-seller, per-period netting of **card (marketplace-collected)
  against COD (seller-collected)** revenue with shortfall billing. Built as a pure, unit-tested
  calculator over idempotent `grundi_settlement*` tables, with admin generate/approve/pay screens
  and a seller-facing statement. Verified against real production data.
- **Phase 2 — settlement invoicing.** A provider abstraction over **Számlázz.hu Agent-XML** and
  **Billingo v3** with dry-run mode, per-seller API keys, and the three spec outcomes
  (commission / payout+notify / shortfall+notify) plus batch issue.
- **Phase 3 — payments.** Per-seller payment-method toggle enforced at checkout through OpenCart
  events; SimplePay integration prepped end to end.
- **Phase 4 — access control.** Two scoped admin roles, strong-password policy for admins and
  customers, email verification on registration.
- **Phase 5 — catalogue integrations.** Admin-configurable feed banned-words, XLSX import, and an
  idempotent outbound stock-decrement webhook per seller on paid orders.
- **Phase 6 — support.** In-house ticketing for customers and an admin/support queue, seller review
  replies, native newsletter.
- **Phase 8 — lifecycle email (Hungarian).** Weekly/monthly seller summaries, event-driven
  first-sale congratulations, milestone incentives, admin digests and coupon campaigns with dry-run.
  Delivery is **at-most-once**, guarded by `grundi_notify_log` rows, behind a token-guarded daily
  cron endpoint. 34 unit checks plus an end-to-end run against a real database.
- **Phase 9 — carrier integrations** built; **Phase 10 QA** cleared twice.
- **Security review** (dated, in-repo) with findings split into fixed / confirmed-clean /
  accepted-open, plus ops hardening. Buyer-identifying shipping documents are explicitly gitignored
  as PII.

**Status.** Live on a Hetzner VPS. Everything not blocked on the client is finished; the remaining
items need carrier credentials, production SMTP, SimplePay contracts and legal sign-off.

---

### Anfisa Beauty — GetCourse replacement (WordPress)
`getcourse` · PHP / WordPress · Shell tooling · **deployed to VPS staging**

**What it is.** A self-owned WordPress/WooCommerce/LearnDash platform replacing a client's rented
GetCourse online school — a subscription wellness business (face yoga, massage, meditation,
nutrition, skincare).

**The problem it solves.** The client's courses, members and payments lived inside a closed
third-party platform they neither owned nor could extend. This migrates the whole business onto
infrastructure they control, without losing the course, membership or booking features.

**Stack.** WordPress · WooCommerce 10.9 · WooCommerce Subscriptions · LearnDash · Amelia booking ·
Fluent Forms · TranslatePress · Bunny Stream · Podman local stack · Docker + Caddy on a VPS ·
Playwright smoke suite.

**Features**

- **Reproducible environments first.** A `stack.sh` harness gives `up / down / destroy`, arbitrary
  WP-CLI passthrough, `backup`, and a **`restore-drill`** that proves the latest backup actually
  restores by standing it up on a scratch stack on another port. Site install and eight test
  personas are seeded idempotently.
- **Playwright smoke suite as the deploy gate** — every phase adds specs; green is required before
  any deploy.
- **Phase 1 — commerce core.** Products, coupons, EU €5 / worldwide €15 shipping zones, classic
  cart and checkout, COD placeholder gateway, mail logging for email verification.
- **Phase 2 — LMS.** LearnDash courses, lessons and progression.
- **Phase 3 — membership + trial** via WooCommerce Subscriptions.
- **Phase 5 — booking.** Amelia appointments (Zoom pending client credentials).
- **Phase 6 — funnel.** Fluent Forms gated webinar and assessment flows.
- **Phase 7 — bilingual EN/HU** chrome and flows via TranslatePress.
- **Phase 4 — video gating layer** code-complete against Bunny Stream.
- **Deployment** to a VPS staging environment behind Caddy with nightly backups, plus a documented
  phase-by-phase status file that separates "done" from "blocked on the client".

**Status.** Phases 0–3 and 5–7 done on VPS staging with a green smoke suite; Phase 4 code-complete.
Remaining blockers are commercial, not technical — Stripe keys, plugin licences, a Bunny account,
Zoom credentials and GetCourse admin access for the content migration.

---

### Tripora — trip-plan marketplace (React Native)
`tripora` · React Native / Expo

**What it is.** A cross-platform mobile marketplace for buying premade multi-day trip itineraries,
with an AI trip generator and subscriptions.

**Stack.** React Native + Expo 54 · TypeScript · Supabase (Postgres, Auth, Storage, Edge Functions) ·
Stripe React Native · React Navigation · i18next (7 languages) · react-native-maps.

**Features**

- **Marketplace** with search, filter modal, favourites and reviews; trip detail with image gallery;
  day-by-day itinerary view; activity detail with maps.
- **AI generator + questionnaire + "plan my trip"** flows for bespoke itineraries.
- **Subscriptions** — plans screen, management screen, value calculator, Stripe Edge Functions for
  create / cancel / resume plus a webhook proxy and subscription webhook.
- **Purchases** with a secured purchases table and payment-update fix path.
- **Completion tracking** — mark activities done, resume where you left off.
- **Offline support** — network-status hook, offline banner, trips cache service, cached images.
- **Extras** — calendar sync, push notifications, trip sharing with shared-trips screen, social
  sharing, dark mode, skeleton loading, image optimisation pipeline, 7-language localisation.

---

## 3. Mobile apps & games

### Lumino Block Puzzle — published Android game
`toms-blocks` · Flutter · **v4.52.2 (build 156) live at 100% on Google Play production**

**What it is.** A polished, heavily featured block-puzzle game with cloud save, leaderboards, IAP
and a live-ops content calendar. The most commercially mature app in this portfolio.

**Stack.** Flutter · Riverpod · GoRouter · Firebase (Auth, Firestore, Analytics, App Check) ·
Google Mobile Ads · RevenueCat · Google Play Billing 8.3 · flutter_soloud · Sign in with Apple ·
Codemagic CI.

**Features**

- **Multiple game modes** behind a `GameModeStrategy` abstraction, including a full **hexagonal
  board** implementation (own coord system, grid, clear detection, piece model and renderer)
  alongside the classic square board.
- **Puzzle campaign** — puzzle packs, stages, sessions, a **solver** used to validate that puzzles
  are actually solvable, a progress map screen and a remote pack source.
- **Live-ops content** — daily challenges, weekly challenges, monthly challenges, daily login
  rewards and achievements, each with its own controller and test suite.
- **Economy** — coins, boosters (with a purchase sheet and catalogue), bundles, shop, free-coin
  loops, rewarded ads and a **purchase ledger** with account-isolation tests.
- **17 distinct clear effects** (confetti, dissolve, domino, fireworks, freeze, glitch, laser,
  lightning, magnet, particles, shatter, slide, vortex …) behind a catalogue + factory, with a
  live preview widget.
- **Theming** — unlockable themes, seasonal themes, palettes, glass board rendering, texture cache
  and custom painters.
- **Cloud sync + leaderboards + accounts** (Google, Apple), with account-isolation guarantees.
- **11 languages / 12 locales** — EN, HU, DE, ES, FR, IT, JA, NL, PL, PT (+ PT-BR), ZH.
- **43 test files** covering engine, board, grid, pieces, hex, modes, scoring, serialisation,
  rotation/game-over edge cases, second chance, challenges, achievements, purchases and UI.
- **Store automation** — a `store_design` HTML→screenshot pipeline, capture scripts for store art
  and IAP shots, and a Play internal-track publish script.

**Status.** Shipped and monetising. Billing 8 migration promoted to 100% production, meeting
Google's deadline.

---

### Lumino Rush — arcade spin-off
`lumino-rush` · Flutter · **published to Google Play internal testing**

**What it is.** A time-pressure arcade variant of the block-puzzle core, built as a six-milestone
brief with a rhythm mode prototype.

**Stack.** Flutter · shared_preferences · audioplayers · RevenueCat.

**Features**

- **Rush engine** — separate from the Lumino engine: zones, objectives, a tray planner, tier table
  and level definitions.
- **BEAT mode** — a conveyor/rhythm prototype where the board is tapped on the beat, with its own
  screen and test coverage.
- **Level select + campaign progression**, shop, themes, sound service and monetisation hooks.
- **Remote config** for tuning economy and difficulty without a release.
- **Accessibility** — colour-blindness test coverage.
- **9 test files** across the engine, tray planner, beat mode, pause, polish, remote config and
  colour-blindness.
- **Tooling** — a music generator, a Play listing script, screenshot capture tooling and a
  `.claude` skill for driving the app during development.

---

### Cossy — social network for cosplayers
`cossy` · Flutter + Supabase · **v1.0.0+21 on Google Play internal track**

**What it is.** A mobile-first social network for cosplayers with the whole cosplay-creation toolkit
built in — built solo for RevenueCat Shipaton 2026.

**Stack.** Flutter · Supabase (Postgres, Auth, Realtime, Storage — EU region) with **30 SQL
migrations** and **8 Edge Functions** · RevenueCat (premium + coins) · Cloudflare R2 (media) ·
self-hosted ONNX moderation · OneSignal · Sentry · PostHog · Codemagic.

**Features**

- **Social core** — chronological feed with seen-tracking and discovery ranking, creator profiles,
  WIP/activity posts, media upload via signed URLs, tags and leaderboards.
- **Cosplan project tracker** — nested checklists, budgets and reference images per costume build.
- **Chat** — DMs, group chat, topic rooms, room-creation RLS policies.
- **Events** — convention listings, "met at con" connections, QR scanning, packing sheets.
- **Coins economy** — awards sent between users, supporter spotlight, a paid Coins currency and a
  premium subscription, both through RevenueCat with a webhook-driven entitlement sync.
- **Moderation** — an admin moderation queue plus self-hosted ONNX-based content moderation.
- **Notifications** — per-category preferences, push service, OneSignal tag sync and a weekly digest
  function.
- **Bilingual** EN/JA, including localised Play Store listings.
- **Security** — a dated security audit in-repo, dedicated hardening migrations and an explicit RPC
  surface migration.
- **Account deletion** as a first-class Edge Function, plus community guidelines and DMCA policy.
- **Seeding tooling** — demo and launch seeders in Python and SQL, with matching cleanup scripts.

---

### Country Guide — travel etiquette & law, with an editorial QA gate
`country-guide` · Flutter + Node/TS + JSON Schema

**What it is.** An offline-capable app that tells a traveller, per country, how to behave like a
local and what will get them into legal trouble — built on **one versioned data layer powering
three products**: the consumer app, a licensable API, and white-label B2B builds.

**Why it's interesting engineering.** The hard part isn't the app, it's *trust*: this is legal and
cultural advice, so wrong content is worse than no content. The repo's centre of gravity is a
machine-enforced editorial pipeline that makes un-reviewed claims physically unable to ship.

**Stack.** Flutter (schema-driven offline cards) · Node/TypeScript API · git-versioned JSON country
packs · zero-dependency JSON Schema validator · Claude Code content pipeline.

**Features**

- **Country packs as source of truth** — one git-versioned JSON file per country, validated against
  a formal JSON Schema with a per-field metadata envelope (severity, sources, confidence, hedge,
  `ai_generated`, `human_reviewed`).
- **A two-layer validator** (`schema/validate.mjs`, no dependencies): structure first, then a
  **non-negotiable QA gate** that fails publish if any field at `severity >= caution` has empty
  `sources[]`, if any such field still has `human_reviewed: false`, if a `confidence: low` field
  lacks a hedge string, or if `meta.overall_risk` disagrees with the value auto-derived from the
  pack's severity spread.
- **Publish vs staging modes** — gate failures are hard errors in publish mode (CI blocks the
  merge) and warnings in staging so drafts can progress.
- **AI drafts, humans approve.** The first real pack (Japan) is deliberately committed in the
  pre-approval state and is *correctly blocked* from publishing until a human verifies each cited
  source. The block is the feature.
- **Passport-rules join layer** — visa, consular and extra-jurisdiction rules resolved at query
  time rather than duplicated into every pack.
- **Content pipeline** — fetch sources → draft → QA gate → human review → publish, plus a
  risk re-derivation command to bring stored values back in line after severity edits.

**Status.** Schema, validator, QA gate and the first cited pack are built; app and API are
scaffolded. Flutter app at v0.4.0+4.

---

### Two Lies and a Truth — party/social game
`liesandtruth` · Flutter + Firebase

**What it is.** A cross-platform social deduction game — each player writes three statements, one
true, and others vote — with five distinct product modes.

**Stack.** Flutter · Riverpod · GoRouter · Freezed · Firebase (Auth, Firestore, Cloud Functions,
Messaging, Analytics, Crashlytics, Remote Config) · AdMob · RevenueCat · GitHub Actions CI.

**Features**

- **Five modes** — realtime multiplayer, async friend challenges, daily challenge, single-player
  trivia ladder and a couples/date mode with its own question banks and premium sessions.
- **Server-authoritative rules** in Cloud Functions with enforced invariants: exactly three
  statements, one valid true index, no vote after the reveal deadline, no round finalisation without
  votes, winner is first to the target score.
- **11 callable functions** — create room, quick-play enqueue/cancel, rematch, ready check, submit
  round, submit vote, next round, solo results, solo progress save, content seeding.
- **Room lifecycle engineering** — presence, recovery, state, quick-play matchmaking, rate limiting
  and blocking, each with a dedicated test file.
- **Progression** — achievements, avatar studio, match history, player stats, leaderboards, friends
  and an activity feed.
- **Shop** — category packs, couples packs, cosmetics, IAP catalogue and validation.
- **Polish** — custom GLSL shader backdrops (fire/blue/passion portals), emoji reactions, reveal
  sequences, story summaries, share cards, haptics, audio.
- **Privacy engineering** — a Crashlytics PII scrubber with its own test.
- **Analytics** — BigQuery SQL for daily retention, async conversion, quick-play conversion and a
  monetisation summary.
- **Quality** — 26 Flutter test files, two integration tests (smoke + full journey), and three CI
  workflows (CI, E2E, security).
- **Content pipeline** — see `lie2me-collector` below.

---

### Block Puzzle (Unity) — the original
`block-puzzle` · C# / Unity · ~118 MB

**What it is.** The Unity predecessor to the Flutter Lumino line: a commercial block-puzzle build
with full mobile monetisation and platform services wired up.

**Stack.** Unity · C# · Firebase (App, Auth, Database) · Google Mobile Ads (banner, interstitial,
rewarded, app-open, plus UMP consent and iOS ATT) · Google Play Games Services · Google Sign-In ·
RevenueCat · LeanTween · Spine animation.

**Features** — board/piece engine with multiple game modes (bomb, time), boosters and a booster
subscription, a hold slot, currency and spark managers, a shop catalogue and UI, theme definitions
and a theme manager, a tutorial system with scripted board data, a popup/screen framework, a dev
overlay, and editor tooling for batch Android/iOS builds with Gradle and Xcode post-processors.

---

## 4. Automation, pipelines & developer tooling

### n8n workflow library — 21 productised automations
`n8n-workflows` · JavaScript

**What it is.** A version-controlled library of production n8n automations built and deployed
through the n8n MCP server, each packaged as a reusable product rather than a one-off flow.

**Features**

- **21 workflow packages**, each with a README, an intake script, a runner, status-event handling,
  an ops test and exported n8n draft JSON: AI email assistant, automatic invoicing, booking/review
  onboarding, content client status, CRM↔Google Sheets, customer follow-up (generic and Gmail with a
  reply monitor), document generation, Hermes task automation (email intake, GitHub PR, daily
  review), incoming-invoice AI, advanced invoice extraction (Gmail *and* Outlook intake, Sheets
  template), lead automation with follow-up runner, Meta Ads assistant, payment reminders, report
  automation, supplier feed/inventory, support-ticket routing, webshop chatbot, WooCommerce admin
  automation, WordPress security monitor (IMAP intake + daily digest), and a catalogue upsert that
  feeds the Webautomatizáció storefront.
- **MCP-driven authoring** — a connection checker that asserts the server advertises the required
  builder tools (`search_nodes`, `validate_workflow`, `create_workflow_from_code`, `update_workflow`,
  `prepare_test_pin_data`, `test_workflow`, `get_execution`).
- **Safety rule** — workflow drafts stay inactive until publishing is explicitly approved.

---

### Automated trading pipeline — research-first quant scaffold
`trading` · Python · NautilusTrader

**What it is.** A research → backtest → paper → small-live pipeline for stocks/ETFs and forex on a
small account, built on NautilusTrader targeting Interactive Brokers.

**Why it's interesting engineering.** It is written primarily as a defence against self-deception.
The repo leads with `RESEARCH.md` — framework and broker selection rationale, cost arithmetic,
Hungarian tax treatment (ETÜ/TBSZ) and a realism section — and the code enforces its conclusions.

**Features**

- **Five "iron rules" encoded as constraints**, not comments: no backtest may run without the real
  fee model (gross P&L is treated as fiction); trade frequency capped by design; a cash account
  means long-only, no shorting or leverage; the kill switch lives at the broker, not in code.
- **A validation ladder that cannot be skipped** — in-sample → walk-forward → untouched holdout →
  ≥4 weeks IBKR paper → tiny live, justified by the observed R² < 0.025 between backtest Sharpe and
  live performance.
- **Real IBKR/OANDA fee models** in `config/fees.py`, mandatory in every backtest.
- **A smoke backtest that is designed to lose money** — an EMA cross over synthetic random-walk data
  whose expected output is a small net loss, most of it fees, demonstrating exactly what ~30
  trades/day costs a $5k account.

**Status.** Research, stack selection and a green smoke backtest done (Nautilus 1.231). Next rung
is an IBKR paper account.

---

### Question Collector — AI content pipeline at scale
`lie2me-collector` · Python

**What it is.** The content factory behind the trivia modes of *Two Lies and a Truth* — designed for
repeatable drops of 60, 600 or 10,000+ questions without quality collapse.

**Stack.** Python · OpenAI-compatible providers (plus a mock provider needing no key) · Docker ·
PostgreSQL · n8n integration.

**Features**

- **Quota-driven content map** balancing category and difficulty, emitting the exact
  `soloQuestions[]` / `soloLevels[]` schema the app consumes.
- **Nine validation checks** — schema shape, exact duplicate prompts, windowed near-duplicate
  detection, option count/uniqueness/length balance, difficulty fit, category fit and profanity.
- **CI-grade quality gates** — `--max-warnings`, `--max-warning-rate` and per-code rate ceilings
  (e.g. `near_duplicate_prompt=0.02`) that fail the build.
- **Chunked scaling** — generate in batches, dedupe each batch against previous bundles with
  `--seed-bundle`, then `merge-bundles` into one deduplicated release.
- **Internet research mode** — searches Wikipedia and Wikidata, extracts facts *with source URLs*,
  and builds multiple-choice questions from verified facts rather than model recall.
- **Reviewer CSV** output for the human review pass, alongside the JSON bundle and validation report.
- **Async job API** (`POST /api/internet-build` → `202` + job id, poll `GET /api/jobs/{id}`) with
  API-key auth, designed to be driven by n8n; ships four n8n workflow definitions and a Firestore
  publisher.

---

### Amelia ↔ Számlázz.hu bridge
`amelia-szamlazzhu` · PHP · WordPress plugin

**What it is.** A commercial WordPress plugin that connects Amelia Booking directly to Számlázz.hu
invoicing, so a completed booking produces a compliant Hungarian invoice with no manual step.

**Features** — Amelia bridge, invoice generator, Számlázz.hu API client, a custom **field manager**
with a field-setup admin page, a **retry queue** for failed invoice attempts, an invoice log viewer,
a **licence manager** for commercial distribution, and Hungarian localisation (`.po`/`.pot`).

---

### Resume Maker — JD-targeted CV variant generator
`resumemaker` · JavaScript

**What it is.** A local-first pipeline that keeps one canonical JSON Resume and generates
role-targeted variants from a job description — without ever letting the AI edit protected facts.

**Features** — profile-weighted keyword extraction from a job description; deterministic *or*
AI-enhanced variant generation; a **path-based field-lock config** so personal details are immutable
while `basics.summary` may be tailored; a validator that fails if a locked field drifted; and
generation metadata logged per run for auditability.

---

## 5. Websites & client work

### nstudio.hu — agency site
`nstudio` · Next.js 16 · React 19 · next-intl (EN/HU) · Tailwind · Radix · Framer Motion ·
Resend + nodemailer · Cloudflare Turnstile · Redis-backed rate limiting.

Bilingual agency site with services, case studies, an audit funnel, a project-start flow, a CV page
and dark mode — plus the **app-compliance surface** for the mobile portfolio: privacy policies,
terms, refund policy and GDPR account/data-deletion pages and API endpoints for Lumino, Lie to Me
and SoforPlus (driver and passenger), as required by the app stores.

### oldal.ai — product marketing site
`oldal.ai` · Next.js 16 (Turbopack) · React 19 · Tailwind 4 · Motion · MDX blog
(`gray-matter` + `next-mdx-remote`) · Docker standalone build on Coolify.

Marketing site for the Oldal builder: features, solutions, pricing, about, contact, a git-based MDX
blog with reading time and GFM, sitemap/robots, and an animation-forward but restrained design
system optimised for Core Web Vitals.


---

## 6. Earlier work & experiments

### Crypto trading Telegram bot
`crypto-telegram-bot` · Python 3.13 · python-telegram-bot v22 · SQLite · Stripe + Web3.py ·
Matplotlib · Render.

A production Telegram bot with 40+ commands: real-time prices, matplotlib charts with technical
indicators (RSI, MACD, Bollinger), multi-timeframe analysis, sentiment and gas tracking across four
chains; portfolio tracking with P/L and BTC/ETH benchmarking; price and DEX-swap ("whale") alerts
polled every 60 s; multi-chain wallet tracking (Ethereum, BSC, Polygon, Arbitrum); and a four-tier
subscription system billed either by **Stripe** or **on-chain USDT/USDC on Polygon** with
transaction verification.

### Multi-agent travel planner
`travelplanning` · Python · Anthropic SDK · Flask · SQLite.
An early multi-agent system — flight, hotel, itinerary, multi-city, seasonal-optimiser, group/budget,
insurance/visa, accessibility/carbon and photo/voice agents behind a trip orchestrator, with Amadeus
and Booking integrations and a Flask API. Superseded by later work but a clear demonstration of
agent decomposition.

---

## 7. Full repository index

Legend — **Prod**: running in production · **Ship**: published to an app store ·
**Build**: complete, not deployed · **WIP**: active development · **Arch**: archived/superseded ·
**Dup**: duplicate of another entry · **Empty**: placeholder, no code.

| Repository | What it is | Primary stack | State |
|---|---|---|---|
| `app.oldal.ai` | AI website-builder SaaS (multi-tenant, credit-billed) | Next.js 15, Drizzle/PG, Claude | Prod |
| `oldal.ai` | Marketing site for the above | Next.js 16, MDX | Prod |
| `xprize` | AI "business employee" — booking, invoicing, discoverability | Next.js, PG event log, Gemini | Prod |
| `sentinel` | WordPress security & ops platform + WP agent plugin | Next.js 15, Drizzle/PG, PHP | Build |
| `cossy` | Cosplay social network with creator economy | Flutter, Supabase, RevenueCat | Ship |
| `country-guide` | Travel etiquette & law app with a machine-enforced editorial QA gate | Flutter, Node/TS, JSON Schema | WIP |
| `toms-blocks` | Lumino Block Puzzle — published game | Flutter, Firebase, RevenueCat | Ship |
| `lumino-rush` | Arcade spin-off with rhythm mode | Flutter, RevenueCat | Ship |
| `soforapp` | Ride-hailing with partner bidding | NestJS, Prisma, Flutter ×2 | Build |
| `cook-solutions` | `sear` — single-moment cooking coach (audio ML) | Python, numpy DSP | WIP |
| `trading` | Research-first quant pipeline (fees mandatory, validation ladder) | Python, NautilusTrader | WIP |
| `grundi-` | Multi-vendor marketplace (greenfield rebuild) | Next.js, NestJS, Drizzle | Build |
| `grundi-opencart` | The live grundi.hu OpenCart marketplace: settlement engine, invoicing, lifecycle email | PHP, OpenCart 3, Webkul | Prod |
| `getcourse` | Anfisa Beauty — GetCourse replacement (LMS, memberships, booking) | WordPress, LearnDash, Woo | Prod |
| `IngatlanSpanyol/ingatlanspanyol` | Costa del Sol property search (EN/ES/HU) | Next.js 16, Drizzle/PG | Prod |
| `eutory` | European brand directory, deals & comparisons | Next.js 16, Prisma/PG | Build |
| `eutory-collector` | AI research → validated content drafts for Eutory | Python, Typer, LLM | Build |
| `jp-jobs` | Japan hospitality job board (v3.6.49) | Next.js 16, Prisma, NextAuth | Prod |
| `euroszaki` | EU skilled-trades job board | Laravel, Filament, Blade | Build |
| `webautomatizacio` | Productised automation shop | Next.js, Medusa, PG | Build |
| `liesandtruth` | Two Lies and a Truth — social game (client + backend) | Flutter, Firebase Functions | Build |
| `hermes-vps-agent/hermes-agent` | Autonomous agent task protocol + delivered work | JS, WP theming, n8n | Build |
| `hermes-vps-agent/hermes-dashboard` | Phone-first approval dashboard for the agent | Next.js 14, PG, OpenRouter | Build |
| `n8n-workflows` | 21 productised n8n automations | JS, n8n MCP | Prod |
| `lie2me-collector` | AI question-generation pipeline with quality gates | Python, Docker, n8n | Build |
| `aiwp2` | NexusPress — AI WordPress block SaaS | Next.js 14, Prisma/MySQL, Gemini | Build |
| `aiwp-plugin` | NexusPress WordPress companion plugin v1.0.0 | PHP, JWT | Build |
| `nstudio` | Agency site + app compliance/legal surface | Next.js 16, next-intl | Prod |
| `tripora` | Trip-plan marketplace app | React Native/Expo, Supabase | Build |
| `block-puzzle` | Unity predecessor of Lumino | Unity, C#, Firebase | Arch |
| `cookai` | Earlier weekend-scoped attempt at `sear` | Python | Arch |
| `crypto-telegram-bot` | Crypto trading/alerts bot with subscriptions | Python, Stripe, Web3 | Arch |
| `amelia-szamlazzhu` | Amelia ↔ Számlázz.hu invoicing bridge | PHP, WordPress | Build |
| `hostjobs` | WordPress predecessor of the JP job board | WordPress, Blocksy, Fluent | Arch |
| `jp-hospitality-import-jobs` | PHP job-feed importer for the above | PHP, SQLite | Arch |
| `resumemaker` | JD-targeted CV variant generator with field locks | Node.js | Build |
| `travelplanning` | Multi-agent trip planner | Python, Anthropic, Flask | Arch |
| `nagytamasgit/ingatlanspanyol` | Earlier snapshot of the property platform | Next.js 16 | Dup |
| `soforplus` · `hermes` | Reserved names, no code | — | Empty |

---

## Working style

A few things visible across these repositories that may matter more than any single feature list:

- **Security is designed in, not bolted on.** HMAC-signed wire protocols with nonce replay
  protection, AES-256-GCM secrets at rest, hand-implemented and tested TOTP, RLS proven by test,
  a hardener with its own bypass-attack suite, PII scrubbing before crash reporting. Inherited
  systems get a dated, written security review that separates fixed from accepted-open, and
  buyer-identifying documents are treated as PII and kept out of version control.
- **Honest evaluation.** `sear` freezes its test split and refuses to look at it until day 11;
  `hermes-agent` calibrates its QA scorer against a deliberately-bad control site; `trading` ships
  a smoke backtest *designed to lose money* so the fee drag is impossible to ignore; every README
  states what is *not* done.
- **Decisions are written down.** `DECISIONS.md`, `locked-decisions.md`, `PROJECT_GUARDRAILS.md`,
  `FIX-BACKLOG.md` — including the ideas that were cut and the condition that would reopen them.
- **Correctness enforced by machine, not by discipline.** `country-guide` cannot publish a legal
  claim that lacks a source or a human sign-off; `trading` refuses a backtest without the real fee
  model; `getcourse` gates every deploy on a green Playwright suite and proves its backups by
  restoring them onto a scratch stack; `grundi-opencart` guarantees at-most-once email through
  database guard rows.
- **Tested where it counts.** 43 test files in Lumino, 26 in Two Lies and a Truth, ~200 assertions
  across 15 verification suites in Oldal, Playwright E2E in four platforms, a settlement engine
  unit-tested then verified against real production payouts.
- **Shipped, not just built.** Google Play production and internal tracks, VPS and Cloud Run
  deployments, a live multi-vendor marketplace handling real seller payouts, store listing
  automation, staged rollouts and migration deadlines met.

---

*Prepared 2026-08-22. Repository contents are private; demos and read-only code access available on
request — [nagytmas@gmail.com](mailto:nagytmas@gmail.com).*
