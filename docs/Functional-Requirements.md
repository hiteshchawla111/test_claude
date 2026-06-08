# Functional Requirements — Property Listing Portal

**Status:** Draft v1.0 · **Last updated:** 2026-06-05
**Related:** [PRD](./PRD.md) · [User Stories](./User-Stories.md) · [User Roles](./User-Roles.md) · [Non-Functional Requirements](./Non-Functional-Requirements.md)

**Priority key:** P0 = MVP (Must) · P1 = Phase 2 (Should) · P2 = Phase 3+ (Could)
Each FR maps to user-story epics A–I.

---

## FR-1 · Accounts & Authentication (Epic A)

| ID | Requirement | Priority |
|---|---|---|
| FR-1.1 | Users can register with email + password; email verification required before contacting/listing. | P0 |
| FR-1.2 | Users can log in/out; sessions expire per policy; "remember me" optional. | P0 |
| FR-1.3 | Login is protected against brute force (rate limiting, lockout/backoff) and avoids user enumeration. | P0 |
| FR-1.4 | Users can reset password via single-use, time-limited email link. | P0 |
| FR-1.5 | Listers must verify a phone number via OTP before creating listings. | P0 |
| FR-1.6 | Users can edit basic profile (name, avatar, contact preferences). | P0 |
| FR-1.7 | A single account may hold both Seeker and Lister capabilities. | P0 |
| FR-1.8 | Social/SSO login (Google/Apple) with safe account linking. | P1 |
| FR-1.9 | Optional MFA for listers/admins. | P1 |
| FR-1.10 | Account deletion / data export (GDPR) self-serve request flow. | P0 |

---

## FR-2 · Listing Management (Epic B)

| ID | Requirement | Priority |
|---|---|---|
| FR-2.1 | Listers can create a listing with: transaction type (sale/rent), price, currency, property type, beds, baths, area/size, address, geolocation (lat/long), amenities, description, availability date. | P0 |
| FR-2.2 | Address is geocoded; lister can fine-tune the map pin; system stores both approximate and exact coordinates. | P0 |
| FR-2.3 | Listers can upload multiple photos; server resizes/transcodes to responsive variants and strips EXIF/geo metadata. | P0 |
| FR-2.4 | Listers can reorder photos and set a cover image; enforce max count and per-file size limits. | P0 |
| FR-2.5 | Listing lifecycle: draft → pending review → live → expired/archived; rejected → editable. | P0 |
| FR-2.6 | Listers can edit, unpublish, and soft-delete their own listings; material edits may re-trigger review. | P0 |
| FR-2.7 | Listings expire after a configurable period; lister is notified pre-expiry and can refresh in one action. | P0 |
| FR-2.8 | Per-listing privacy controls: expose exact address (Y/N), contact reveal mode. | P0 |
| FR-2.9 | Per-listing analytics (views, unique viewers, saves, leads) for the owner. | P1 |
| FR-2.10 | Bulk listing management (multi-select edit/refresh/archive) for agents. | P1 |
| FR-2.11 | Listing video / virtual tour media. | P1 |
| FR-2.12 | Bulk import (CSV/feed) for agents. | P1 |

---

## FR-3 · Search & Discovery (Epic C)

| ID | Requirement | Priority |
|---|---|---|
| FR-3.1 | Keyword search across title/description/locality. | P0 |
| FR-3.2 | Structured filters: location/locality, price range, transaction type, property type, beds, baths, area range, amenities, availability. Filters combine with AND. | P0 |
| FR-3.3 | Filter and sort state is encoded in the URL (shareable, SSR-rendered). | P0 |
| FR-3.4 | Map-based search: clustered pins, pan/zoom, "search this area" bounding-box query. | P0 |
| FR-3.5 | List and map views are synchronized (hover/select highlights in both). | P0 |
| FR-3.6 | Sort by relevance, price (asc/desc), newest, distance-from-point. | P0 |
| FR-3.7 | Pagination or infinite scroll with stable ordering. | P0 |
| FR-3.8 | "No results" state with relaxed-filter suggestions. | P0 |
| FR-3.9 | Geosearch radius / draw-polygon area search. | P1 |
| FR-3.10 | Listing comparison (side-by-side). | P1 |
| FR-3.11 | Personalized recommendations / "similar listings." | P2 |
| FR-3.12 | Commute/points-of-interest and school overlays. | P2 |

---

## FR-4 · Listing Detail (Epic D)

| ID | Requirement | Priority |
|---|---|---|
| FR-4.1 | Detail page shows gallery, all public attributes, amenities, description, map pin, lister card. | P0 |
| FR-4.2 | Page is server-rendered with schema.org structured data and canonical URL for SEO. | P0 |
| FR-4.3 | Map pin respects per-listing approximate/exact setting. | P0 |
| FR-4.4 | "Report listing" action with reason capture. | P0 |
| FR-4.5 | "Similar listings" / nearby listings module. | P1 |
| FR-4.6 | Price history / recently-sold context. | P2 |

