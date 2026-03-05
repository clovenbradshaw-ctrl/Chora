# Khora: Goals, Committed Functionality, and Long-Standing Issues

## Overall Goal

**Khora is an encryption-first, federated case management system built on the Matrix protocol that puts client data sovereignty at the center of social services.**

Instead of a traditional centralized CRM where agencies own and control all client data, Khora flips the model: the **individual** (client) owns their data in an encrypted "vault," decides which providers can see which fields, and can revoke access at any time. All data is encrypted at rest and in transit with no plaintext secrets stored on any server.

**Core principles:**
- **Client data sovereignty** — individuals control their own records
- **Field-level encryption** — each provider gets a unique AES-256-GCM key; sharing is granular per-field
- **Federated architecture** — built entirely on Matrix (open protocol), no traditional backend
- **Epistemic integrity** — the GIVEN/FRAMEWORK/MEANT model separates raw observations from interpretive frameworks from institutional classifications, all traceable
- **Zero-trust hosting** — single static HTML file on GitHub Pages, all logic runs in the browser

---

## Committed Functionality

### Already Implemented

1. **Four Actor Types** — Client, Provider, Organization Admin, Network Coordinator (detected at login from Matrix room membership)
2. **Client Vault** — 11 built-in fields across 5 categories (Identity, Contact, Details, Case, Sensitive), plus custom fields; all encrypted per-field
3. **Bridge System** — Client↔Provider connections with field-level sharing, messaging, soft/hard revocation
4. **7 Room Types** — Vault, Bridge, Organization, Provider Roster, Metrics, Network, Schema rooms — all as Matrix rooms with `io.khora.*` state events
5. **Epistemic Operations (EO)** — 9 operators in 3 triads (Identity/Structure/Interpretation) with full provenance tracing and state projection
6. **Activity Stream** — Visualization of EO events in Stream, Table, and Triad views with filtering
7. **Schema System** — GIVEN/FRAMEWORK/MEANT architecture with Forms, Authorities, Definitions, Bindings, Transforms; propagation levels (Required/Standard/Recommended/Optional)
8. **3-Layer Encryption** — Megolm E2EE (room-level) + per-field AES-256-GCM + IndexedDB encryption at rest
9. **Metrics & Anonymization** — Age bucketing, geohashing, PII blocking, cohort hashing via SHA-256
10. **Data Import** — CSV/JSON bulk import with auto-column mapping, per-record encryption, n8n webhook integration
11. **Governance System** — Consent-based decision-making with `adopt_as_is`, `adopt_with_extension`, `needs_modification`, `cannot_adopt` (veto) positions
12. **Vault Backup/Restore** — Encrypted backup and restore of vault data
13. **Provider Dashboard** — Case list, CRM database view, messaging, staff UI

### Designed but Not Yet Implemented

14. **Resource Tracking** — Resource allocation, inventory tracking, consumption events at individual/org/network levels (full design in `DESIGN-resource-tracking.md`)
15. **Schema Builder & Notification UX** — Spatial hierarchy form library, inline change indicators, timeline versioning (design in `DESIGN-forms-inbox-view.md`)

---

## Longest-Standing Issues

### Tier 1: Architectural Gaps (From Project Inception)

These are documented in `ARCHITECTURE_GAP_ANALYSIS.md` and have been unresolved since the project's earliest days:

**Gap 1: Organization Entity** — CRITICAL, BLOCKS GAPS 2-7
- Khora conflates "provider" (individual Matrix user) with "organization" (institutional entity)
- No separate org room type, no staff roster, no org creation flow
- Without this, the system cannot scale to multiple staff at a single organization
- This is the foundational gap — everything else depends on it

**Gap 2: Staff ↔ Org Lifecycle** — HIGH PRIORITY
- No mechanism for staff to join/leave organizations
- No case assignment, no role-based field access
- When a provider leaves, they keep access to all cases

**Gap 3: Org ↔ Network Membership** — MEDIUM PRIORITY
- Network members are individual user IDs, not organizations
- Schema propagation levels are displayed but never enforced in code
- Organizations can ignore network-mandated schema changes

**Gap 4: Schema Versioning & Propagation Enforcement** — MEDIUM PRIORITY
- No version tracking on schema prompts
- Observations don't reference which schema version they were collected under
- "Required" prompts can be modified at org level

**Gap 5: Client ↔ Provider Discovery** — HIGH USABILITY IMPACT
- Clients must enter raw Matrix IDs to connect with providers
- No QR codes, no provider directory, no referral links
- Non-functional for real-world field use

**Gap 6: Provider-Side Structured Observations** — HIGH VALUE
- Providers can only record freeform notes, not structured assessments
- VI-SPDAT, PHQ-9, housing placement assessments should be prompt-based but aren't
- Data not comparable across providers

**Gap 7: Cross-Org Deduplication** — CRITICAL AT SCALE
- Cohort hashing infrastructure exists but blind matching not implemented
- No way to detect when two orgs serve the same person without sharing plaintext PII

### Tier 2: Recurring Operational Bugs (Most Frequent in Git History)

**Data Persistence & Synchronization** — ~13% of recent commits are fixes for this
- Field edits not persisting to Matrix; data not rendering after refresh
- Stale closures and cache sync failures
- Root cause: React state/cache not syncing with Matrix async operations

**Encryption & Decryption Errors** — ~5 fixes in last 60 commits
- E2EE failures, OLM.BAD_MESSAGE_MAC errors, device key mismatches
- Device crypto state not persisting across sessions

**Rate Limiting (429 Errors)** — Persistent Matrix API issue
- Room creation floods during team/individual initialization
- No systematic request throttling or backoff

**White Screen / Crash Bugs** — ~5 fixes in last 60 commits
- Navigation between views triggers unhandled exceptions
- Stale state references, missing hook dependencies

### Tier 3: Missing Infrastructure

- **Zero automated test coverage** — crypto layer, EO state machine, role inference all untested
- **No offline capability** — no service worker, no IndexedDB cache, no offline queue
- **No audit log view** — EO operations provide implicit audit trail but no dedicated UI
