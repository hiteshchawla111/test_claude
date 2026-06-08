# Non-Functional Requirements (NFRs) — Property Listing Portal

**Status:** Draft v1.0 · **Last updated:** 2026-06-05
**Related:** [PRD](./PRD.md) · [Functional Requirements](./Functional-Requirements.md) · [Development Phases](./Development-Phases.md)

NFR IDs are referenced from MVP exit criteria and phase exit criteria.

---

## NFR-1 · Performance

| ID | Requirement |
|---|---|
| NFR-1.1 | Search results API p75 < 800 ms (MVP), < 400 ms (mature). |
| NFR-1.2 | Listing detail mobile LCP p75 < 2.5 s; CLS < 0.1; INP < 200 ms. |
| NFR-1.3 | Map search must use viewport bounding-box + server-side clustering; never ship all points to the client. |
| NFR-1.4 | Images served as responsive, CDN-cached, modern formats (WebP/AVIF) with lazy loading. |
| NFR-1.5 | Initial JS payload budget for key pages kept lean (code-splitting, route-based chunks); track via performance budget in CI. |
| NFR-1.6 | Search/list pages SSR/SSG where possible for fast first paint and SEO. |

---

## NFR-2 · Scalability

| ID | Requirement |
|---|---|
| NFR-2.1 | Architecture supports horizontal scaling of stateless API/web tiers. |
| NFR-2.2 | Search uses a dedicated geo + full-text engine (e.g., OpenSearch/Elasticsearch with geo queries) at scale; PostGIS acceptable for early volume. |
| NFR-2.3 | Media pipeline (upload → transcode → CDN) scales independently and asynchronously. |
| NFR-2.4 | Target capacity: 25k+ live listings and 75k WAU at mature targets without architecture change. |
| NFR-2.5 | Caching strategy (CDN, search result cache, geocode cache) to bound third-party cost and load. |

---

## NFR-3 · Security

| ID | Requirement |
|---|---|
| NFR-3.1 | All traffic over HTTPS/TLS; HSTS enabled. |
| NFR-3.2 | Passwords hashed with a strong adaptive algorithm (bcrypt/argon2); no plaintext or reversible storage. |
| NFR-3.3 | Authorization enforced server-side on every request (own-only vs admin); never trust client role claims. |
| NFR-3.4 | Contact data (phone/email) never present in public HTML/API responses; revealed only post-auth + consent. |
| NFR-3.5 | Rate limiting & bot defense on auth, contact, report, and search-scrape endpoints; captcha on high-abuse actions. |
| NFR-3.6 | Input validation/sanitization; protection against XSS, CSRF, SQL/NoSQL injection, SSRF (geocode proxying). |
| NFR-3.7 | Secrets managed via a secrets manager; no secrets in code/repo. |
| NFR-3.8 | File-upload security: validate MIME/type/size, transcode, strip metadata, scan for malware, serve from isolated origin. |
| NFR-3.9 | Audit logging for admin/privileged actions (ties to FR-7.7). |
| NFR-3.10 | Periodic dependency and security scanning in CI; security review before each phase release. |

---

## NFR-4 · Accessibility (WCAG)

| ID | Requirement |
|---|---|
| NFR-4.1 | Conform to WCAG 2.1 AA for all primary user flows (search, detail, contact, listing creation, auth). |
| NFR-4.2 | Full keyboard operability incl. gallery, map controls, filters, and modals (focus trapping/return). |
| NFR-4.3 | Semantic HTML + ARIA where needed; form errors programmatically associated and announced. |
| NFR-4.4 | Color contrast ≥ 4.5:1 (text); visible focus indicators. |
| NFR-4.5 | Images require meaningful alt text; map provides an accessible (list) alternative. |
| NFR-4.6 | Automated a11y checks in CI + manual audit of key flows before release. |

---

## NFR-5 · SEO

