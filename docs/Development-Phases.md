# Development Phases — Property Listing Portal

**Status:** Draft v1.0 · **Last updated:** 2026-06-05
**Related:** [PRD](./PRD.md) · [MVP Scope](./MVP-Scope.md) · [Functional Requirements](./Functional-Requirements.md) · [Non-Functional Requirements](./Non-Functional-Requirements.md) · [Feature Roadmap](./Feature-Roadmap.md)

Phases are sequenced to **de-risk early** (foundation, trust, performance) and **deliver value incrementally**. Each phase lists goal, scope, key deliverables, dependencies, and exit criteria.

---

## Phase 0 — Foundation (Enablement)

**Goal:** Stand up the technical and product foundations so feature work is fast, safe, and observable.

**Scope / Key Deliverables**
- Architecture decisions (ADRs): frontend (React + TypeScript, SSR via Next.js recommended), API style (REST or GraphQL), data store (PostgreSQL + PostGIS), search engine plan, object storage + CDN, media pipeline.
- Repo, CI/CD, environments (dev/staging/prod), feature flags.
- Design system / component library + base responsive layout shell.
- AuthN/AuthZ scaffolding and role model ([User Roles](./User-Roles.md)).
- Observability baseline: structured logging, metrics, tracing, error tracking (NFR-8).
- Security baseline: secrets management, dependency scanning, TLS/HSTS (NFR-3).
- i18n scaffolding (externalized strings, locale formatting, RTL-ready) (NFR-11).
- Analytics event schema for KPIs (CFR-5, NFR-8.5).

**Dependencies:** Vendor selections (map/geocode, email, storage/CDN, search).

**Exit Criteria**
- [ ] CI green with lint, type-check, tests, performance-budget, a11y checks (NFR-7.4).
- [ ] "Hello world" SSR page deploys to staging+prod via pipeline with zero-downtime.
- [ ] Logging/metrics/tracing visible in dashboards; error alerts fire.
- [ ] Auth scaffolding enforces server-side authorization.
- [ ] ADRs documented and approved.

---

## Phase 1 — MVP (Minimum Lovable Product)

**Goal:** Launch the trustworthy, map-first marketplace with safe contact. Validate hypotheses H1–H3 ([MVP Scope](./MVP-Scope.md) §1).

**Scope** — all "Now/Must" items in [Feature Roadmap](./Feature-Roadmap.md) §2:
- **Auth & onboarding:** FR-1.1–1.7, 1.10.
- **Listings:** create/edit/lifecycle/expiry + photos (FR-2.1–2.8).
- **Search:** structured + keyword + **map/clustering** + sort/paginate (FR-3.1–3.8).
- **Detail:** SSR + SEO + report (FR-4.1–4.4).
- **Contact/Leads:** privacy-preserving inquiry, lead inbox, reveal-on-consent (FR-5.1–5.6).
- **Engagement:** favorites + saved searches w/ email alerts (FR-6.1–6.4).
- **Trust/Admin:** moderation queue, fraud/dup signals, suspend/ban, audit, admin mgmt, KPI dashboards (FR-7.1–7.8, FR-8.1–8.2).
- **Notifications:** transactional + alert emails (FR-9.1–9.2).
- **Cross-cutting NFRs:** performance budgets, WCAG 2.1 AA, SEO, security, i18n-ready.

