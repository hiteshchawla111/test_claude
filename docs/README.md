# Property Listing Portal — Product Planning Docs

**Working name:** PropFind
**Status:** Draft v1.0 · **Last updated:** 2026-06-05

A two-sided property marketplace (Zillow / 99acres / Rightmove style): property owners and agents list properties for sale or rent; buyers and renters search, browse on a map, save, and contact listers through a privacy-preserving flow. These documents define the product, scope, requirements, and delivery plan. Suggested stack: **React + TypeScript (SSR)** frontend, REST/GraphQL backend, PostgreSQL+PostGIS / dedicated search engine, object storage + CDN — kept implementation-agnostic where possible.

---

## Document Index

| # | Document | Summary |
|---|---|---|
| 1 | [PRD.md](./PRD.md) | **Product Requirements Document** — vision, problem statement, goals & success metrics (concrete KPIs), personas, assumptions, constraints, dependencies, risks, out-of-scope, and a high-level solution summary. The "why" and "what." |
| 2 | [MVP-Scope.md](./MVP-Scope.md) | **MVP Scope** — the Minimum Lovable Product: what's IN vs explicitly OUT (deferred), with rationale, the non-negotiable quality bar, and MVP exit criteria. |
| 3 | [User-Roles.md](./User-Roles.md) | **User Roles & Permissions** — Guest, Seeker, Lister (Owner/Agent), Admin, Super Admin; a full permissions matrix, field-visibility rules, and lister/listing trust states. |
| 4 | [User-Stories.md](./User-Stories.md) | **User Stories** — organized by role and epic in "As a [role], I want [goal] so that [benefit]" format, each with acceptance criteria, MoSCoW/P-tag priority, and story points. |
| 5 | [Functional-Requirements.md](./Functional-Requirements.md) | **Functional Requirements** — numbered (FR-1…FR-10) by feature area: auth, listings, search/map, detail, contact/leads, favorites/saved searches, trust & moderation, admin, notifications, monetization, plus cross-cutting rules. |
| 6 | [Non-Functional-Requirements.md](./Non-Functional-Requirements.md) | **Non-Functional Requirements** — performance, scalability, security, accessibility (WCAG 2.1 AA), SEO, availability, maintainability, observability, compliance (GDPR), browser support, and i18n. |
| 7 | [Feature-Roadmap.md](./Feature-Roadmap.md) | **Feature Roadmap** — Now / Next / Later releases mapped to phases, prioritized via MoSCoW + value/effort, with FR and story traceability and sequencing rationale. |
| 8 | [Development-Phases.md](./Development-Phases.md) | **Development Phases** — Phase 0 (Foundation) → Phase 1 (MVP) → Phase 2 (Engagement & Trust) → Phase 3 (Monetization & Scale), each with goals, scope, build sequencing, dependencies, and exit criteria. |

---

## How the Docs Connect (Traceability)

```
PRD (goals, personas, KPIs)
  └─► MVP-Scope (what we build first)
        └─► User-Roles ──┐
        └─► User-Stories ─┼─► Functional-Requirements (FR-IDs)
        └─► NFRs ─────────┘        │
                                   └─► Feature-Roadmap (Now/Next/Later)
                                         └─► Development-Phases (delivery + exit criteria)
```

- **Goals** (PRD) trace to **requirements** (FRs/NFRs), which trace to **user stories** (acceptance criteria).
- **Roadmap** items reference the same **FR-IDs** and **story IDs** so naming stays consistent across documents.
- **Phases** gate delivery against **exit criteria** tied back to NFRs and MVP scope.

---

## Reading Order

1. Start with the **PRD** for context and goals.
2. Read **MVP-Scope** to see what ships first and why.
3. Use **User-Roles**, **User-Stories**, **Functional-Requirements**, and **NFRs** as the buildable spec.
4. Use **Feature-Roadmap** and **Development-Phases** to plan and sequence delivery.

---

## Open Questions (consolidated)

1. Launch market, default currency/language.
2. Map/geocoding vendor and budget ceiling.
3. Agent verification (KYC/badge) at launch or Phase 2.
4. Monetization timing vs unit economics.
5. Applicable data-privacy regime(s).
6. Pre-publish vs post-publish moderation default (recommendation: pre-publish for new listers, auto-publish for trusted listers).
7. Allow gated guest inquiry or require signup to contact (recommendation: require signup for MVP).
