# Technical Architecture — Property Listing Portal (PropFind)

**Status:** Draft v1.0 · **Last updated:** 2026-06-05
**Audience:** Senior engineers building Phase 0 → Phase 1.
**Related:** [PRD](./PRD.md) · [MVP Scope](./MVP-Scope.md) · [Functional Requirements](./Functional-Requirements.md) · [Non-Functional Requirements](./Non-Functional-Requirements.md) · [User Roles](./User-Roles.md) · [Development Phases](./Development-Phases.md) · [Tech Stack](./Tech-Stack.md) · [Database Schema](./Database-Schema.md) · [API Design](./API-Design.md) · [ADRs](./ADR.md)

---

## 0. Guiding Principle

**The simplest architecture that is genuinely scalable.** For the MVP (Phase 1) we ship a **modular monolith API** + **Next.js web app** + **managed services** (Postgres/PostGIS, object storage + CDN, email, queue). Every seam where the monolith would later split into a service (search, media, notifications, fraud) is drawn explicitly so Phase 2/3 scale-out is a refactor, not a rewrite. This directly satisfies NFR-2.1 (horizontal scaling of stateless tiers) and NFR-2.4 (25k listings / 75k WAU without architecture change), while avoiding premature microservices.

This is captured formally in [ADR-002](./ADR.md#adr-002-modular-monolith-vs-microservices-for-mvp).

---

## 1. High-Level System Architecture

```
                                  ┌─────────────────────────────────────────────┐
                                  │                  CDN (edge)                   │
                                  │  static assets · image variants · ISR pages   │
                                  └───────────────┬───────────────────────────────┘
                                                  │
        Browser (mobile-first, WCAG 2.1 AA)       │
   ┌───────────────────────────┐                  │
   │  Next.js Web App           │◄─────────────────┘
   │  (SSR/SSG/ISR + RSC)       │  HTML, JSON
   │  React 19 + TS             │
   └───────────┬───────────────┘
               │ HTTPS / REST (JSON)   Bearer access token (JWT) + refresh cookie
               ▼
   ┌─────────────────────────────────────────────────────────────────────────┐
   │                       API Gateway / Edge (rate limit, WAF, TLS)            │
   └───────────┬───────────────────────────────────────────────────────────────┘
               ▼
   ┌─────────────────────────────────────────────────────────────────────────┐
   │           Modular Monolith API  (Node.js + TypeScript, NestJS)            │
   │                                                                           │
   │  ┌────────┐ ┌─────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────────┐ │
   │  │ Auth   │ │ Listings│ │ Search │ │ Media  │ │ Leads  │ │ Moderation │ │
   │  │ module │ │ module  │ │ module │ │ module │ │ module │ │  module    │ │
   │  └────────┘ └─────────┘ └────────┘ └────────┘ └────────┘ └────────────┘ │
   │  ┌──────────────┐ ┌────────────┐ ┌──────────────┐ ┌────────────────────┐ │
   │  │ Engagement   │ │Notification│ │ Admin/Audit  │ │ Shared kernel       │ │
   │  │(fav/searches)│ │  module    │ │  module      │ │ (authz, events, cfg)│ │
   │  └──────────────┘ └────────────┘ └──────────────┘ └────────────────────┘ │
   └──┬────────────┬──────────────┬──────────────┬───────────────┬────────────┘
      │            │              │              │               │
      ▼            ▼              ▼              ▼               ▼
 ┌─────────┐ ┌──────────┐  ┌──────────┐   ┌──────────┐   ┌─────────────┐
 │Postgres │ │  Redis   │  │ Object   │   │  Queue   │   │  3rd-party  │
 │+PostGIS │ │ cache +  │  │ storage  │   │ (SQS/    │   │  providers  │
 │(primary)│ │ rate-lim │  │ (S3)     │   │ BullMQ)  │   │             │
 │ + read  │ │ + session│  └────┬─────┘   └────┬─────┘   │ map/geocode │
 │ replica │ │ revoke   │       │              │         │ email (SES) │
 └─────────┘ └──────────┘       ▼              ▼         │ captcha     │
                          ┌──────────┐   ┌──────────────┐│ (Turnstile) │
                          │  Image   │   │  Workers     ││ malware scan│
                          │ pipeline │   │ (transcode,  │└─────────────┘
                          │ (Lambda) │   │ email, alerts,│
                          └──────────┘   │ fraud, expiry)│
                                         └──────────────┘
```

**Data flow, read path (search):** Browser → Next.js (SSR) → API `/v1/search` → query Postgres/PostGIS (with Redis result cache) → JSON → SSR HTML cached at CDN via ISR. (FR-3.x, NFR-1.1, NFR-1.6.)

**Data flow, write path (create listing):** Browser → API `/v1/listings` (authz: lister + phone-verified) → Postgres (status `pending_review`) → emits `listing.submitted` event → fraud worker computes signals → moderation queue → admin approve → status `live` → search refresh + `listing.approved` notification. (FR-2.x, FR-7.x.)

**Data flow, media:** Browser requests presigned URL from API → uploads directly to S3 (bypasses API tier) → S3 event triggers image pipeline (transcode, EXIF strip, variants) → CDN serves. (FR-2.3, NFR-1.4, NFR-3.8.)

---

## 2. Frontend Architecture

### 2.1 Rendering strategy (SSR/SSG/ISR vs CSR) — see [ADR-001](./ADR.md#adr-001-rendering-strategy)

**Next.js (App Router, React Server Components)** is mandated by NFR-5.1, NFR-1.6 and FR-4.2. Per-route strategy:

| Route | Strategy | Why |
|---|---|---|
| `/` home, landing, locality hubs | **SSG / ISR** | Marketing/SEO, rarely changes (NFR-5.x) |
| `/search` results (list + map) | **SSR** with cached data | Filters in URL must be crawlable + shareable (FR-3.3); first paint fast (NFR-1.1) |
| `/listing/[slug]` detail | **ISR** (revalidate on edit/expiry) | SEO + schema.org + LCP < 2.5s (FR-4.2, NFR-1.2); content stable between edits |
| Lister dashboard, lead inbox, admin | **CSR** (client components, no SEO) | Auth-gated, dynamic, not indexed |
| Auth pages | **SSR** shell + client form | Fast, no index needed |

Map interaction is a **client component** hydrated inside an SSR shell — the list is server-rendered for SEO and a no-JS fallback (NFR-10.3 progressive enhancement, NFR-4.5 accessible list alternative to map).

### 2.2 Folder structure (App Router, feature-oriented)

```
web/
  app/
    (marketing)/            # SSG/ISR public pages, locality hubs
    (search)/search/        # SSR search results (list + map)
    listing/[slug]/         # ISR detail page
    (auth)/login | signup | reset/
    dashboard/              # CSR lister area: listings, leads, saved-searches, favorites
    admin/                  # CSR admin: moderation, users, audit, KPIs
    api/                    # BFF route handlers (token refresh proxy, sitemap, OG images)
    layout.tsx              # locale + dir(rtl/ltr) provider, theme, analytics
  src/
    features/               # one folder per domain, mirrors API modules
      listings/  search/  leads/  favorites/  saved-searches/
      auth/  moderation/  notifications/
        components/  hooks/  api/  types.ts  schema.ts (zod)
    components/ui/          # design system primitives (shadcn/ui-style)
    components/shared/      # ListingCard, Gallery, MapPanel, FilterBar, DataTable
    lib/                    # api-client, auth, query-client, i18n, analytics, geo
    types/                  # shared cross-feature types (generated from OpenAPI)
  messages/{en}/            # externalized i18n strings (NFR-11.1)
  tests/                    # unit, integration, e2e (Playwright)
```

Mirroring API modules in `features/` keeps the mental model identical front-to-back and isolates the **seams** for future splits (NFR-7.2).

### 2.3 State management & data fetching

- **Server state:** **TanStack Query** for all API reads/mutations in client components — caching, dedup, optimistic updates (favorites toggle F-1), retry. Server components fetch directly via the typed API client.
- **URL as state:** search filters, sort, pagination, map bounds live in the **URL query string** (FR-3.3, FR-3.5). This is the source of truth for search — shareable, SSR-able, back-button-correct.
- **Global UI state:** lightweight **Zustand** store for cross-cutting UI (auth/session snapshot, locale, map↔list hover sync, toast). No Redux — the app's shared mutable state is small; Redux would be an unnecessary abstraction here (NFR-7.2).
- **Forms:** **React Hook Form + Zod** (listing create B-1, auth). Zod schemas shared with the API contract layer for end-to-end type safety (NFR-7.1).

> Note: this is a greenfield, SEO-first, server-rendered marketplace — not the UMS-GB internal app. I deliberately do not import UMS-GB's Redux-Toolkit/`getApi` conventions; server-state via TanStack Query + URL state is the better-fit, simpler pattern for public SSR. If the team later standardizes on RTK, the `features/*/api` layer is the single swap point.

### 2.4 Component strategy

- **Design system** in `components/ui/` (Phase 0 deliverable) — Radix primitives + Tailwind, accessible by construction (NFR-4.x: focus trap, ARIA, keyboard).
- **Shared composites:** `ListingCard`, `PhotoGallery` (keyboard/touch nav + alt text, D-1), `MapPanel` (clustering, viewport sync), `FilterBar`, `LeadForm`, `ModerationTable`.
- Map↔list sync (FR-3.5) via shared Zustand slice holding `hoveredListingId` / `selectedListingId`.

### 2.5 Performance (ties to NFR-1, NFR-5.6, Core Web Vitals)

- **Route-based code splitting** automatic in App Router; `next/dynamic` for the map bundle (heavy, client-only) so it never blocks search-list first paint (NFR-1.5).
- **`next/image`** → AVIF/WebP, responsive `srcset`, lazy load, blur placeholder (NFR-1.4, FR-4.x gallery).
- **Virtualize** result lists > 100 items (`@tanstack/virtual`).
- **Performance budget in CI** (NFR-1.5, NFR-7.4): Lighthouse CI + bundle-size gate; fail PR on regression. LCP/CLS/INP tracked via `web-vitals` → analytics (NFR-8.2).
- **Streaming SSR + Suspense** so above-the-fold listing detail paints before the gallery/map hydrate (LCP < 2.5s, NFR-1.2).

### 2.6 i18n (NFR-11)

`next-intl` with externalized message catalogs; locale-aware number/currency/date formatting from day one; `dir="rtl|ltr"` driven at the layout root so RTL is a config flip, not a rewrite (NFR-11.2). Single locale ships in MVP; hreflang/locale routing wired in Phase 3 (NFR-11.4).

---

## 3. Backend Architecture

### 3.1 Modular monolith (NestJS) — see [ADR-002](./ADR.md#adr-002-modular-monolith-vs-microservices-for-mvp)

One deployable Node.js + TypeScript service, internally partitioned into **modules** with explicit boundaries. Modules communicate **in-process via interfaces + a domain event bus**, never by reaching into each other's tables. This discipline makes later extraction cheap.

| Module | Owns | FRs | Future split candidate |
|---|---|---|---|
| **Auth** | identity, sessions, JWT, OTP, password reset, GDPR export/delete | FR-1.x | — (stays core) |
| **Users/Profile** | profile, lister trust state, contact prefs | FR-1.6, Roles §5 | — |
| **Listings** | listing CRUD, lifecycle, expiry, privacy flags | FR-2.x | — |
| **Media** | presign, variant metadata, ordering | FR-2.3–2.4 | → media service (Phase 2/3) |
| **Search** | query build, geo/viewport, clustering, sort | FR-3.x | → OpenSearch service (Phase 3, NFR-2.2) |
| **Leads** | inquiries, lead inbox, contact reveal, spam block | FR-5.x | → chat service (Phase 2, FR-5.7) |
| **Engagement** | favorites, saved searches, alert matching | FR-6.x | — |
| **Moderation/Trust** | queue, fraud signals, suspend/ban, trust states | FR-7.x | → fraud-ML service (Phase 3) |
| **Admin/Audit** | user/listing admin, audit log, KPI dashboards | FR-8.x | — |
| **Notifications** | provider-abstracted email (later SMS/push) | FR-9.x | → notification service (Phase 2) |
| **Shared kernel** | authz guards, event bus, config/flags, request context | CFR-1, NFR-3.3 | — |

### 3.2 API layer

- **REST/JSON**, versioned `/v1` — see [API-Design.md](./API-Design.md). REST chosen over GraphQL for MVP simplicity, edge/CDN cacheability of GETs, and team velocity.
- **OpenAPI** spec is the contract (NFR-7.5); TypeScript types **generated** for both the API and the web client — single source of truth, no drift, satisfies NFR-7.1 (no `any`).
- Validation at the edge with **Zod/class-validator** on every request body/query (NFR-3.6).

### 3.3 Background jobs / workers (NFR-6.3)

A separate **worker process** (same codebase, different entrypoint) consumes a queue (BullMQ on Redis for MVP; SQS at scale). Jobs:

- **Image transcode** (S3 event triggered) — variants, EXIF strip, malware scan (FR-2.3, NFR-3.8).
- **Email send** (verification, leads, alerts, expiry) with retries + DLQ (NFR-6.3, FR-9.x).
- **Saved-search alert matcher** — on `listing.approved`, match saved searches, enqueue emails (FR-6.3).
- **Fraud/duplicate signal computation** — address/phone/image-hash heuristics on submit (FR-7.3).
- **Listing expiry sweeper** — scheduled; expire + pre-expiry reminders (FR-2.7).

All jobs are **idempotent** (dedupe key = event id) and emit structured logs with correlation IDs (NFR-8.1).

### 3.4 Domain events (the extraction seam)

Modules publish events (`listing.submitted`, `listing.approved`, `lead.created`, `user.suspended`) to an in-process bus that also persists to an **outbox table** and forwards to the queue. Today consumers are in-process; tomorrow the same event can cross a network boundary to an extracted service with no caller changes. This is how the monolith stays splittable (NFR-2, Phase 2/3).

---

## 4. Database Design

Full schema in [Database-Schema.md](./Database-Schema.md). Approach summary:

- **Primary store: PostgreSQL 16 + PostGIS** ([ADR-003](./ADR.md#adr-003-primary-datastore-postgresql--postgis)). Relational integrity + first-class **geography** for map search (FR-3.4) without a second datastore in MVP.
- **Read/write split:** single primary for writes; **one read replica** for search/public reads (NFR-2.1, NFR-6.1). Read-your-own-write goes to primary.
- **Caching layer: Redis** — (a) search-result cache keyed by normalized filter+bbox+zoom (short TTL, NFR-2.5, NFR-1.1), (b) geocode cache to bound provider cost (NFR-2.5), (c) rate-limit counters (NFR-3.5), (d) refresh-token denylist / session revocation, (e) hot listing-detail cache.
- **Geo indexing:** GiST index on `GEOGRAPHY(Point,4326)`; viewport queries use bbox `&&`; server-side clustering via grid snap (`ST_SnapToGrid`) per zoom (FR-3.4, NFR-1.3).
- **Soft delete + audit:** `deleted_at` on user-destructive entities (CFR-4); immutable `audit_log` for admin actions (FR-7.7).

---

## 5. Authentication Strategy — see [ADR-005](./ADR.md#adr-005-authentication-approach)

**JWT access token + rotating refresh token.** Stateless API scaling (NFR-2.1) with server-side revocation for safety.

- **Access token:** short-lived (15 min) JWT (RS256): `sub`, `roles`, `trust_state`, `email_verified`, `phone_verified`. Sent as `Authorization: Bearer`. Never trusted for fine-grained authz — ownership is re-checked server-side (NFR-3.3, CFR-1).
- **Refresh token:** opaque, long-lived, **HttpOnly + Secure + SameSite=Strict cookie**, rotated on every use (reuse detection → revoke family). Stored hashed so it is revocable (logout, ban, password change — A-2, A-3).
- **Password storage:** **Argon2id** (NFR-3.2).
- **Email verification (FR-1.1):** single-use, time-limited signed token; required before contact/listing (server-enforced gate).
- **Phone OTP (FR-1.5, A-4):** 6-digit OTP via SMS provider, Redis-stored with attempt + rate limits; success sets `phone_verified` and grants lister capability on the *same* account (FR-1.7).
- **Password reset (FR-1.4):** single-use, time-limited link; no user enumeration (A-3); may invalidate existing sessions.
- **Login hardening (FR-1.3):** generic errors (no enumeration), Redis rate limit + backoff, captcha after N failures.
- **GDPR (FR-1.10, NFR-9.1):** self-serve export + deletion; deletion is soft + scheduled purge respecting retention (NFR-9.5).
- **Deferred:** social/SSO (FR-1.8) and MFA (FR-1.9) — Phase 2 — extend the *same* Auth module and token model, no parallel flow.

---

## 6. Authorization Strategy — see [ADR-006](./ADR.md#adr-006-authorization-model)

**RBAC + resource-ownership checks**, enforced server-side on every request (NFR-3.3, CFR-1), mapped to [User-Roles.md](./User-Roles.md) §3.

- **Roles** as capability sets (Guest, Seeker, Lister[Owner|Agent], Admin, SuperAdmin); a single account holds multiple capabilities (FR-1.7) — roles are a *set*, not exclusive.
- **Permission model:** `(role) → permissions[]` lookup (seedable), checked by a NestJS **guard**. Coarse gate first (`listing:create`?), then **ownership predicate** (`listing.owner_id === user.id`) for `⚠️ own-only` matrix rows.
- **Resource ownership:** every owned entity carries `owner_id`; a reusable `@OwnsResource()` guard asserts ownership or admin-override (`✅ any`). Admins bypass ownership but are **audited** (FR-7.7).
- **Trust-state gating:** publishing path checks lister trust state (Roles §5) — `unverified` → forced review; `trusted` → auto-publish (FR-7.8).
- **Field-level visibility** (Roles §4): contact data + moderation flags stripped from public serializers; raw phone/email only via consent-gated reveal endpoint (CFR-3, NFR-3.4). Two serializer profiles: `public` and `owner/admin`.
- **Frontend mirrors but never enforces:** `<PermissionGuard>` / `hasPermission()` hides UI; the server is the only authority.

---

## 7. File Upload Strategy — see [ADR-007](./ADR.md#adr-007-filemedia-storage)

**Direct-to-object-storage via presigned URLs**, processed asynchronously, served via CDN. Keeps large uploads off the API tier (NFR-2.3) and enforces security (NFR-3.8).

**Flow (FR-2.3, FR-2.4, B-2):**
1. Client requests `POST /v1/media/presign` (content-type + size); API validates MIME/size/count vs listing limits, returns a **presigned PUT** to a quarantine prefix + a `media_id` row (`status=pending`).
2. Client uploads directly to S3 with progress.
3. S3 `ObjectCreated` event → **image pipeline worker**: magic-number validation (not just extension), **malware scan**, **strip EXIF/GPS** (NFR-3.8, NFR-9.6, R-3), generate responsive variants (thumb/card/gallery/og) in **AVIF + WebP** (NFR-1.4), write to public CDN-fronted bucket, set `media_id.status=ready` + dimensions + variant keys.
4. Reorder + cover set via `PATCH /v1/listings/{id}/media` (FR-2.4).

- **CDN** in front of the public bucket; long cache + content-hash keys; **isolated origin** domain for user media (NFR-3.8).
- **Limits enforced server-side**: max images/listing, max bytes/file, allowed MIME.
- **Originals** private; only derived variants public.

---

## 8. Search Strategy — see [ADR-004](./ADR.md#adr-004-search)

**MVP: PostgreSQL + PostGIS. Scale: OpenSearch.** Aligns with NFR-2.2 ("PostGIS acceptable for early volume", dedicated engine at scale); avoids operating a search cluster before liquidity exists.

**MVP (Phase 1):**
- **Structured filters** (FR-3.2): indexed columns + composite/partial indexes on `status='live'` rows (CFR-2) — see [Database-Schema.md](./Database-Schema.md) (NFR-1.1).
- **Keyword** (FR-3.1): Postgres full-text (`tsvector` on title/description/locality) + GIN index — sufficient for MVP volume.
- **Geo / map search** (FR-3.4, NFR-1.3): viewport bbox query (`geom && ST_MakeEnvelope(...)`) on GiST; **server-side clustering** by zoom via `ST_SnapToGrid` returning cluster centroids + counts — never all points to the client.
- **Sort** (FR-3.6): relevance (ts_rank), price, newest (`published_at`), distance (`ST_Distance`).
- **Result cache** in Redis keyed by normalized query + bbox + zoom, short TTL, invalidated on `listing.approved/expired` (NFR-2.5).

**Scale (Phase 3, NFR-2.2, mature < 400ms NFR-1.1):** project `live` listings into **OpenSearch** (geo_point + geohash_grid clustering, full-text relevance, facet counts). The **Search module interface stays identical**; only its impl swaps. CDC via the outbox/event stream keeps the index fresh.

**Graceful degradation (NFR-6.2):** if map/geocode provider fails, list/filter search still works (geo is additive).

---

## 9. Notification Strategy — see [ADR-008](./ADR.md#adr-008-notifications)

**Event-driven + provider abstraction.** Modules emit domain events; the Notification module decides channel + template.

- **MVP channel: email only** (PRD A-3, FR-9.1) via a `NotificationProvider` interface (impl: AWS SES). Templated, i18n-ready (NFR-11.1), compliant unsubscribe on alerts (FR-9.2, NFR-9.1).
- **Triggers (FR-9.1):** verification, password reset, lead received (FR-5.4), approved/rejected (FR-7.2), expiry reminder (FR-2.7), saved-search alert (FR-6.3).
- **Reliability:** queued sends, retries + DLQ; idempotent by event id (NFR-6.3); deliverability monitored (NFR-8.4).
- **Phase 2:** SMS/push (FR-9.4) + per-channel/per-type preferences (FR-9.3) drop in behind the same interface; `notification_preferences` table already modeled — no caller changes.

---

## 10. Deployment Strategy — see [ADR-009](./ADR.md#adr-009-hostingdeployment-platform)

**Environments:** dev → staging → prod, identical infra-as-code (NFR-6.5).

**Hosting (AWS-class):**
- **Web (Next.js):** Vercel *or* AWS (ECS Fargate + CloudFront). Vercel for fastest SSR/ISR + edge + CWV tooling at MVP; AWS if consolidating ops. Either way the web tier is stateless + horizontally scalable (NFR-2.1).
- **API + workers:** **ECS Fargate** behind an ALB — stateless, auto-scaled (NFR-2.1, NFR-2.4).
- **Data:** RDS Postgres (Multi-AZ) + PostGIS + read replica; ElastiCache Redis; S3 + CloudFront; SES; SQS (or Redis/BullMQ at MVP).

**CI/CD (NFR-7.4):** GitHub Actions → lint, type-check, unit/integration/E2E, **performance budget (Lighthouse CI)**, **a11y checks (axe)**, dependency/security scan (NFR-3.10) → containerize → deploy. **Zero-downtime** rolling/blue-green; **feature flags** gate risky changes (NFR-6.5).

**Scaling path (ties to Development-Phases):**
- Phase 1: single API service + replica + Redis; horizontal API replicas cover MVP targets.
- Phase 2: extract Notification + Chat services off the event bus; add SMS/push.
- Phase 3: extract Search → OpenSearch; CDC pipeline; media service; cost/caching optimization (NFR-2).

**Observability (NFR-8):** OpenTelemetry tracing across request + jobs (NFR-8.3); structured JSON logs + correlation IDs (NFR-8.1); KPI + health dashboards (latency, error rate, queue depth — NFR-8.2); error tracking (Sentry); alerting on SLO breach, error spikes, fraud-signal anomalies, deliverability drops (NFR-8.4); product analytics events (CFR-5, NFR-8.5).

**Backups/DR (NFR-6.4):** automated RDS snapshots + PITR; tested restore; RPO ≤ 24h, RTO ≤ 4h.

---

## 11. Cross-Cutting Concerns

- **Rate limiting & bot defense (NFR-3.5, FR-5.2):** Redis token-bucket per user/IP/endpoint on auth, contact, report, search-scrape; **captcha (Cloudflare Turnstile / hCaptcha)** on signup, contact (E-1), report (D-2). `RateLimit-*` headers (API-Design).
- **Spam/fraud prevention (FR-7.3, R-2, R-6):** submit-time heuristic signals (duplicate address/phone, perceptual image hash, anomalous price); high-severity **auto-holds** publishing (FR-7.4); lead throttling + reportable spam leads (FR-5.6).
- **Contact-data protection (NFR-3.4, CFR-3):** raw phone/email never in public HTML/API; only via authenticated, consent-gated, logged reveal endpoint (E-3).
- **Secrets (NFR-3.7):** AWS Secrets Manager / SSM; nothing in repo; rotated.
- **Error handling:** consistent error envelope; no stack traces to clients; correlation id surfaced for support.
- **Input security (NFR-3.6):** validation/sanitization everywhere; parameterized queries (ORM) prevent SQLi; geocode calls **proxied + allow-listed** (SSRF); CSP + output encoding (XSS); CSRF defense via SameSite cookies + double-submit on cookie-auth paths.

---

## 12. Traceability — Architecture → Requirements

| Architectural decision | Satisfies |
|---|---|
| Next.js SSR/ISR per route | NFR-5.1, NFR-1.6, FR-4.2, FR-3.3 |
| Modular monolith + event seams | NFR-2.1, NFR-2.4, NFR-7.2 |
| Postgres + PostGIS primary | FR-3.4, NFR-2.2 |
| JWT + rotating refresh | FR-1.2/1.3, NFR-2.1, NFR-3.2/3.3 |
| RBAC + ownership + field visibility | Roles §3/§4, CFR-1/3, NFR-3.3/3.4 |
| Presigned upload + async pipeline | FR-2.3/2.4, NFR-1.4, NFR-3.8, NFR-9.6 |
| PostGIS-now / OpenSearch-later | FR-3.x, NFR-1.1, NFR-2.2 |
| Event-driven email + provider abstraction | FR-9.x, NFR-6.3 |
| Containers + CI gates + observability | NFR-6, NFR-7.4, NFR-8 |
| Rate limit + captcha + fraud signals | NFR-3.5, FR-5.2, FR-7.3/7.4 |