**Suggested build sequencing (to de-risk):**
1. Auth + role model + listing data model + media pipeline (riskiest infra first).
2. Listing create/lifecycle + moderation queue (supply + trust together — don't ship publishing without moderation).
3. Search (structured) → then map search (highest-effort/perf-risk item, C-2).
4. SSR detail + SEO.
5. Contact/leads (privacy-preserving) + email notifications.
6. Favorites + saved searches + alerts.
7. Admin dashboards + hardening (security, a11y, performance, fraud tuning).

**Dependencies:** Phase 0 complete; map/geocode + email + storage/CDN + search live.

**Exit Criteria** (= [MVP Scope](./MVP-Scope.md) §5)
- [ ] All P0 user stories pass acceptance criteria ([User Stories](./User-Stories.md)).
- [ ] Search p75 < 800 ms; detail mobile LCP p75 < 2.5 s (NFR-1).
- [ ] No contact data in public HTML/API (verified by security review, NFR-3.4).
- [ ] Fraud/fake-listing rate measured < 1.5% on live listings.
- [ ] WCAG 2.1 AA automated + key-flow manual audit passed (NFR-4).
- [ ] KPI instrumentation reporting to dashboards (PRD §3.2).
- [ ] Security review passed (auth, uploads, rate limiting, authorization).

---

## Phase 2 — Engagement & Trust Depth

**Goal:** Improve retention and lister value; harden trust and add channels.

**Scope** — "Next" items ([Feature Roadmap](./Feature-Roadmap.md) §3):
- Real-time in-app chat threads (FR-5.7).
- SMS/push notifications + notification preferences (FR-6.5, 9.3, 9.4).
- Social/SSO login (FR-1.8); optional MFA (FR-1.9).
- Lister analytics (FR-2.9) + bulk management + import (FR-2.10, 2.12).
- Verified-agent KYC + badge (FR-7.9).
- Listing comparison (FR-3.10), geosearch polygon/radius (FR-3.9), similar listings (FR-4.5).
- Lead quality scoring (FR-5.8); video/virtual tours (FR-2.11).
- Admin role granularity, config, template mgmt (FR-8.3–8.5).

**Dependencies:** MVP contact/lead model (FR-5), trust states ([User Roles](./User-Roles.md) §5), notification infra.

**Exit Criteria**
- [ ] Chat adoption and lead conversion meet defined targets; no regression in spam metrics.
- [ ] Verified-agent badge live with revocation workflow + audit.
- [ ] Multi-channel notifications respect preferences and compliant opt-out.
- [ ] Lister analytics accurate and performant for high-volume agents.
- [ ] Fraud rate trending toward < 0.5% with tightened signals.

---

## Phase 3 — Monetization & Scale

**Goal:** Introduce revenue and prepare for market/scale expansion.

**Scope** — "Later" items ([Feature Roadmap](./Feature-Roadmap.md) §4):
- Featured/boosted listings (clearly labeled, bounded ranking) (FR-10.1).
- Agent subscription tiers + billing/payments (FR-10.2, 10.3); pay-per-lead (FR-10.4).
- Full i18n: multi-language + multi-currency + RTL + hreflang (NFR-11.3–11.4).
- Recommendations/personalization (FR-3.11); ML fraud/image-similarity at scale (FR-7.10).
- Scale-out: dedicated search engine if not already, caching/cost optimization (NFR-2).

**Dependencies:** Phase 2 analytics + agent tooling; billing infra; accumulated data for reco/ML.

**Exit Criteria**
- [ ] Monetization live with billing, invoices, and clear promoted-listing labeling; ranking integrity preserved.
- [ ] At least one additional locale/currency shipped end-to-end (incl. SEO hreflang).
- [ ] System sustains mature-target load (PRD §3.2) within NFR-1/NFR-2 budgets.
- [ ] Unit-economics reporting available to the business.

---

## Cross-Phase Risk Handling

| Risk (PRD) | Phase addressed | Mitigation in plan |
|---|---|---|
| R-1 Cold start | Phase 1 | Free posting, seed/import tooling (B-6 import accelerated if needed) |
| R-2 Fake listings | Phase 1 → 2 | Moderation + signals (MVP) → KYC + ML (Phase 2/3) |
| R-3 Contact leakage | Phase 0 → 1 | Privacy model designed in foundation; enforced + reviewed in MVP |
| R-4 Geo/media cost | Phase 0 → 3 | Caching/transcoding from foundation; monetization offsets in Phase 3 |
| R-5 SEO underperformance | Phase 1 | SSR + structured data + sitemaps in MVP |
| R-6 Lead spam | Phase 1 → 2 | Rate limit + captcha (MVP) → lead scoring (Phase 2) |

---

## Sequencing Summary

```
Phase 0 (Foundation) ──► Phase 1 (MVP) ──► Phase 2 (Engagement & Trust) ──► Phase 3 (Monetization & Scale)
   enablement            launch & validate     retention & hardening          revenue & expansion
```

Each phase is independently shippable and gated by its exit criteria before the next begins.
