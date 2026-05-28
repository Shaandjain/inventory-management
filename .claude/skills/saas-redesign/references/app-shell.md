# App Shell — saas-redesign

Spec for restructuring `client/src/App.vue` into a sidebar SaaS layout. Hand this file (and `assets/App.vue.template`) to `vue-expert`. The template is a reference, not a copy target — `vue-expert` adapts it to the existing imports, composables, and translation keys.

## Goal

Replace the current top-nav header with a left vertical sidebar that contains the logo, six router links, and a footer block hosting the existing `LanguageSwitcher` and `ProfileMenu` components. The main column hosts the existing `FilterBar` (sticky at the top of the main scroll region) and `<router-view />`.

## Layout (desktop ≥1100px)

```
┌──────────┬───────────────────────────────────────┐
│ Logo     │ FilterBar (sticky, top: 0 of main)    │
│          ├───────────────────────────────────────┤
│ Nav      │                                       │
│  • Over… │ <router-view />                       │
│  • Inv…  │                                       │
│  • Ord…  │   max-width 1440px, centered          │
│  • Fin…  │   padding var(--s-8) var(--s-10)      │
│  • Dem…  │                                       │
│  • Rep…  │                                       │
│          │                                       │
│ ─────    │                                       │
│ Lang     │                                       │
│ Profile  │                                       │
└──────────┴───────────────────────────────────────┘
  248px            flex: 1, scrollable
```

## Layout (<1100px)

Sidebar collapses to 72px, showing icons only. Logo collapses to the short mark (or first letter of company name from `t('nav.companyName')`). Nav labels hide; tooltip on hover reveals the label. Footer items become icon-only (globe icon + avatar).

Use a single breakpoint at `1100px`. (WHY: sidebar 248 + main padding 40 + content ~720 = 1008px minimum readable; 1100 gives breathing room.)

## Markup contract (Composition API + `<script setup>` is acceptable; existing file uses options-style — keep whichever vue-expert decides, but stay Composition API)

```vue
<template>
  <div class="app">
    <aside class="sidebar" :class="{ collapsed: isCompact }">
      <div class="sidebar-logo">
        <!-- logo mark + (when expanded) company name from t('nav.companyName') -->
      </div>

      <nav class="sidebar-nav">
        <router-link to="/"          :class="{ active: $route.path === '/' }">
          <span class="nav-icon" aria-hidden="true"><!-- inline svg --></span>
          <span class="nav-label">{{ t('nav.overview') }}</span>
        </router-link>
        <router-link to="/inventory" :class="{ active: $route.path === '/inventory' }"> … </router-link>
        <router-link to="/orders"    :class="{ active: $route.path === '/orders' }">    … </router-link>
        <router-link to="/spending"  :class="{ active: $route.path === '/spending' }">  … </router-link>
        <router-link to="/demand"    :class="{ active: $route.path === '/demand' }">    … </router-link>
        <router-link to="/reports"   :class="{ active: $route.path === '/reports' }">   … </router-link>
      </nav>

      <div class="sidebar-footer">
        <LanguageSwitcher />
        <ProfileMenu
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
      </div>
    </aside>

    <main class="main">
      <FilterBar />
      <div class="main-content">
        <router-view />
      </div>
    </main>

    <!-- Modals (unchanged from current App.vue) -->
    <ProfileDetailsModal v-if="showProfileDetails" @close="showProfileDetails = false" />
    <TasksModal v-if="showTasks" @close="showTasks = false" />
  </div>
</template>
```

## Critical preservation rules

