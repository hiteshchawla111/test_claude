# User Stories — Property Listing Portal

**Status:** Draft v1.0 · **Last updated:** 2026-06-05
**Related:** [PRD](./PRD.md) · [User Roles](./User-Roles.md) · [Functional Requirements](./Functional-Requirements.md) · [MVP Scope](./MVP-Scope.md)

---

## Conventions

- Format: *As a [role], I want [goal] so that [benefit].*
- **Priority (MoSCoW + P-tag):** P0 = Must (MVP) · P1 = Should (Phase 2) · P2 = Could (Phase 3+).
- **SP** = story points (Fibonacci estimate, relative).
- Acceptance criteria use Given/When/Then where useful.
- Epics map to FR groups in [Functional Requirements](./Functional-Requirements.md).

---

## Epic A — Accounts & Authentication (FR-1)

### A-1 · Sign up (Seeker) — P0 · SP 3
As a **guest**, I want to create an account with email and password so that I can save listings and contact listers.
**Acceptance:**
- Given valid email/password, when I submit, then an account is created and a verification email is sent.
- Email must be verified before I can contact listers or create listings.
- Duplicate email is rejected with a clear message.
- Password meets policy (min length, complexity); errors are accessible (announced to screen readers).

### A-2 · Login & session — P0 · SP 2
As a **registered user**, I want to log in securely so that I can access my account.
**Acceptance:** Valid credentials log in; invalid show generic error (no user enumeration); session expires per policy; "remember me" optional; rate-limited against brute force.

### A-3 · Password reset — P0 · SP 2
As a **registered user**, I want to reset my password via email so that I can recover access.
**Acceptance:** Reset link is single-use, time-limited; old sessions optionally invalidated; no user enumeration on the request form.

### A-4 · Lister onboarding + phone OTP — P0 · SP 3
As a **seeker**, I want to become a lister by verifying my phone so that I can post listings and reduce fraud.
**Acceptance:** OTP sent to phone; verified phone unlocks listing creation; failed/expired OTP handled; rate-limited.

### A-5 · Social/SSO login — P1 · SP 3
As a **guest**, I want to sign in with Google/Apple so that I can onboard faster.
**Acceptance:** OAuth flow links/creates account; existing email is merged safely; no duplicate accounts.

---

## Epic B — Listing Management (FR-2)

### B-1 · Create listing — P0 · SP 8
As a **lister**, I want to create a listing with all key attributes so that seekers can find my property.
**Acceptance:**
- Required fields validated (type sale/rent, price, property type, beds/baths, area, location, ≥1 photo).
- Address geocoded to lat/long; map pin adjustable.
- Listing saved as draft, then submitted to review.
- Validation errors are inline and accessible.

### B-2 · Upload & manage photos — P0 · SP 5
As a **lister**, I want to upload, reorder, and set a cover photo so that my listing looks attractive.
**Acceptance:** Multi-upload with progress; images resized/transcoded server-side; reorder + cover selection; max count + size enforced; non-image/oversized rejected with message; EXIF/location stripped for privacy.

### B-3 · Edit / unpublish / delete own listing — P0 · SP 3
As a **lister**, I want to edit, unpublish, or delete my own listing so that I keep it accurate.
**Acceptance:** Only owner (or admin) can modify; edits to material fields may re-trigger review; unpublish hides from search immediately; delete is soft-delete with audit.

### B-4 · Listing lifecycle & expiry — P0 · SP 3
As a **lister**, I want my listing to expire and be refreshable so that stale listings don't mislead seekers.
**Acceptance:** Live listing expires after configured period; lister notified before expiry; one-click refresh resets timer; expired listings drop out of search.

### B-5 · Listing analytics — P1 · SP 5
As an **agent**, I want to see views and leads per listing so that I can measure performance.
**Acceptance:** Per-listing views, unique viewers, saves, leads over time; respects privacy; loads < 1s for typical agent volume.

### B-6 · Bulk management — P1 · SP 8
As an **agent**, I want to manage many listings (bulk edit/refresh/archive) so that I save time.
**Acceptance:** Multi-select actions; guardrails against accidental mass changes; audit recorded.

---

## Epic C — Search & Discovery (FR-3)

### C-1 · Structured/keyword search — P0 · SP 8
As a **seeker**, I want to search by location, price, type, beds, baths, and amenities so that I find relevant properties.
**Acceptance:**
- Filters combine (AND) and reflect in URL (shareable, SSR).
- Empty state and "no results" with suggestions.
- Results p75 < 800 ms (see [NFRs](./Non-Functional-Requirements.md)).

### C-2 · Map-based search — P0 · SP 13
As a **seeker**, I want to search on a map with clustering and pan/zoom so that I explore by area.
**Acceptance:**
- Map shows clustered pins; clusters expand on zoom.
- Panning updates results to the visible bounding box ("search this area").
- List and map stay in sync; selecting a pin highlights the list item and vice versa.
- Performance stable at high pin counts (viewport + clustering, not all-points).

### C-3 · Sort & paginate — P0 · SP 3
As a **seeker**, I want to sort by relevance/price/newest/distance and page through results so that I scan efficiently.
**Acceptance:** Sort persists in URL; pagination or infinite scroll with stable ordering; accessible controls.

### C-4 · Listing comparison — P1 · SP 5
As a **buyer**, I want to compare selected listings side-by-side so that I decide faster.
**Acceptance:** Select up to N listings; comparison table of key attributes; remove/clear.

---

## Epic D — Listing Detail (FR-4)

