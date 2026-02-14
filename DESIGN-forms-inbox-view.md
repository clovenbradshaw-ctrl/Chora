# Forms Inbox View — Design Document

## Problem

The current Schema Builder treats forms as isolated objects. The "Form Library" is a small modal listing saved forms with version numbers — there's no spatial sense of *where* a form sits relative to other forms, which organization authored it, what project it belongs to, or how versions relate over time. Users need to see **the forest and the tree simultaneously**: navigate a structured collection of forms while working on one in focused detail.

---

## Design Principles

1. **Inbox metaphor, not file-browser metaphor** — Forms arrive from networks, orgs, and collaborators. They're closer to messages than static files. Show recency, unread/changed status, and provenance.
2. **Context over chrome** — The active form gets most of the screen. The sidebar reveals *just enough* structure to orient you without stealing focus.
3. **Propagation is visible** — Forms flow down the hierarchy (Network → Org → Provider → Client). The inbox should make the *source* and *direction* of each form legible at a glance.
4. **Versions are a timeline, not a list** — Show version progression as a compact visual timeline, not a flat table.

---

## Layout: Three-Zone Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│  TOOLBAR (top bar — breadcrumb + actions)                               │
├──────────────┬──────────────────────────────────────────┬───────────────┤
│              │                                          │               │
│   SIDEBAR    │          MAIN CANVAS                     │  CONTEXT      │
│   (forms     │          (active form —                  │  PANEL        │
│    inbox)    │           Schema Builder                 │  (relations,  │
│              │           as it exists)                  │   lineage,    │
│   280px      │                                          │   versions)   │
│   fixed      │          flex: 1                         │               │
│              │                                          │   260px       │
│              │                                          │   collapsible │
│              │                                          │               │
└──────────────┴──────────────────────────────────────────┴───────────────┘
```

### Zone 1: Sidebar — Forms Inbox (left, 280px)

The sidebar replaces the current modal-based "Form Library." It is always visible when in Schema view, providing persistent navigation.

#### Structure

```
┌─────────────────────────┐
│ FORMS              [+]  │  ← header + new form button
├─────────────────────────┤
│ 🔍 Search / filter...   │  ← search bar
├─────────────────────────┤
│ ▸ INBOX (3)             │  ← incoming forms from network/org
│   ┌───────────────────┐ │
│   │ ● Status & Engag… │ │  ← dot = unreviewed change
│   │   Network · v2    │ │     source + version
│   │   normative       │ │     maturity badge
│   ├───────────────────┤ │
│   │ ● Intake Event    │ │
│   │   Network · v1.1  │ │
│   │   trial  ▲updated │ │  ← "updated" flag when newer
│   ├───────────────────┤ │     than local copy
│   │   Context Form    │ │
│   │   Network · v1    │ │
│   │   draft           │ │
│   └───────────────────┘ │
├─────────────────────────┤
│ ▸ MY FORMS (2)          │  ← locally authored
│   ┌───────────────────┐ │
│   │ ■ Housing Assess… │ │  ← filled square = active/editing
│   │   Local · v3      │ │
│   │   trial           │ │
│   ├───────────────────┤ │
│   │   Risk Screen     │ │
│   │   Local · v1      │ │
│   │   draft           │ │
│   └───────────────────┘ │
├─────────────────────────┤
│ ▸ ORG FORMS (4)         │  ← forms from my organization
│   (collapsed by default)│
├─────────────────────────┤
│ ▸ NETWORK COMMONS (8)   │  ← full schema commons catalog
│   (collapsed by default)│
└─────────────────────────┘
```

#### Grouping Rules

| Group | Source | What goes here |
|-------|--------|----------------|
| **INBOX** | Network/Org push | Forms with `propagation: required\|standard` that have been updated since last viewed. Auto-clears when opened. |
| **MY FORMS** | Local saves | Forms saved via the current "Save" button. These are the user's working copies. |
| **ORG FORMS** | Org room state | Forms published by the user's organization (from `io.khora.schema.form` events in org room). |
| **NETWORK COMMONS** | Network room state | All forms in the network schema room. Read-only unless user has network admin role. |

#### Item States

Each form row shows:
- **Dot indicator** (left edge): `●` unread/updated, `■` currently active, `○` read/unchanged, none for stale
- **Name** (truncated with ellipsis)
- **Source tag**: `Network` / `Org` / `Local` — color-coded (teal/blue/muted)
- **Version**: `v{n}` in mono font
- **Maturity badge**: `draft` / `trial` / `normative` / `deprecated` — same badges as current UI
- **Update flag** (conditional): `▲ updated` in gold when network/org version > local version

#### Interactions

- **Click** → loads form into Main Canvas (Schema Builder)
- **Right-click / long-press** → context menu: Duplicate, Delete, Export, Compare with...
- **Drag** → reorder within "My Forms" group
- **Collapse/expand** group headers with `▸` / `▾`
- **Search** filters across all groups by name, key, or field content

---

### Zone 2: Main Canvas (center, flex)

This is the **existing Schema Builder** (`FormBuilder` component) — compose/wire/preview modes. No structural change needed here. The only modification is:

- Remove the floating "Library" button from the toolbar (sidebar replaces it)
- Keep Save, Version Bump, History, and mode toggle buttons
- The form loaded from the sidebar populates this view
- Add a subtle breadcrumb at top: `My Forms / Housing Assessment / v3`

---

### Zone 3: Context Panel (right, 260px, collapsible)

This panel shows **relational context** for the currently active form. It answers: "Where does this form sit in the bigger picture?"

#### Sections

```
┌─────────────────────────┐
│ CONTEXT           [◀]   │  ← collapse toggle
├─────────────────────────┤
│                         │
│ LINEAGE                 │
│ ┌─────────────────────┐ │
│ │ Network: CoC Net    │ │  ← which network defined it
│ │ ↓ required          │ │     propagation level
│ │ Org: Harbor House   │ │  ← which org adopted it
│ │ ↓ extended (+2 fld) │ │     local extensions noted
│ │ Provider: You       │ │  ← current user's copy
│ │ ↓ active            │ │
│ │ → 12 clients        │ │  ← how many clients have it
│ └─────────────────────┘ │
│                         │
├─────────────────────────┤
│                         │
│ VERSION TIMELINE        │
│                         │
│  v1 ──○── Mar 2025     │  ← dot timeline, vertical
│         "Initial"       │     notes shown inline
│  v2 ──○── Jun 2025     │
│         "+housing q's"  │
│  v3 ──●── Feb 2026     │  ← filled dot = current
│         "Risk section"  │
│                         │
│  [Compare versions]     │  ← opens diff modal
│                         │
├─────────────────────────┤
│                         │
│ RELATED FORMS           │
│ ┌─────────────────────┐ │
│ │ Intake Event        │ │  ← shares fields or
│ │  2 shared fields    │ │     crosswalks with this form
│ │  1 crosswalk        │ │
│ ├─────────────────────┤ │
│ │ Additional Context  │ │
│ │  1 shared field     │ │
│ └─────────────────────┘ │
│                         │
├─────────────────────────┤
│                         │
│ USED BY ORGS            │
│ ┌─────────────────────┐ │
│ │ Harbor House (you)  │ │  ← orgs in the network
│ │ PATH Services       │ │     using this form
│ │ Salvation Army      │ │
│ │ 3 of 5 adopted      │ │  ← adoption count
│ └─────────────────────┘ │
│                         │
├─────────────────────────┤
│                         │
│ ACTIVITY                │
│  Feb 14 — You edited   │  ← recent EO operations
│  Feb 12 — v3 bumped    │     on this form
│  Jan 30 — Network sync │
│                         │
└─────────────────────────┘
```

#### Context Panel Collapse Behavior

- Default: **open** on desktop (>1100px), **closed** on narrow screens
- Toggle button `[◀]` / `[▶]` in panel header
- When collapsed, a thin 36px strip with icons remains visible for quick re-open

---

## Data Model Extensions

### New State Required

```javascript
// Per-form metadata (extend existing savedForms array items)
{
  id,
  name,
  key,
  version,
  maturity,
  savedAt,
  form,             // existing — full form snapshot
  frameworks,       // existing
  bindings,         // existing
  crosswalks,       // existing

  // ── NEW ──
  sourceType,       // 'local' | 'org' | 'network'
  sourceId,         // org room ID or network room ID (null for local)
  sourceName,       // display name of source
  propagation,      // 'required' | 'standard' | 'recommended' | 'optional'
  lastViewedAt,     // timestamp — for unread detection
  lastSyncedVersion,// version number last synced from upstream
  linkedFormKeys,   // [key] — form keys that share fields or have crosswalks
  adoptedByOrgs,    // [{orgId, orgName, localVersion, extensions}]
  clientCount,      // number of clients with this form active
  activityLog,      // [{ts, action, actor, detail}] — recent EO ops
}
```

### Derived Computations

```
isUnread(form)     = form.version > form.lastSyncedVersion
                     || form.savedAt > form.lastViewedAt

