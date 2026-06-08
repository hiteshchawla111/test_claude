# MVP Scope — Property Listing Portal

**Status:** Draft v1.0 · **Last updated:** 2026-06-05
**Related:** [PRD](./PRD.md) · [Functional Requirements](./Functional-Requirements.md) · [Development Phases](./Development-Phases.md) · [Feature Roadmap](./Feature-Roadmap.md)

---

## 1. MVP Thesis (Minimum Lovable Product)

> A seeker can **search a real, fresh, trustworthy set of listings on a map and in a list, view rich detail, save what they like, and contact the lister without leaking their personal data** — and a lister can **create, manage, and get qualified leads on a listing that goes live quickly but safely.**

The MVP is not just minimal; it must be **lovable**: discovery feels fast, listings feel trustworthy (moderated, not spammy), and the contact flow protects both sides. Those three qualities — **fast discovery, trust, safe contact** — are the differentiators we will not cut.

**Hypotheses the MVP validates:**
- H1: Seekers prefer map-first, well-filtered discovery and will convert to leads at ≥ 4%.
- H2: Listers will self-serve listings if creation is simple and leads are qualified.
- H3: Lightweight moderation + masked contact keeps fraud < 1.5% and protects contact data.

---

## 2. What's IN the MVP

Feature IDs map to [Functional Requirements](./Functional-Requirements.md) (FR-*) and roadmap items.

### 2.1 Accounts & Auth (FR-1.x)
- [x] Email/password sign-up & login, email verification.
- [x] Roles: Guest, Seeker, Lister (Owner/Agent), Admin. See [User Roles](./User-Roles.md).
- [x] Phone OTP verification for listers (anti-fraud baseline).
- [x] Password reset, basic profile.

### 2.2 Listings (FR-2.x)
- [x] Create/edit listing: type (sale/rent), price, property type, beds/baths, area, address + geolocation, amenities, description.
- [x] Photo upload (multi-image, resize/transcode, ordering, cover image).
- [x] Listing lifecycle: draft → pending review → live → expired/archived.
- [x] Listing expiry + manual refresh.

### 2.3 Search & Discovery (FR-3.x)
- [x] Keyword + structured search (location, price range, type, beds, baths, amenities).
- [x] **Map-based search** with clustering + draw/bounding-box; list/map sync.
- [x] Sort: relevance, price, newest, distance. Pagination/infinite scroll.
- [x] SEO-friendly, SSR listing detail and search pages.

### 2.4 Listing Detail (FR-4.x)
- [x] Photo gallery, attributes, map location (approximate vs exact per privacy), amenities, lister card.
- [x] Report-listing action.

### 2.5 Contact / Leads (FR-5.x)
- [x] Privacy-preserving inquiry: in-platform message form; contact reveal per lister setting.
- [x] Lister lead inbox; email notification of new leads.
- [x] Anti-spam: rate limiting + captcha on contact.

### 2.6 Engagement (FR-6.x)
- [x] Favorites / saved listings.
- [x] Saved searches with **email alerts** (new matches).

### 2.7 Trust & Safety + Admin (FR-7.x, FR-8.x)
- [x] Moderation queue (pending + flagged listings).
- [x] Duplicate detection signal (basic: address/phone/image hash heuristics).
- [x] Report handling, listing takedown, user suspend/ban.
- [x] Audit log of admin actions.

### 2.8 Platform / Cross-cutting
- [x] Responsive web (mobile-first).
- [x] Email notifications (verification, leads, saved-search alerts).
- [x] Baseline analytics events (search, view, lead, save).
- [x] WCAG 2.1 AA baseline, SEO baseline, performance budgets — see [NFRs](./Non-Functional-Requirements.md).

---

## 3. What's OUT of the MVP (Deferred) — with Rationale

| Deferred item | Why deferred | Target phase |
|---|---|---|
| In-app real-time chat / messaging threads | Lead form + email validates contact demand first; real-time adds infra cost | Phase 2 |
| SMS & push notifications | Email validates alert demand; SMS adds cost/compliance | Phase 2 |
| Social/SSO login (Google/Apple) | Email auth sufficient to validate; add to reduce signup friction later | Phase 2 |
| Monetization: featured/boost, agent subscriptions, payments | Need liquidity before charging; avoids dampening supply | Phase 3 |
| Verified-agent KYC (document-based) | Phone OTP is a good-enough trust baseline for launch | Phase 2 |
| Listing comparison tool | Nice-to-have; not core to first lead | Phase 2 |
| Advanced fraud ML / image-similarity dedupe at scale | Heuristic signals suffice at low volume | Phase 2/3 |
| Recommendations / personalization | Needs data volume first | Phase 3 |
| Recently-sold / price-history / AVM | Data sourcing + accuracy risk | Later |
| Multi-language UI / multi-currency | Architecture is i18n-ready; full localization deferred | Phase 3 |
| Native mobile apps | Responsive web covers launch | Out of horizon |
| Virtual tours / video / 3D | Photos validate media demand first | Phase 2/Later |
| Bulk listing import / agent CRM tools | Manual + simple multi-listing mgmt covers launch | Phase 2 |

---

## 4. Explicit MVP Quality Bar (non-negotiable)

These are IN even though they add effort, because cutting them breaks the "lovable" thesis:
- Masked/privacy-preserving contact (no raw contact data in public HTML/API).
- Pre-publish moderation (or fast post-publish review) to keep fraud low at launch.
- Map search performance within budget (clustering + viewport queries).
- SSR/SEO for search + detail pages (organic acquisition is core to growth).
- Image transcoding/resizing + CDN (cost + performance).

---

## 5. MVP Exit Criteria

- All IN features pass acceptance criteria in [User Stories](./User-Stories.md).
- KPIs instrumented and reporting (see PRD §3.2).
- Fraud/fake-listing rate measured < 1.5% on live listings.
- Search p75 < 800 ms; listing detail mobile LCP p75 < 2.5 s.
- WCAG 2.1 AA automated + key-flow manual audit passed.
- Security review (auth, contact-data exposure, rate limiting) passed.

---

## 6. Open Questions
1. Pre-publish review (slower, safer) vs post-publish review (faster supply)? Recommendation: **pre-publish for new listers, auto-publish for trusted/verified listers** after first approvals.
2. Exact vs approximate map pin default for privacy?
3. Saved-search alert frequency default (instant vs daily digest)?
