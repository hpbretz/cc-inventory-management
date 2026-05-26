---
description: Redesign the Vue 3 app UI into a modern SaaS layout with a vertical sidebar navigation
---

# Redesign to SaaS Sidebar Layout

Transform this app's UI from a horizontal top-nav bar into a modern SaaS-style interface with a vertical left sidebar.

**MANDATORY: Delegate ALL .vue file creation and modification to the vue-expert subagent.** Do not edit .vue files yourself.

---

## Goal

Replace the `<header class="top-nav">` horizontal navigation in `client/src/App.vue` with a collapsible vertical sidebar on the left. The main content area shifts to the right. The result should look like a polished B2B SaaS product (think Linear, Vercel dashboard, or Retool).

---

## Design Tokens

Apply these consistently across all modified files. Do not introduce new color values outside this palette.

```css
/* Layout */
--sidebar-width: 240px;
--sidebar-collapsed-width: 64px;
--topbar-height: 56px;

/* Colors */
--bg-base: #0f172a;         /* app background */
--bg-sidebar: #1e293b;      /* sidebar bg */
--bg-surface: #ffffff;      /* card / content bg */
--bg-hover: #334155;        /* sidebar item hover */
--bg-active: #3b82f6;       /* sidebar item active */

--text-primary: #f8fafc;    /* sidebar text */
--text-secondary: #94a3b8;  /* muted sidebar text */
--text-body: #1e293b;       /* main content text */
--text-muted: #64748b;

--border-default: #e2e8f0;
--border-sidebar: #334155;

/* Typography */
--font-sans: 'Inter', system-ui, -apple-system, sans-serif;

/* Spacing scale: 4px base */
--space-1: 4px;
--space-2: 8px;
--space-3: 12px;
--space-4: 16px;
--space-5: 20px;
--space-6: 24px;
--space-8: 32px;
```

---

## App Shell Layout (`client/src/App.vue`)

Restructure the app template into this two-column layout:

```
┌─────────────────────────────────────────────────────┐
│  sidebar (240px fixed)  │  main area (flex: 1)      │
│                         │                            │
│  [Logo + company name]  │  [page-topbar]             │
│                         │  [FilterBar]               │
│  [nav items]            │  [router-view]             │
│    Overview             │                            │
│    Inventory            │                            │
│    Orders               │                            │
│    Finance              │                            │
│    Demand Forecast      │                            │
│    Reports              │                            │
│                         │                            │
│  [spacer]               │                            │
│  [LanguageSwitcher]     │                            │
│  [ProfileMenu]          │                            │
└─────────────────────────────────────────────────────┘
```

**Template structure:**
```html
<div class="app-shell">
  <aside class="sidebar">
    <!-- logo, nav items, bottom controls -->
  </aside>
  <div class="app-main">
    <div class="page-topbar"><!-- page title from route meta or current route name --></div>
    <FilterBar />
    <main class="main-content">
      <router-view />
    </main>
  </div>
</div>
```

**CSS requirements:**
- `.app-shell`: `display: flex; height: 100vh; overflow: hidden;`
- `.sidebar`: `width: var(--sidebar-width); height: 100vh; background: var(--bg-sidebar); display: flex; flex-direction: column; flex-shrink: 0; border-right: 1px solid var(--border-sidebar);`
- `.app-main`: `flex: 1; display: flex; flex-direction: column; overflow: hidden; background: #f1f5f9;`
- `.main-content`: `flex: 1; overflow-y: auto; padding: var(--space-6);`

---

## Sidebar Nav Items

Each `<router-link>` becomes a styled block nav item:

```html
<nav class="sidebar-nav">
  <router-link to="/" class="nav-item" :class="{ active: $route.path === '/' }">
    <span class="nav-icon"><!-- SVG icon --></span>
    <span class="nav-label">Overview</span>
  </router-link>
  <!-- repeat for all routes -->
</nav>
```

**Nav item CSS:**
```css
.nav-item {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-2) var(--space-4);
  border-radius: 6px;
  margin: 2px var(--space-2);
  color: var(--text-secondary);
  text-decoration: none;
  font-size: 14px;
  font-weight: 500;
  transition: background 0.15s, color 0.15s;
  cursor: pointer;
}
.nav-item:hover { background: var(--bg-hover); color: var(--text-primary); }
.nav-item.active { background: var(--bg-active); color: #ffffff; }
.nav-icon { width: 18px; height: 18px; flex-shrink: 0; }
```