relatedForms(form) = allForms.filter(f =>
                       f.key !== form.key &&
                       (sharedFields(f, form).length > 0
                        || hasCrosswalk(f, form)))

sharedFields(a, b) = intersection of field keys between two forms

hasCrosswalk(a, b) = crosswalks.some(xw =>
                       xw.from === a.key && xw.to === b.key
                       || xw.from === b.key && xw.to === a.key)
```

---

## Interaction Flows

### Flow 1: Open Schema View (first load)

1. Sidebar loads with groups populated:
   - **INBOX**: forms from network/org with `savedAt > lastViewedAt`
   - **MY FORMS**: locally saved forms from IndexedDB
   - **ORG FORMS**: from org room state events
   - **NETWORK COMMONS**: from network room state events
2. If there's a form with unread updates, it auto-selects in the inbox.
3. Otherwise, most recently edited local form loads.
4. Context panel populates with selected form's lineage.

### Flow 2: Network pushes an updated form

1. New version of "Status & Engagement" arrives via Matrix state event.
2. INBOX group count increments, form shows `▲ updated` badge.
3. User clicks → form loads in canvas.
4. Context panel shows version timeline with new version highlighted.
5. If breaking change (major version bump), answer crosswalk modal triggers.
6. User reviews, optionally extends, saves to "My Forms."
7. INBOX clears the `●` indicator.

### Flow 3: Creating and linking forms

1. User clicks `[+]` in sidebar header → new blank form in "My Forms."
2. User builds form in Schema Builder as today.
3. After adding fields, Context Panel auto-detects shared field keys with existing forms.
4. "RELATED FORMS" section populates showing overlap.
5. User can click a related form to open a split-diff view.

### Flow 4: Comparing versions

1. User clicks a version dot in the Version Timeline.
2. Side-by-side diff opens (reuses existing `diffFormVersions` logic).
3. Changes highlighted: green = added, red = removed, gold = renamed.
4. "Restore this version" button available for rollback.

---

## Visual Design Notes

### Color Language

| Element | Color | Meaning |
|---------|-------|---------|
| Teal (`--teal`) | Network source | Form comes from network commons |
| Blue (`--blue`) | Org source | Form comes from organization |
| Muted (`--tx-2`) | Local source | Form is locally authored |
| Gold (`--gold`) | Update/attention | New version available, action needed |
| Green (`--green`) | Normative/stable | Maturity = normative |
| Purple (`--purple`) | Classification | MEANT-side interpretation link |
| Red (`--red`) | Deprecated/breaking | Deprecated maturity or major version break |

### Typography

- **Sidebar form names**: 12.5px, Manrope 600, `--tx-0`
- **Sidebar metadata**: 10px, IBM Plex Mono 400, `--tx-2`
- **Context panel headers**: 10px, IBM Plex Mono 600, `--tx-3`, letter-spacing 0.08em (matches existing `section-label` class)
- **Version timeline dates**: 10px, IBM Plex Mono 400, `--tx-2`
- **Breadcrumb**: 11px, Manrope 400, `--tx-2`, separator `/` in `--tx-3`

### Responsive Behavior

| Breakpoint | Sidebar | Context Panel | Canvas |
|------------|---------|---------------|--------|
| >1100px | 280px visible | 260px visible | flex |
| 900–1100px | 260px visible | collapsed (36px strip) | flex |
| <900px | overlay drawer (hamburger toggle) | hidden (accessible via tab) | full width |

### Animation

- Sidebar group expand/collapse: 150ms ease-out height transition
- Form selection: 100ms background-color fade
- Context panel slide: 200ms ease-out transform
- Unread dot pulse: 2s infinite subtle opacity pulse (0.6 → 1.0)

---

## Wireframe: Full View

```
┌──────────────────────────────────────────────────────────────────────────────────┐
│  Schema Builder    My Forms / Housing Assessment / v3              [Save] [⋮]   │
├──────────┬───────────────────────────────────────────────────────┬───────────────┤
│ FORMS [+]│  Housing Assessment                                  │ CONTEXT    [◀]│
│ ┄┄┄┄┄┄┄┄ │  trial · v3 · 8 questions · 24 options · 2 fw       │               │
│ 🔍 search│  ┌─── Compose ─── Wire ─── Preview ───┐             │ LINEAGE       │
│          │  │                                      │             │ CoC Network   │
│ ▾ INBOX 3│  │  ▾ General                           │             │ ↓ required    │
│ ● Status…│  │    ┌──────────────────────────┐      │             │ Harbor House  │
│   Net v2 │  │    │ What is your current     │      │             │ ↓ +2 fields   │
│   normat…│  │    │ housing situation?       │      │             │ You (active)  │
│ ● Intake…│  │    │ ☐ Sheltered              │      │             │ → 12 clients  │
│   Net v1…│  │    │ ☐ Unsheltered            │      │             │               │
│   trial ▲│  │    │ ☐ At risk                │      │             │ VERSION       │
│   Context│  │    │ ☐ Stably housed          │      │             │ v1 ─○─ Mar 25 │
│   Net v1 │  │    └──────────────────────────┘      │             │ v2 ─○─ Jun 25 │
│          │  │                                      │             │ v3 ─●─ Feb 26 │
│ ▾ MY   2 │  │    ┌──────────────────────────┐      │             │               │
│ ■ Housing│  │    │ How long in current      │      │             │ RELATED       │
│   Loc v3 │  │    │ situation?               │      │             │ Intake Event  │
│   trial  │  │    │ ☐ <1 month               │      │             │  2 shared fld │
│   Risk Sc│  │    │ ☐ 1-6 months             │      │             │ Add'l Context │
│   Loc v1 │  │    │ ☐ 6-12 months            │      │             │  1 shared fld │
│          │  │    │ ☐ >1 year                │      │             │               │
│ ▸ ORG  4 │  │    └──────────────────────────┘      │             │ USED BY ORGS  │
│ ▸ NET  8 │  │                                      │             │ Harbor House  │
│          │  │  ▸ Risk Assessment                    │             │ PATH Services │
│          │  │  ▸ Demographics                       │             │ 2 of 5 adopted│
│          │  │                                      │             │               │
│          │  └──────────────────────────────────────┘             │ ACTIVITY      │
│          │                                                       │ Feb 14 edited │
│          │                                                       │ Feb 12 v3 bump│
└──────────┴───────────────────────────────────────────────────────┴───────────────┘
```

---

## Mapping to Existing Code

| Current Code | Change Needed |
|---|---|
| `FormBuilder` component (line 3416) | Wrap in new `SchemaView` layout container with sidebar + context panel |
| `savedForms` state (line 3431) | Extend with `sourceType`, `lastViewedAt`, `linkedFormKeys`, etc. |
| `showFormList` modal (line 4536-4593) | **Replace entirely** with sidebar component |
| `view==='schema'` render (line 8281) | Render `SchemaView` instead of bare `FormBuilder` |
| `DEFAULT_FORMS` (domain config) | Feed into NETWORK COMMONS group |
| Org room `io.khora.schema.form` events | Feed into ORG FORMS group |
| `diffFormVersions` function (line 3383) | Reuse for version comparison in context panel |
| `doSaveForm` / `doLoadForm` | Keep, but trigger sidebar refresh on save |
| Version history modal (line 4640) | Keep as modal, but also show compact timeline in context panel |

---

## Component Breakdown (Implementation Plan)

### New Components

1. **`SchemaView`** — Layout wrapper. Renders Sidebar + FormBuilder + ContextPanel in the three-zone grid.
2. **`FormsSidebar`** — Left panel. Manages groups (INBOX, MY FORMS, ORG, NETWORK), search, selection.
3. **`FormListItem`** — Individual form row in sidebar. Shows name, source, version, status indicators.
4. **`ContextPanel`** — Right panel. Sections: Lineage, Version Timeline, Related Forms, Used By, Activity.
5. **`VersionTimeline`** — Vertical dot-timeline of form versions. Clickable dots open diff.
6. **`LineageTree`** — Network → Org → Provider → Client chain visualization.
7. **`FormBreadcrumb`** — Top bar breadcrumb showing group / form name / version.

### Modified Components

1. **`FormBuilder`** — Remove Library button, accept `form` and `onFormChange` as props instead of owning state.
2. **`ProviderApp`** — `view==='schema'` renders `SchemaView` instead of `FormBuilder`.

---

## Open Questions

1. **Multi-form editing** — Should users be able to have multiple forms "open" in tabs within the canvas, or strictly one at a time? (Recommendation: one at a time, with fast switching via sidebar click.)
2. **Org form publishing** — When a user saves a form and they're an org admin, should there be an explicit "Publish to Org" action? (Recommendation: yes, separate from local save.)
3. **Network proposal flow** — Should proposing a form to the network schema happen from this view, or remain in the Network governance view? (Recommendation: add a "Propose to Network" action in the context menu, which opens the governance proposal flow.)
4. **Offline forms** — Since the app uses IndexedDB, should the sidebar show a connectivity indicator for forms that haven't synced? (Recommendation: yes, subtle icon.)
