# Feature Roadmap — Property Listing Portal

**Status:** Draft v1.0 · **Last updated:** 2026-06-05
**Related:** [PRD](./PRD.md) · [MVP Scope](./MVP-Scope.md) · [Functional Requirements](./Functional-Requirements.md) · [Development Phases](./Development-Phases.md)

Prioritization framework: **MoSCoW** for scope + **value-vs-effort** for ordering within a release. Now/Next/Later maps to the phases in [Development Phases](./Development-Phases.md).

---

## 1. At-a-Glance

| Horizon | Phase | Theme | Goal |
|---|---|---|---|
| **Now** | Phase 0 + Phase 1 | Foundation + MVP | Launch a trustworthy, map-first marketplace with safe contact |
| **Next** | Phase 2 | Engagement & Trust depth | Richer comms, more channels, stronger trust, agent tooling |
| **Later** | Phase 3 + Beyond | Monetization & Scale | Revenue, personalization, localization |

---

## 2. NOW — Foundation + MVP (Phase 0 → Phase 1)

**Theme:** Liquidity + Trust + Safe contact. Ship the Minimum Lovable Product.

| Priority | Feature | FRs | Stories |
|---|---|---|---|
| Must | Email auth, verification, password reset | FR-1.1–1.4, 1.6, 1.10 | A-1, A-2, A-3 |
| Must | Lister phone-OTP onboarding | FR-1.5, 1.7 | A-4 |
| Must | Listing create/edit + lifecycle + expiry | FR-2.1, 2.5–2.8 | B-1, B-3, B-4 |
| Must | Photo upload/transcode/order | FR-2.3, 2.4 | B-2 |
| Must | Structured + keyword search | FR-3.1–3.3, 3.6–3.8 | C-1, C-3 |
| Must | Map search w/ clustering | FR-3.4, 3.5 | C-2 |
| Must | SSR listing detail + SEO | FR-4.1–4.3 | D-1 |
| Must | Privacy-preserving contact + lead inbox | FR-5.1–5.6 | E-1, E-2, E-3 |
| Must | Favorites + saved searches w/ email alerts | FR-6.1–6.4 | F-1, F-2 |
| Must | Moderation queue + fraud signals + audit | FR-7.1–7.8 | G-1, G-2, G-3, G-4 |
| Must | Admin user/listing mgmt + KPI dashboards | FR-8.1, 8.2 | (admin) |
| Must | Transactional + alert emails | FR-9.1, 9.2 | H-1 |
| Must | Report listing | FR-4.4, 7.5 | D-2 |

**Release goal/KPIs:** see PRD §3.2 (MVP targets). Exit criteria in [MVP Scope](./MVP-Scope.md) §5.

---

## 3. NEXT — Engagement & Trust Depth (Phase 2)

**Theme:** Deepen retention and lister value; harden trust.

| Priority | Feature | FRs | Stories |
|---|---|---|---|
| Should | Real-time in-app chat threads | FR-5.7 | E-4 |
| Should | SMS/push notifications + preferences | FR-6.5, 9.3, 9.4 | F-3, H-2 |
| Should | Social/SSO login | FR-1.8 | A-5 |
| Should | Listing analytics for listers | FR-2.9 | B-5 |
| Should | Bulk listing management (agents) | FR-2.10, 2.12 | B-6 |
| Should | Verified-agent KYC + badge | FR-7.9 | G-5 |
| Should | Listing comparison | FR-3.10 | C-4 |
| Should | Geosearch radius / polygon | FR-3.9 | (search ext) |
| Should | Similar/nearby listings | FR-4.5 | (detail ext) |
| Should | Lead quality scoring | FR-5.8 | (leads ext) |
| Could | Video / virtual tour media | FR-2.11 | (media ext) |
| Could | Trusted-lister tuning + role granularity | FR-8.3–8.5 | (admin ext) |

---

## 4. LATER — Monetization & Scale (Phase 3 + Beyond)

**Theme:** Revenue, personalization, and market expansion.

| Priority | Feature | FRs | Stories |
|---|---|---|---|
| Could | Featured/boosted listings | FR-10.1 | I-1 |
| Could | Agent subscription tiers | FR-10.2 | I-2 |
| Could | Billing/payments integration | FR-10.3 | (billing) |
| Could | Pay-per-lead packages | FR-10.4 | (leads monetization) |
| Could | Multi-language + multi-currency (full i18n/RTL) | NFR-11.3–11.4 | (i18n) |
| Could | Recommendations / personalization | FR-3.11 | (reco) |
| Could | ML fraud + image-similarity at scale | FR-7.10 | (trust ML) |
| Won't (now) | Price history / recently-sold / AVM | FR-4.6, 3.12 | — |
| Won't (now) | Native mobile apps, payments/escrow | — | — |

---

## 5. Sequencing Rationale

1. **Trust before scale:** moderation, fraud signals, and masked contact ship in the MVP because a marketplace that leaks contact data or hosts fake listings dies fast (PRD R-2, R-3).
2. **SEO + map are core, not later:** organic acquisition and map discovery are differentiators, so they are MVP despite cost/effort.
3. **Monetize after liquidity:** charging before supply/demand exist would suppress the very liquidity we need (PRD A-4), so monetization is Phase 3.
4. **Channels follow validation:** email validates alert/lead demand cheaply; SMS/push/chat come once the funnel is proven.
5. **i18n-ready, localize later:** architecture supports i18n/RTL from day one (NFR-11), but full localization waits for expansion.

---

## 6. Dependencies Across Releases

- Real-time chat (Phase 2) depends on the lead/contact model shipped in MVP (FR-5).
- Verified-agent badge (Phase 2) depends on lister trust states defined in MVP ([User Roles](./User-Roles.md) §5).
- Monetization (Phase 3) depends on listing analytics + agent tooling (Phase 2) and billing infra.
- Personalization (Phase 3) depends on MVP analytics events (CFR-5, NFR-8.5) accumulating data.
