# CMS UI Extension Install Flow — UX Specification

## Overview

Three-role flow for installing and consuming UI extensions from the Optimizely Connect Platform (OCP) into the CMS.

```
DEVELOPER                    ADMIN                         MARKETER
━━━━━━━━━                    ━━━━━                         ━━━━━━━━

Build app with               Browse OCP                    Open CMS UI
UI extensions                Marketplace                        │
     │                            │                         CMS calls OCP
Define manifest              View app detail                Discovery API
(injection points,           (extensions listed)            (background)
 permissions)                     │                              │
     │                       Install app                    Extensions
Deploy to OCP                (consent to                    auto-injected at
     │                        permissions)                  injection points
App available                     │                              │
in marketplace               Configure app                  User sees native
                             settings                       CMS experience
                                  │                              │
                             Enable extensions               User interacts
                             per CMS instance                naturally
                                  │
                             Extensions live
                             for marketers
```

---

## Design System

**OptiAxiom** — Optimizely's React component library.

Key components used throughout:
- `Sidebar` — OCP and CMS navigation
- `Tabs` — section switching within pages
- `Table` — listing apps, extensions, instances
- `Card` — app tiles in marketplace
- `Badge` — status indicators (Enabled, Disabled, New)
- `Button` — primary (blue), secondary (outline), danger (red)
- `Dialog` / `AlertDialog` — confirmation flows, install consent
- `Toast` — success/error notifications
- `Toggle` — enable/disable extensions per instance
- `Select` — CMS instance picker
- `SearchInput` — marketplace search
- `Spinner` — loading states
- `Tooltip` — contextual help
- `Field` + `Input` — configuration forms
- `DropdownMenu` — overflow actions

---

## Phase A: Developer Flow

### Screen D1: App Manifest Editor

**Path:** `ocp.optimizely.com/apps/{app-id}/manifest`
**Role:** Developer
**Purpose:** Define UI extensions and their injection points