### D-1 · View listing detail — P0 · SP 5
As a **seeker**, I want a rich detail page with gallery, attributes, map, and amenities so that I evaluate the property.
**Acceptance:** Gallery with keyboard/touch nav and alt text; map pin (approx/exact per setting); SSR for SEO; LCP mobile p75 < 2.5s; structured data (schema.org) present.

### D-2 · Report a listing — P0 · SP 2
As a **seeker**, I want to report a suspicious/fraudulent listing so that the platform stays trustworthy.
**Acceptance:** Report with reason; confirmation shown; report enters moderation queue; abuse of reporting is rate-limited.

---

## Epic E — Contact & Leads (FR-5)

### E-1 · Privacy-preserving inquiry — P0 · SP 5
As a **seeker**, I want to contact a lister without exposing my personal data prematurely so that I feel safe.
**Acceptance:**
- Inquiry form (message + preferred contact) sent without revealing lister's raw phone/email in page HTML/API.
- Captcha + rate limiting prevent spam.
- Lister receives lead; seeker gets confirmation.

### E-2 · Lead inbox — P0 · SP 5
As a **lister**, I want a lead inbox so that I can see and respond to inquiries.
**Acceptance:** Leads listed with listing context, timestamp, seeker message; mark read/responded; email notification on new lead; spam leads can be reported/blocked.

### E-3 · Reveal contact per consent — P0 · SP 2
As a **seeker**, I want to reveal a lister's contact when they allow it so that I can call/email directly.
**Acceptance:** Reveal action requires login; logged event; respects lister's per-listing setting; masked until revealed.

### E-4 · Real-time chat threads — P1 · SP 8
As a **seeker and lister**, I want a threaded in-app conversation so that we communicate in one place.
**Acceptance:** Persistent threads tied to a listing; read receipts optional; notifications; spam/block controls.

---

## Epic F — Engagement: Favorites & Saved Searches (FR-6)

### F-1 · Favorite listings — P0 · SP 3
As a **seeker**, I want to save listings so that I can revisit them.
**Acceptance:** Toggle favorite from card/detail; favorites page; persists across sessions; works only when logged in (guest prompted to sign in).

### F-2 · Saved searches + email alerts — P0 · SP 5
As a **seeker**, I want to save a search and get alerts for new matches so that I act fast on new listings.
**Acceptance:** Save current filter set; choose alert frequency; email sent when new matching listings appear; unsubscribe link; manage/delete saved searches.

### F-3 · SMS/push alerts — P1 · SP 5
As a **seeker**, I want SMS/push alerts so that I'm notified instantly.
**Acceptance:** Opt-in channels; respects quiet hours; compliant unsubscribe.

---

## Epic G — Trust, Safety & Moderation (FR-7)

### G-1 · Moderation queue — P0 · SP 8
As an **admin**, I want a queue of pending and flagged listings so that I approve/reject quickly.
**Acceptance:** Queue sortable/filterable by status, age, signal severity; approve/reject with reason; SLA timer; actions audited; rejection notifies lister with reason.

### G-2 · Duplicate/fraud signals — P0 · SP 5
As an **admin**, I want automatic signals (duplicate address/phone/image hash, suspicious pricing) so that I prioritize risky listings.
**Acceptance:** Signals surfaced on the listing in queue; high-severity auto-holds publishing; tunable thresholds; false positives can be cleared.

### G-3 · Suspend/ban user — P0 · SP 3
As an **admin**, I want to suspend/ban abusive users so that the marketplace stays safe.
**Acceptance:** Suspension hides their listings and blocks contact; reason + duration recorded; user notified; reversible; audited.

### G-4 · Audit log — P0 · SP 3
As an **admin**, I want an audit log of admin actions so that we have accountability.
**Acceptance:** Immutable record of who/what/when/why; searchable; read-only for admins.

### G-5 · Verified-agent KYC/badge — P1 · SP 8
As an **agent**, I want a verified badge so that seekers trust my listings.
**Acceptance:** Document/identity check workflow; badge shown on profile/listings; revocable on violation.

---

## Epic H — Notifications & Platform (FR-9)

### H-1 · Transactional & alert emails — P0 · SP 3
As a **user**, I want timely emails (verification, leads, alerts) so that I stay informed.
**Acceptance:** Templated, localized-ready, with unsubscribe where applicable; deliverability monitored; rate-limited.

### H-2 · Notification preferences — P1 · SP 3
As a **user**, I want to control which notifications I receive so that I'm not overwhelmed.
**Acceptance:** Per-channel, per-type toggles; honored across the system.

---

## Epic I — Monetization (FR-10) — Phase 3

### I-1 · Featured/boost listing — P2 · SP 8
As an **agent**, I want to boost a listing so that it gets more visibility.
**Acceptance:** Paid placement clearly labeled; ranking boost bounded to preserve relevance/trust; billing recorded.

### I-2 · Agent subscription tiers — P2 · SP 8
As an **agent**, I want a subscription with higher limits/analytics so that I scale.
**Acceptance:** Tiered limits enforced; upgrade/downgrade; billing + invoices.

---

## Traceability Summary

| Epic | FR Group | Phase |
|---|---|---|
| A Accounts/Auth | FR-1 | MVP (A-5 P2 → Phase 2) |
| B Listings | FR-2 | MVP (B-5/B-6 → Phase 2) |
| C Search | FR-3 | MVP (C-4 → Phase 2) |
| D Listing detail | FR-4 | MVP |
| E Contact/Leads | FR-5 | MVP (E-4 → Phase 2) |
| F Engagement | FR-6 | MVP (F-3 → Phase 2) |
| G Trust/Safety | FR-7 | MVP (G-5 → Phase 2) |
| H Notifications | FR-9 | MVP (H-2 → Phase 2) |
| I Monetization | FR-10 | Phase 3 |