- `<LanguageSwitcher />` props/events: **unchanged**. Its internal dropdown should open *upward* in this position — accomplish that with a one-line prop or a CSS override on `.language-switcher .dropdown { bottom: 100%; top: auto; }`. Add a WHY comment on the override.
- `<ProfileMenu />` props/events: **unchanged**. Same upward-dropdown CSS rule. The pending-task badge logic in ProfileMenu must keep working.
- `<FilterBar />`: **unchanged component**. New parent CSS sets `position: sticky; top: 0; z-index: 30; background: var(--bg);` so it pins under nothing.
- Route paths are the existing six from `main.js`. Do not add or rename routes.
- Translation keys: `t('nav.overview')`, `t('nav.inventory')`, `t('nav.orders')`, `t('nav.finance')`, `t('nav.demandForecast')`, `t('nav.reports')`. (Confirm exact keys in `client/src/locales/en.js` before mapping — if any nav key is missing in `ja.js`, that is a stop condition; tell the user.)
- The current `App.vue` global style block applies to all views. Keep it global (not scoped) but rewrite to use token vars.

## Style block — sidebar specifics

```css
.app {
  display: flex;
  min-height: 100vh;
  background: var(--bg);
}

.sidebar {
  width: 248px;
  flex: 0 0 248px;
  background: var(--surface);
  border-right: 1px solid var(--line);
  display: flex;
  flex-direction: column;
  position: sticky;
  top: 0;
  height: 100vh;
}

.sidebar-logo {
  padding: var(--s-6) var(--s-5);
  border-bottom: 1px solid var(--line);
  display: flex;
  align-items: center;
  gap: var(--s-3);
}

.sidebar-nav {
  flex: 1;
  padding: var(--s-4) var(--s-3);
  display: flex;
  flex-direction: column;
  gap: var(--s-1);
  overflow-y: auto;
}

.sidebar-nav a {
  display: flex;
  align-items: center;
  gap: var(--s-3);
  padding: var(--s-3) var(--s-4);
  border-radius: var(--r-sm);
  font-size: var(--fs-body);
  font-weight: 500;
  color: var(--ink-2);
  text-decoration: none;
  position: relative;
  transition: background-color .15s ease, color .15s ease;
}

.sidebar-nav a:hover { background: var(--surface-2); color: var(--ink); }

.sidebar-nav a.active {
  background: var(--accent-soft);
  color: var(--accent);
}

/* Left 4px accent bar on active. WHY: stronger directional cue than background alone. */
.sidebar-nav a.active::before {
  content: '';
  position: absolute;
  left: 0;
  top: 8px;
  bottom: 8px;
  width: 3px;
  background: var(--accent);
  border-radius: 0 var(--r-sm) var(--r-sm) 0;
}

.sidebar-footer {
  padding: var(--s-4) var(--s-3);
  border-top: 1px solid var(--line);
  display: flex;
  flex-direction: column;
  gap: var(--s-2);
}

.main {
  flex: 1;
  min-width: 0;          /* WHY: prevents long tables from pushing sidebar wider */
  display: flex;
  flex-direction: column;
}

.main-content {
  max-width: 1440px;
  width: 100%;
  margin: 0 auto;
  padding: var(--s-8) var(--s-10);
}

@media (max-width: 1100px) {
  .sidebar { width: 72px; flex-basis: 72px; }
  .sidebar .nav-label,
  .sidebar-logo .logo-text { display: none; }
  .sidebar-nav a { justify-content: center; padding: var(--s-3); }
  .main-content { padding: var(--s-6) var(--s-5); }
}
```

## Icon strategy

Inline SVGs (12px stroke, currentColor) — no icon library, no emojis. One per route: dashboard grid (Overview), box (Inventory), receipt (Orders), wallet (Finance/Spending), chart-line (Demand), document (Reports). `vue-expert` can author these inline or extract to a small `NavIcon.vue` component if six SVGs in the template feels noisy — that is a judgement call.

## Acceptance for Phase 4

- All six routes render and active-state highlight is correct on each.
- `FilterBar` works as before — changing a filter re-fetches data in the current view.
- `ProfileMenu` and `LanguageSwitcher` open without clipping below the sidebar footer.
- Window narrower than 1100px: sidebar collapses to 72px without overlap or content cut-off.
- No console errors.