**Layout:**
```
┌──────────────────────────────────────────────────────────┐
│ [Sidebar: OCP Dev Console]                               │
│                                                          │
│  App: {App Name}                                         │
│  ┌─[Tabs]──────────────────────────────────────────┐     │
│  │ Overview │ Extensions │ Permissions │ Deploy     │     │
│  └──────────────────────────────────────────────────┘     │
│                                                          │
│  UI Extensions                          [+ Add Extension]│
│  ─────────────────────────────────────────────────────── │
│                                                          │
│  ┌─[Card]──────────────────────────────────────────┐     │
│  │ Product Recommendations Widget                   │     │
│  │ Injection point: content-editor/sidebar-panel    │     │
│  │ Status: Configured ✓                  [Edit][⋮] │     │
│  └──────────────────────────────────────────────────┘     │
│                                                          │
│  ┌─[Card]──────────────────────────────────────────┐     │
│  │ Analytics Overlay                                │     │
│  │ Injection point: content-editor/toolbar          │     │
│  │ Status: Configured ✓                  [Edit][⋮] │     │
│  └──────────────────────────────────────────────────┘     │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**Add/Edit Extension Dialog:**
```
┌─[Dialog]─────────────────────────────────────────────┐
│  Add UI Extension                              [✕]   │
│  ─────────────────────────────────────────────────── │
│                                                      │
│  Extension name                                      │
│  [Field: "Product Recommendations Widget"       ]    │
│                                                      │
│  Description                                         │
│  [Field: "Shows AI-powered content recs..."     ]    │
│                                                      │
│  Injection point                                     │
│  [Select: ▾ Choose injection point              ]    │
│    ┌──────────────────────────────────────────┐      │
│    │ content-editor/sidebar-panel             │      │
│    │ content-editor/toolbar                   │      │
│    │ content-editor/field-extension           │      │
│    │ dashboard/widget                         │      │
│    │ navigation/menu-item                     │      │
│    └──────────────────────────────────────────┘      │
│                                                      │
│  Entry file URL                                      │
│  [Field: "https://cdn.example.com/widget.js"    ]    │
│                                                      │
│  Required permissions                                │
│  [✓] Read content        [ ] Write content           │
│  [✓] Read user profile   [ ] Manage settings         │
│                                                      │
│                          [Cancel]  [Save Extension]  │
└──────────────────────────────────────────────────────┘
```

**States:**
- **Empty:** No extensions defined. Show illustration + "Add your first UI extension" + `[+ Add Extension]` button
- **Populated:** Cards listed as above
- **Validation error:** Red border on Field, error text below ("Extension name is required", "Invalid URL format")

---

### Screen D2: Deploy

**Path:** `ocp.optimizely.com/apps/{app-id}/deploy`
**Role:** Developer
**Purpose:** Deploy app version to OCP marketplace

**Layout:**
```
┌──────────────────────────────────────────────────────────┐
│  Deploy                                                  │
│  ─────────────────────────────────────────────────────── │
│                                                          │
│  Version: [Field: "1.2.0"  ]                             │
│                                                          │
│  Release notes                                           │
│  [Textarea: "Added sidebar panel extension..."      ]    │
│                                                          │
│  This version includes:                                  │
│  • 2 UI extensions                                       │
│  • 3 permissions required                                │
│                                                          │
│  Visibility                                              │
│  (•) Public — listed in OCP Marketplace                  │
│  ( ) Private — only available via direct link             │
│                                                          │
│                              [Cancel]  [Deploy to OCP]   │
└──────────────────────────────────────────────────────────┘
```

**Post-deploy Toast:** `Toast (success): "v1.2.0 deployed successfully. App is now available in the marketplace."`

---

## Phase A: Admin Flow

### Screen A1: OCP Marketplace

**Path:** `ocp.optimizely.com/marketplace`
**Role:** Admin
**Purpose:** Browse and discover available apps

**Layout:**
```
┌──────────────────────────────────────────────────────────┐
│ [Sidebar: OCP Admin]                                     │
│                                                          │
│  Marketplace                                             │
│  ─────────────────────────────────────────────────────── │
│                                                          │
│  [SearchInput: "Search apps..."              ]           │
│                                                          │
│  [Tabs: All │ CMS Extensions │ Data │ Analytics │ ...]   │
│                                                          │
│  ┌─[Card]────────────┐  ┌─[Card]────────────┐           │
│  │ 🔌 Product Recs   │  │ 📊 Content        │           │
│  │    Pro             │  │    Analytics      │           │
│  │                    │  │                    │           │
│  │ AI-powered content │  │ Real-time content  │           │
│  │ recommendations    │  │ performance        │           │
│  │ in your editor     │  │ in your editor     │           │
│  │                    │  │                    │           │
│  │ By: Acme Inc       │  │ By: DataCo         │           │
│  │ [Badge: New]       │  │ ★ 4.5 (120)       │           │
│  │                    │  │                    │           │
│  │     [View App]     │  │     [View App]     │           │
│  └────────────────────┘  └────────────────────┘           │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**States:**
- **Loading:** `Spinner` centered in content area
- **Empty search:** "No apps match your search." + clear search link
- **Error:** "Couldn't load marketplace. Try again." + `[Retry]` button

---

### Screen A2: App Detail Page

**Path:** `ocp.optimizely.com/marketplace/apps/{app-id}`
**Role:** Admin
**Purpose:** Review what the app does before installing