**Use these inline SVG icons (stroke-based, currentColor):**
- Overview / Dashboard: grid icon (`M3 3h7v7H3V3zm11 0h7v7h-7V3zM3 14h7v7H3v-7zm11 0h7v7h-7v-7z`)
- Inventory: box/cube icon (`M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z`)
- Orders: clipboard list icon (`M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 0 2-2h2a2 2 0 0 0 2 2`)
- Finance: dollar sign (`M12 1v22M17 5H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6`)
- Demand Forecast: trending up (`M23 6l-9.5 9.5-5-5L1 18`)
- Reports: bar chart (`M18 20V10M12 20V4M6 20v-6`)

---

## Sidebar Logo Area

```html
<div class="sidebar-logo">
  <div class="logo-mark"></div>
  <div class="logo-text">
    <span class="logo-name">{{ t('nav.companyName') }}</span>
    <span class="logo-sub">{{ t('nav.subtitle') }}</span>
  </div>
</div>
```

```css
.sidebar-logo {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-5) var(--space-4);
  border-bottom: 1px solid var(--border-sidebar);
  margin-bottom: var(--space-2);
}
.logo-mark {
  width: 32px; height: 32px;
  background: var(--bg-active);
  border-radius: 8px;
  flex-shrink: 0;
}
.logo-name { color: var(--text-primary); font-size: 14px; font-weight: 600; display: block; }
.logo-sub  { color: var(--text-secondary); font-size: 11px; display: block; }
```

---

## Sidebar Bottom Section

Push `LanguageSwitcher` and `ProfileMenu` to the bottom of the sidebar using `margin-top: auto`:

```html
<div class="sidebar-bottom">
  <LanguageSwitcher />
  <ProfileMenu ... />
</div>
```

```css
.sidebar-bottom {
  margin-top: auto;
  padding: var(--space-4);
  border-top: 1px solid var(--border-sidebar);
  display: flex;
  align-items: center;
  gap: var(--space-2);
}
```

---

## Page Topbar

Add a thin topbar at the top of the main area showing the current page title:

```html
<div class="page-topbar">
  <h2 class="page-title">{{ currentPageTitle }}</h2>
</div>
```

Derive `currentPageTitle` in `setup()` using `useRoute()` and a route-name map:
```javascript
const pageTitles = { '/': 'Overview', '/inventory': 'Inventory', '/orders': 'Orders', '/spending': 'Finance', '/demand': 'Demand Forecast', '/reports': 'Reports', '/backlog': 'Backlog' }
const currentPageTitle = computed(() => pageTitles[route.path] || '')
```

```css
.page-topbar {
  height: var(--topbar-height);
  display: flex;
  align-items: center;
  padding: 0 var(--space-6);
  background: #ffffff;
  border-bottom: 1px solid var(--border-default);
  flex-shrink: 0;
}
.page-title { font-size: 18px; font-weight: 600; color: var(--text-body); margin: 0; }
```

---

## FilterBar Placement

`FilterBar` sits **below the page topbar** and **above the router-view**, inside `.app-main`. It should have a white background and bottom border to visually separate it from the content area. The FilterBar component itself may need its own container styling updated — the vue-expert should check `client/src/components/FilterBar.vue` and ensure its wrapper has:

```css
background: #ffffff;
border-bottom: 1px solid var(--border-default);
padding: var(--space-3) var(--space-6);
flex-shrink: 0;
```

---

## Global Style Updates (`client/src/App.vue` `<style>`)

Remove all `.top-nav`, `.nav-container`, `.nav-tabs` rules. Add the design token variables to `:root` and apply `font-family: var(--font-sans)` to `body` or `.app-shell`.

Keep existing scoped styles for ProfileMenu and modals.

---

## Execution Instructions

1. **Spawn vue-expert** with all of the above context. Have it:
   a. Read `client/src/App.vue` in full first
   b. Read `client/src/components/FilterBar.vue` to understand its current wrapper styling
   c. Rewrite `client/src/App.vue` according to this spec (do not touch other .vue files unless FilterBar needs its wrapper style adjusted)

2. After the vue-expert completes, **start the dev server** (`/start`) and use Playwright to screenshot `http://localhost:3000` at 1280x800 to verify the sidebar renders correctly.

3. Report what changed and flag any visual issues to the user.

---

## What NOT to change

- Route configuration (`client/src/router/` or wherever routes are defined)
- Any view `.vue` files inside `client/src/views/`
- Backend code
- `api.js`, composables, or i18n files
- The behavior of LanguageSwitcher, ProfileMenu, or modals
