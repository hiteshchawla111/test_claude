# User Roles & Permissions — Property Listing Portal

**Status:** Draft v1.0 · **Last updated:** 2026-06-05
**Related:** [PRD](./PRD.md) · [User Stories](./User-Stories.md) · [Functional Requirements](./Functional-Requirements.md)

---

## 1. Roles Overview

| Role | Description | Auth required | Personas |
|---|---|---|---|
| **Guest / Visitor** | Unauthenticated user. Can browse and search publicly. | No | (pre-signup Riya/Bhavin) |
| **Seeker** (Buyer/Renter) | Registered user looking for properties. | Yes | Renter Riya, Buyer Bhavin |
| **Lister — Owner** | Individual property owner listing 1–few properties. | Yes (+ phone OTP) | Owner Omar |
| **Lister — Agent** | Professional agent/broker managing many listings. | Yes (+ phone OTP; verifiable badge) | Agent Anita |
| **Admin / Moderator** | Internal trust & safety / ops staff. | Yes (elevated) | Admin Aarav |
| **Super Admin** | Internal platform owner; manages admins, config, billing (later). | Yes (elevated, restricted) | Internal |

**Notes**
- A single account can hold **both Seeker and Lister capabilities** (a user can search and also list). Role = set of granted capabilities, not mutually exclusive for seeker+lister.
- **Owner** vs **Agent** differ mainly by listing volume limits, badge/verification, and (later) subscription tier — same base lister permissions.
- Admin and Super Admin are internal-only and provisioned, not self-serve.

---

## 2. Capability Definitions

- **Browse/Search:** view search results, map, listing detail (public fields only).
- **Save/Favorite & Saved Searches:** persist favorites and search alerts (requires account).
- **Contact lister:** send inquiry/lead via privacy-preserving flow.
- **Create/Manage listing:** CRUD on own listings, media, lifecycle.
- **View lead inbox & listing analytics:** see inquiries and performance for own listings.
- **Report content:** flag listings/users.
- **Moderate:** review queue, approve/reject, take down, suspend users.
- **Admin manage:** user management, config, audit logs, reporting.

---

## 3. Permissions Matrix

Legend: ✅ allowed · ⚠️ conditional/own-only · ❌ not allowed

| Capability | Guest | Seeker | Lister (Owner) | Lister (Agent) | Admin | Super Admin |
|---|:--:|:--:|:--:|:--:|:--:|:--:|
| Browse & search listings | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| View listing detail (public fields) | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| View masked lister contact | ✅* | ✅ | ✅ | ✅ | ✅ | ✅ |
| Reveal full contact (per lister setting) | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Save / favorite listings | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Create & manage saved searches + alerts | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Contact lister / send inquiry | ❌† | ✅ | ✅ | ✅ | ✅ | ✅ |
| Create listing | ❌ | ❌‡ | ✅ | ✅ | ⚠️ (on behalf) | ✅ |
| Edit / delete own listing | ❌ | ❌ | ⚠️ own | ⚠️ own | ✅ any | ✅ any |
| Upload / manage listing media | ❌ | ❌ | ⚠️ own | ⚠️ own | ✅ any | ✅ any |
| Publish / unpublish own listing | ❌ | ❌ | ⚠️ own | ⚠️ own | ✅ any | ✅ any |
| View own lead inbox | ❌ | ❌ | ⚠️ own | ⚠️ own | ✅ any | ✅ any |
| View own listing analytics | ❌ | ❌ | ⚠️ own | ⚠️ own | ✅ any | ✅ any |
| Bulk listing management | ❌ | ❌ | ❌ (limited) | ⚠️ own (Phase 2) | ✅ | ✅ |
| Verified-agent badge | ❌ | ❌ | ❌ | ⚠️ (Phase 2 KYC) | n/a | n/a |
| Report listing / user | ✅ (captcha) | ✅ | ✅ | ✅ | ✅ | ✅ |
| Moderation queue (approve/reject) | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Take down any listing | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Suspend / ban users | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| View audit log | ❌ | ❌ | ❌ | ❌ | ✅ (read) | ✅ |
| Manage admins / roles | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| Platform config / feature flags | ❌ | ❌ | ❌ | ❌ | ⚠️ limited | ✅ |
| Reporting / dashboards | ❌ | ❌ | ⚠️ own listings | ⚠️ own listings | ✅ platform | ✅ platform |

\* Guests see only masked contact (e.g., "Sign in to contact"); no raw data.
† Guests may be allowed a single gated inquiry behind captcha + email capture (config decision; see [MVP Scope](./MVP-Scope.md) open questions) — default OFF for MVP to reduce spam.
‡ A Seeker becomes a Lister by completing lister onboarding (phone OTP); the same account is then granted lister capabilities.

---

## 4. Listing-Level Field Visibility

| Field | Public (Guest/Seeker) | Lister (owner) | Admin |
|---|:--:|:--:|:--:|
| Title, price, type, beds/baths, area, amenities, description | ✅ | ✅ | ✅ |
| Photos | ✅ | ✅ | ✅ |
| Approximate location / pin | ✅ | ✅ | ✅ |
| Exact address | ⚠️ per lister setting | ✅ | ✅ |
| Lister name + masked contact | ✅ | ✅ | ✅ |
| Lister raw phone/email | ❌ (revealed via consent flow) | ✅ | ✅ |
| Moderation status / fraud flags | ❌ | ⚠️ own status only | ✅ |
| Lead/inquiry data | ❌ | ⚠️ own | ✅ |

---

## 5. Trust States (applies to listers & listings)

- **Unverified lister:** email only → listings require pre-publish review.
- **Phone-verified lister:** email + phone OTP → standard MVP lister.
- **Trusted lister:** N approved listings + no violations → eligible for auto-publish.
- **Verified agent (Phase 2):** KYC/badge → priority trust signals, higher limits.
- **Suspended/Banned:** no listing/contact capability; existing listings hidden.

---

## 6. Open Questions
1. Allow gated guest inquiry, or require signup to contact? (Default: require signup for MVP.)
2. Listing volume caps for Owner vs Agent tiers?
3. Admin role granularity — split Moderator vs Ops vs Support now or later?