**Layout:**
```
┌──────────────────────────────────────────────────────────┐
│  ← Back to Marketplace                                   │
│                                                          │
│  Product Recommendations Pro          [Install App]      │
│  By Acme Inc • v1.2.0 • ★ 4.5 (120 reviews)            │
│  ─────────────────────────────────────────────────────── │
│                                                          │
│  [Tabs: Overview │ UI Extensions │ Permissions │ Reviews]│
│                                                          │
│  ┌─ Overview ────────────────────────────────────────┐   │
│  │ AI-powered content recommendations that appear    │   │
│  │ directly in your CMS editor. Suggests related     │   │
│  │ content, optimizes internal linking, and surfaces  │   │
│  │ high-performing content to reference.              │   │
│  │                                                    │   │
│  │ Screenshots:                                       │   │
│  │ [image carousel showing extension in CMS context]  │   │
│  └────────────────────────────────────────────────────┘   │
│                                                          │
│  ┌─ UI Extensions tab ───────────────────────────────┐   │
│  │                                                    │   │
│  │ This app adds 2 UI extensions to your CMS:        │   │
│  │                                                    │   │
│  │ ┌─[Card]──────────────────────────────────────┐   │   │
│  │ │ Product Recommendations Widget               │   │   │
│  │ │ Appears in: Editor sidebar panel             │   │   │
│  │ │ [Badge: sidebar-panel]                       │   │   │
│  │ │                                              │   │   │
│  │ │ Shows content recommendations while you      │   │   │
│  │ │ edit. Analyses current content and suggests   │   │   │
│  │ │ related pages to link.                       │   │   │
│  │ └──────────────────────────────────────────────┘   │   │
│  │                                                    │   │
│  │ ┌─[Card]──────────────────────────────────────┐   │   │
│  │ │ Analytics Overlay                            │   │   │
│  │ │ Appears in: Editor toolbar                   │   │   │
│  │ │ [Badge: toolbar]                             │   │   │
│  │ │                                              │   │   │
│  │ │ Adds a performance button to the toolbar.    │   │   │
│  │ │ Click to see page views, engagement, and     │   │   │
│  │ │ conversion data for the current page.        │   │   │
│  │ └──────────────────────────────────────────────┘   │   │
│  └────────────────────────────────────────────────────┘   │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

### Screen A3: Install Consent Dialog

**Trigger:** Admin clicks `[Install App]`
**Component:** `AlertDialog`

```
┌─[AlertDialog]────────────────────────────────────────┐
│  Install Product Recommendations Pro?          [✕]   │
│  ─────────────────────────────────────────────────── │
│                                                      │
│  This app requires the following permissions:        │
│                                                      │
│  ✓  Read content — Access content in your CMS        │
│  ✓  Read user profile — See who's editing            │
│  ✓  Read analytics — Access page performance data    │
│                                                      │
│  ┌──────────────────────────────────────────────┐    │
│  │ ⓘ  This app adds UI extensions to your CMS.  │    │
│  │    After installing, you'll choose which CMS  │    │
│  │    instances to enable them on.               │    │
│  └──────────────────────────────────────────────┘    │
│                                                      │
│                     [Cancel]  [Install]               │
└──────────────────────────────────────────────────────┘
```

**Post-install Toast:** `Toast (success): "Product Recommendations Pro installed. Enable it for your CMS instances to make it available to users."`

---

### Screen A4: Installed App Management

**Path:** `ocp.optimizely.com/apps/installed/{app-id}`
**Role:** Admin
**Purpose:** Configure app and enable extensions per CMS instance

**Layout:**
```
┌──────────────────────────────────────────────────────────┐
│  ← Installed Apps                                        │
│                                                          │
│  Product Recommendations Pro                             │
│  v1.2.0 • Installed 2 days ago        [⋮ Uninstall]     │
│  ─────────────────────────────────────────────────────── │
│                                                          │
│  [Tabs: CMS Instances │ Settings │ Extensions]           │
│                                                          │
│  ┌─ CMS Instances ───────────────────────────────────┐   │
│  │                                                    │   │
│  │  Enable this app's extensions for each CMS         │   │
│  │  instance. Extensions will be available to all      │   │
│  │  users on enabled instances.                        │   │
│  │                                                    │   │
│  │  ┌─[Table]────────────────────────────────────┐   │   │
│  │  │ Instance           │ Status     │ Action   │   │   │
│  │  │────────────────────│────────────│──────────│   │   │
│  │  │ Production         │ [Toggle:ON]│ [⋮]     │   │   │
│  │  │ cms.example.com    │ [Badge:    │          │   │   │
│  │  │                    │  Enabled]  │          │   │   │
│  │  │────────────────────│────────────│──────────│   │   │
│  │  │ Staging            │ [Toggle:OFF│ [⋮]     │   │   │
│  │  │ staging.example.com│ [Badge:    │          │   │   │
│  │  │                    │  Disabled] │          │   │   │
│  │  │────────────────────│────────────│──────────│   │   │
│  │  │ Dev                │ [Toggle:ON]│ [⋮]     │   │   │
│  │  │ dev.example.com    │ [Badge:    │          │   │   │
│  │  │                    │  Enabled]  │          │   │   │
│  │  └────────────────────────────────────────────┘   │   │
│  │                                                    │   │
│  └────────────────────────────────────────────────────┘   │
│                                                          │
│  ┌─ Extensions tab ──────────────────────────────────┐   │
│  │                                                    │   │
│  │  ┌─[Table]────────────────────────────────────┐   │   │
│  │  │ Extension              │ Injection Point   │   │   │
│  │  │────────────────────────│───────────────────│   │   │
│  │  │ Product Recs Widget    │ sidebar-panel     │   │   │
│  │  │ Analytics Overlay      │ toolbar           │   │   │
│  │  └────────────────────────────────────────────┘   │   │
│  │                                                    │   │
│  └────────────────────────────────────────────────────┘   │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**Toggle interaction:**
- ON → `Toast (success): "Extensions enabled for Production. Changes take effect immediately."`
- OFF → `AlertDialog: "Disable extensions for Production? Users on this instance will no longer see extensions from this app." [Cancel] [Disable]`
- Then `Toast: "Extensions disabled for Production."`

**⋮ DropdownMenu per instance:**
- Configure settings for this instance
- View extension activity
- Disable

---

### Screen A5: App Settings (per instance)

**Path:** `ocp.optimizely.com/apps/installed/{app-id}/settings/{instance-id}`
**Role:** Admin

```
┌──────────────────────────────────────────────────────────┐
│  ← Product Recommendations Pro                           │
│                                                          │
│  Settings — Production (cms.example.com)                 │
│  ─────────────────────────────────────────────────────── │
│                                                          │
│  API Key                                                 │
│  [Field: "sk-••••••••••••3f2a"           ] [Show]        │
│                                                          │
│  Max recommendations                                     │
│  [Field: "5"                              ]              │
│  How many content suggestions to show in the panel       │
│                                                          │
│  Content types to analyse                                │
│  [Select (multi): Pages, Blog Posts, Landing Pages  ▾]   │
│                                                          │
│  Language                                                │
│  [Select: ▾ Match CMS locale           ]                 │
│                                                          │
│                              [Cancel]  [Save Settings]   │
└──────────────────────────────────────────────────────────┘
```

---

## Phase B: Marketer (End User) Flow

### Screen M1: CMS Loads — Discovery API Call

**What happens (invisible to user):**

```
Sequence:
1. Marketer opens CMS UI (e.g., cms.example.com)
2. CMS client-side code calls OCP Discovery API:
   GET ocp.optimizely.com/api/discovery/extensions
     ?instance_id={cms-instance-id}
     &injection_points=sidebar-panel,toolbar,dashboard-widget
3. API returns list of enabled extensions for this instance:
   {
     "extensions": [
       {
         "id": "ext-prod-recs",
         "name": "Product Recommendations Widget",
         "injection_point": "sidebar-panel",
         "entry_url": "https://cdn.acme.com/recs-widget.js",
         "version": "1.2.0",
         "config": { "max_recs": 5 }
       },
       {
         "id": "ext-analytics",
         "name": "Analytics Overlay",
         "injection_point": "toolbar",
         "entry_url": "https://cdn.acme.com/analytics.js",
         "version": "1.2.0"
       }
     ]
   }
4. CMS extension loader injects each extension at its
   designated injection point
5. Extensions render inside the CMS UI
```

### Screen M2: CMS Content Editor with Extensions

**Path:** `cms.example.com/content/edit/{page-id}`
**Role:** Marketer
**Purpose:** Edit content — extensions appear as native CMS UI

```
┌──────────────────────────────────────────────────────────────────┐
│ [CMS Sidebar Nav]  │  Content Editor                            │
│                    │                                             │
│  Dashboard         │  ┌─[Toolbar]──────────────────────────┐    │
│  Content           │  │ B  I  U  H1  H2  ¶  🔗  📷        │    │
│  Media             │  │                        ┌──────────┐│    │
│  Forms             │  │                        │📊 Perf.  ││    │
│  Settings          │  │                        │(extension)││    │
│                    │  │                        └──────────┘│    │
│                    │  └────────────────────────────────────┘    │
│                    │                                             │
│                    │  [Content editing area]                     │
│                    │                                             │
│                    │  Welcome to our spring collection.          │
│                    │  Discover the latest trends in...           │
│                    │                                             │
│                    │                    ┌───────────────────────┐│
│                    │                    │ ≡ Recommendations     ││
│                    │                    │   (extension)         ││
│                    │                    │                       ││
│                    │                    │ Related content:      ││
│                    │                    │                       ││
│                    │                    │ • Summer Collection   ││
│                    │                    │   Guide (92% match)   ││
│                    │                    │                       ││
│                    │                    │ • Style Trends 2026   ││
│                    │                    │   (87% match)         ││
│                    │                    │                       ││
│                    │                    │ • Best Sellers Page   ││
│                    │                    │   (81% match)         ││
│                    │                    │                       ││
│                    │                    │ [+ Link selected]     ││
│                    │                    └───────────────────────┘│
│                    │                                             │
└──────────────────────────────────────────────────────────────────┘
```

**Key:** Extensions are visually indistinguishable from native CMS UI. No "extension" badges, no borders that feel foreign. They use the same OptiAxiom tokens, spacing, and typography as the rest of the CMS.

---

### Extension Loading States

**Loading (during hydration):**
```
┌───────────────────────┐
│ ≡ Recommendations     │
│                       │
│     [Spinner]         │
│   Loading...          │
│                       │
└───────────────────────┘
```
- Skeleton placeholder matching the injection point dimensions
- `Spinner` component from OptiAxiom, small size
- Max loading time: 5 seconds before fallback

**Error (extension fails to load):**
```
┌───────────────────────┐
│ ≡ Recommendations     │
│                       │
│ ⚠ Couldn't load this  │
│   panel. Try          │
│   refreshing.         │
│                       │
│   [Refresh]           │
│                       │
└───────────────────────┘
```
- Quiet, non-alarming. The CMS remains fully functional.
- Extension failure never blocks the CMS editor.
- After 3 failed retries, the panel collapses with no trace.

**API timeout (Discovery API unreachable):**
- CMS loads normally without any extensions
- No error shown to the marketer — extensions simply don't appear
- CMS retries Discovery API silently on next navigation
- Admin sees error in OCP monitoring dashboard

---

## Key Design Decisions

| Decision | Resolution |
|----------|-----------|
| How do admins know which injection points an extension targets? | Shown on App Detail page (A2) under "UI Extensions" tab with visual badges per injection point |
| How do admins enable/disable per CMS instance? | Toggle per instance on the Installed App page (A4). One toggle controls all extensions from that app for that instance |
| What happens when an extension fails to load? | Graceful degradation: skeleton → error message → collapse. CMS never blocked |
| How is versioning handled? | OCP serves the version the admin installed. Updates appear as "Update available" badge on installed app page. Admin explicitly approves updates |
| What does consent look like? | AlertDialog listing required permissions with plain-language descriptions before install (A3) |
| Can admins enable individual extensions separately? | v1: All extensions from an app are enabled/disabled together per instance. v2: Granular per-extension toggles |
| How fast must Discovery API respond? | Target < 200ms. CMS shows extensions within 1s of page load. Timeout at 5s |

---

## Complete Flow Diagram

```
DEVELOPER                         ADMIN                              MARKETER
━━━━━━━━━                         ━━━━━                              ━━━━━━━━

Define extensions                      │                                  │
in app manifest                        │                                  │
     │                                 │                                  │
     ▼                                 │                                  │
Configure injection                    │                                  │
points & permissions                   │                                  │
     │                                 │                                  │
     ▼                                 │                                  │
Deploy to OCP ─────────────────► App appears in                          │
                                 Marketplace                              │
                                      │                                   │
                                      ▼                                   │
                                 Admin browses                            │
                                 marketplace                              │
                                      │                                   │
                                      ▼                                   │
                                 Views app detail                         │
                                 (sees extensions                         │
                                  & injection points)                     │
                                      │                                   │
                                      ▼                                   │
                                 Clicks [Install]                         │
                                      │                                   │
                                      ▼                                   │
                                 Reviews & accepts                        │
                                 permissions                              │
                                      │                                   │
                                      ▼                                   │
                                 App installed                            │
                                      │                                   │
                                      ▼                                   │
                                 Enables extensions ──────────────► CMS instance
                                 per CMS instance                   is configured
                                      │                                   │
                                      │                                   ▼
                                      │                             Marketer opens
                                      │                             CMS UI
                                      │                                   │
                                      │                                   ▼
                                      │                             CMS calls
                                      │                             Discovery API
                                      │                              (background)
                                      │                                   │
                                      │                                   ▼
                                      │                             API returns
                                      │                             enabled extensions
                                      │                                   │
                                      │                                   ▼
                                      │                             Extensions injected
                                      │                             at injection points
                                      │                                   │
                                      │                                   ▼
                                      │                             Marketer sees
                                      │                             native CMS UI
                                      │                             with extensions
                                      │                                   │
                                      │                                   ▼
                                      │                             Marketer interacts
                                      │                             (never knows they're
                                      │                              "extensions")
```