| ID | Requirement |
|---|---|
| NFR-5.1 | SSR/SSG for search and listing detail pages; crawlable, indexable content. |
| NFR-5.2 | Clean, canonical, human-readable URLs; canonical tags to avoid duplicate-content penalties. |
| NFR-5.3 | schema.org structured data for listings (RealEstateListing/Product/Offer). |
| NFR-5.4 | Auto-generated XML sitemaps + robots rules; noindex for thin/expired pages. |
| NFR-5.5 | Per-page meta titles/descriptions, Open Graph/Twitter cards, optimized images. |
| NFR-5.6 | Core Web Vitals within "Good" thresholds (ties to NFR-1.2). |

---

## NFR-6 · Availability & Reliability

| ID | Requirement |
|---|---|
| NFR-6.1 | Target 99.9% monthly availability for core read paths (search/detail). |
| NFR-6.2 | Graceful degradation: if map/geocode provider fails, list search still works. |
| NFR-6.3 | Async jobs (email, transcode, alerts) use retries + dead-letter queues; idempotent processing. |
| NFR-6.4 | Backups with tested restore; defined RPO ≤ 24h, RTO ≤ 4h (MVP). |
| NFR-6.5 | Zero-downtime deploys; feature flags for risky changes. |

---

## NFR-7 · Maintainability

| ID | Requirement |
|---|---|
| NFR-7.1 | TypeScript across frontend (and backend where applicable); strict mode; minimal `any`. |
| NFR-7.2 | Modular architecture; shared design system/component library; reuse over duplication. |
| NFR-7.3 | Automated tests: unit + integration + key E2E flows; meaningful coverage on critical paths. |
| NFR-7.4 | CI gates: lint, type-check, tests, performance budget, a11y checks. |
| NFR-7.5 | Documented API contracts (OpenAPI/GraphQL schema) and ADRs for key decisions. |

---

## NFR-8 · Observability

| ID | Requirement |
|---|---|
| NFR-8.1 | Centralized structured logging (correlation IDs across requests/jobs). |
| NFR-8.2 | Metrics + dashboards for the PRD KPIs and system health (latency, error rate, queue depth). |
| NFR-8.3 | Distributed tracing on request paths (search, contact). |
| NFR-8.4 | Alerting on SLO breaches, error spikes, fraud-signal anomalies, deliverability drops. |
| NFR-8.5 | Product analytics events (search/view/save/lead) for funnel reporting. |

---

## NFR-9 · Compliance & Data Privacy

| ID | Requirement |
|---|---|
| NFR-9.1 | GDPR-aligned: lawful basis, consent for marketing, data export & "right to be forgotten" (ties to FR-1.10). |
| NFR-9.2 | Cookie/consent management for analytics/marketing. |
| NFR-9.3 | Data minimization; PII encrypted at rest; access controlled and logged. |
| NFR-9.4 | Clear privacy policy & terms; contact-data sharing governed by consent. |
| NFR-9.5 | Data retention policy for leads, messages, and expired listings. |
| NFR-9.6 | Photo metadata (EXIF/geo) stripped to protect lister location privacy. |

---

## NFR-10 · Browser & Device Support

| ID | Requirement |
|---|---|
| NFR-10.1 | Latest 2 versions of Chrome, Firefox, Safari, Edge; iOS Safari + Android Chrome. |
| NFR-10.2 | Responsive, mobile-first design (primary traffic assumed mobile). |
| NFR-10.3 | Progressive enhancement; core browsing works without heavy client JS. |

---

## NFR-11 · Internationalization (i18n)

| ID | Requirement |
|---|---|
| NFR-11.1 | i18n-ready architecture from MVP: externalized strings, locale-aware formatting (dates, numbers, currency). |
| NFR-11.2 | RTL/LTR layout support designed-in (even if single language ships first). |
| NFR-11.3 | Multi-currency and multi-language UI delivered in Phase 3 (per [Feature Roadmap](./Feature-Roadmap.md)). |
| NFR-11.4 | Locale-aware URL/SEO strategy (hreflang) when localized content ships. |