---

## FR-5 · Contact & Leads (Epic E)

| ID | Requirement | Priority |
|---|---|---|
| FR-5.1 | Seekers contact listers via an inquiry form; lister's raw phone/email is never exposed in public HTML/API. | P0 |
| FR-5.2 | Contact actions are protected by captcha and rate limiting (per user, IP, listing). | P0 |
| FR-5.3 | Listers have a lead inbox with listing context, message, timestamp, and read/responded status. | P0 |
| FR-5.4 | New-lead email notification to lister. | P0 |
| FR-5.5 | Contact reveal (call/email) honored per lister's per-listing consent setting; event logged. | P0 |
| FR-5.6 | Listers can report/block spam leads. | P0 |
| FR-5.7 | Threaded real-time in-app chat tied to a listing. | P1 |
| FR-5.8 | Lead quality scoring / qualification (e.g., verified seeker, intent signals). | P1 |
| FR-5.9 | Auto-reply / templated responses for agents. | P2 |

---

## FR-6 · Engagement: Favorites & Saved Searches (Epic F)

| ID | Requirement | Priority |
|---|---|---|
| FR-6.1 | Seekers can favorite/unfavorite listings; favorites persist per account. | P0 |
| FR-6.2 | Favorites page lists saved listings with current status (e.g., "no longer available"). | P0 |
| FR-6.3 | Seekers can save a search (filter set) and receive email alerts on new matches. | P0 |
| FR-6.4 | Saved-search alert frequency configurable; one-click unsubscribe; manage/delete. | P0 |
| FR-6.5 | SMS/push alert channels. | P1 |
| FR-6.6 | Shareable favorite collections / shortlists. | P2 |

---

## FR-7 · Trust, Safety & Moderation (Epic G)

| ID | Requirement | Priority |
|---|---|---|
| FR-7.1 | Moderation queue of pending + flagged listings, filterable by status/age/signal severity, with SLA timers. | P0 |
| FR-7.2 | Approve/reject listings with reason; rejection notifies lister; all actions audited. | P0 |
| FR-7.3 | Automated fraud/duplicate signals: duplicate address/phone, perceptual image-hash duplicates, anomalous pricing, blacklisted patterns. | P0 |
| FR-7.4 | High-severity signals auto-hold publishing until reviewed; thresholds tunable. | P0 |
| FR-7.5 | Report handling for listings and users; abusive reporting rate-limited. | P0 |
| FR-7.6 | Suspend/ban users; suspension hides listings and blocks contact; reversible and audited. | P0 |
| FR-7.7 | Immutable, searchable admin audit log (who/what/when/why). | P0 |
| FR-7.8 | Trusted-lister auto-publish after N clean approvals. | P0 |
| FR-7.9 | Verified-agent KYC workflow + badge (revocable). | P1 |
| FR-7.10 | ML-based fraud/image-similarity detection at scale. | P2 |

---

## FR-8 · Admin & Operations (Epic G/H)

| ID | Requirement | Priority |
|---|---|---|
| FR-8.1 | Admins can search/manage users and listings (view, edit, take down). | P0 |
| FR-8.2 | Admins can view platform reporting/dashboards (supply, demand, leads, fraud KPIs). | P0 |
| FR-8.3 | Role/permission management for admins. | P1 (Super Admin only) |
| FR-8.4 | Feature flags / configuration management. | P1 |
| FR-8.5 | Saved-search/alert and notification template management. | P1 |

---

## FR-9 · Notifications (Epic H)

| ID | Requirement | Priority |
|---|---|---|
| FR-9.1 | Transactional emails: verification, password reset, lead received, listing approved/rejected, expiry reminder, saved-search alerts. | P0 |
| FR-9.2 | All marketing/alert emails include compliant unsubscribe; transactional emails are deliverability-monitored. | P0 |
| FR-9.3 | User notification preferences (per channel, per type). | P1 |
| FR-9.4 | SMS/push channels. | P1 |

---

## FR-10 · Monetization (Epic I) — Phase 3

| ID | Requirement | Priority |
|---|---|---|
| FR-10.1 | Featured/boosted listings with clear "promoted" labeling and bounded ranking effect. | P2 |
| FR-10.2 | Agent subscription tiers (limits, analytics, badges). | P2 |
| FR-10.3 | Billing, invoices, payment provider integration. | P2 |
| FR-10.4 | Lead-package / pay-per-lead options. | P2 |

---

## Cross-cutting Functional Rules

- **CFR-1** All write actions are authorized against [User Roles](./User-Roles.md) (own-only vs admin-any).
- **CFR-2** All listings carry a moderation status; only `live` listings appear in public search.
- **CFR-3** Contact data exposure is gated by authentication + lister consent (never in public payloads).
- **CFR-4** All user-facing destructive actions (delete, ban) are soft and audited.
- **CFR-5** Every key event (search, view, save, lead, approve/reject) emits an analytics event for KPI tracking (PRD §3.2).
