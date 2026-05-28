---
name: saas-redesign
description: Redesign this Vue 3 inventory-management app into a modern SaaS interface with a left vertical sidebar nav, consistent 8pt spacing, polished typography, and before/after screenshots. Use when the user asks to redesign the UI to a sidebar layout, polish the app, apply a consistent design system, or make it look like Linear/Rippling/Notion-style SaaS. Locked single-aesthetic system — slate/blue palette, Inter, sidebar shell, surface cards.
metadata:
  scope: project
  app: inventory-management
  version: "1.0.0"
---

# saas-redesign

A six-phase workflow that converts the inventory-management app from a top-nav SPA into a sidebar SaaS layout with a locked design system. The skill orchestrates; all `.vue` edits are delegated to the `vue-expert` subagent per the project's mandatory rule (`/CLAUDE.md` line 12).

## Hard constraints

These are non-negotiable. Verify each before completing.

- **Delegate all `.vue` edits to `vue-expert`.** This skill never edits `.vue` files directly. It hands `vue-expert` the relevant reference file and a precise instruction set.
- **No emojis in any UI text** (`/CLAUDE.md`). Sidebar nav items use text labels only.
- **Composition API only.** No Options API in any new or modified file.
- **Keep `v-for` keys unique by ID** — never use array index.
- **Preserve all existing features:** 6 routes, 4 global filters (`useFilters` composable), `useAuth` profile, `useI18n` translations (en/ja). No backend changes.
- **Project memory rule (`feedback_comment_nonobvious_logic`):** add a one-line WHY comment when a styling decision isn't self-evident.

## Phase 1 — Pre-flight

1. Check dev servers: `lsof -ti:3000,8001`.
   - If either is missing, invoke the project's `start` skill.
2. Verify the working tree:
   ```bash
   cd "/Users/shaanjain/Anthropic Basecamp/inventory-management" && git status --short
   ```
   - If unclean and on `main`, stop and tell the user. Otherwise note the current branch and continue.
3. Confirm `docs/redesign/` exists: `mkdir -p "/Users/shaanjain/Anthropic Basecamp/inventory-management/docs/redesign"`.

## Phase 2 — "Before" screenshots

Use Playwright MCP to capture each route at 1440×900. Save to `docs/redesign/before-<route>.png`.

Routes (slug → filename):
- `/` → `before-overview.png`
- `/inventory` → `before-inventory.png`
- `/orders` → `before-orders.png`
- `/demand` → `before-demand.png`
- `/spending` → `before-spending.png`
- `/reports` → `before-reports.png`

For each route: navigate, wait for the main content to settle (`await browser_wait_for { text: <page header text> }`), full-page screenshot.

## Phase 3 — Install design tokens

1. Copy `assets/tokens.css` to `client/src/styles/tokens.css` (create the `styles` directory).
2. Delegate to `vue-expert`: add `import './styles/tokens.css'` as the **first line** of `client/src/main.js` so tokens are globally available before any component mounts. (WHY comment: tokens must load before Vue components render so `var(--...)` resolves on first paint.)
3. Hand `vue-expert` `references/design-tokens.md` and instruct it to **replace ad-hoc hex codes and spacing values** in `App.vue` and every `client/src/views/*.vue` with the matching CSS variable. Do not touch component logic. Do not introduce new visual treatments yet — this phase is token substitution only.

## Phase 4 — Restructure App.vue shell

Delegate to `vue-expert` with `references/app-shell.md` and `assets/App.vue.template` as the spec.

Critical contract for the subagent:
- The sidebar contains: logo block (top), 6 `router-link`s (middle), footer block with `<LanguageSwitcher>` and `<ProfileMenu>` (bottom).
- The 6 nav items use existing `nav.*` translation keys from `client/src/locales/en.js` and `ja.js` — do not invent new keys.
- `<ProfileMenu>` and `<LanguageSwitcher>` are mounted **with identical props and event handlers** as today. Their dropdown direction must flip upward in the sidebar footer (CSS only — no component code changes).
- The main column hosts `<FilterBar>` sticky at the top, then `<router-view />`. `FilterBar` keeps its current `useFilters()` wiring untouched.
- Sidebar collapses to 72px icon-only below 1100px width.

## Phase 5 — Per-view polish pass

Delegate to `vue-expert` once per view file with `references/view-patterns.md`. For each of `Dashboard.vue`, `Inventory.vue`, `Orders.vue`, `Demand.vue`, `Spending.vue`, `Reports.vue`, `Backlog.vue`:

- Normalize page-header to: `h2` (font-size `var(--fs-h1)`, weight 700, letter-spacing `-0.02em`) + paragraph (color `var(--muted)`, font-size `var(--fs-body)`).
- Card padding → `var(--s-5)` (20px). Card radius → `var(--r-md)`. Card border → `1px solid var(--line)`. Card shadow → `var(--shadow-xs)`.
- Table cell padding → `var(--s-3) var(--s-4)`. Table header text-transform uppercase, letter-spacing `0.06em`, font-size `var(--fs-eyebrow)`, color `var(--muted)`.
- Button padding → `var(--s-2) var(--s-4)`. Radius `var(--r-sm)`.
- Stat-grid gap → `var(--s-5)`. Section margin-bottom → `var(--s-8)`.
- No logic changes. No new components. No new translation keys.

## Phase 6 — "After" screenshots + verification

1. If Vite or FastAPI need a reload, restart them (the `start` skill handles this).
2. Capture `after-<route>.png` for the same six routes using the Phase 2 routine.
3. Write `docs/redesign/index.html` — a minimal side-by-side gallery, one row per route, before on the left, after on the right, with the route name as a header. Inline CSS only (no build step).
4. Open the gallery: `open "/Users/shaanjain/Anthropic Basecamp/inventory-management/docs/redesign/index.html"`.
5. Verify in the browser (manual checklist):
   - Six routes load with no console errors.
   - Sidebar is persistent across navigation; active item highlights.
   - `FilterBar` still gates data: change Time Period on Dashboard → navigate to Orders → filter is preserved.
   - Language switcher toggles `en`↔`ja`; sidebar labels translate.
   - Profile menu opens `ProfileDetailsModal` and `TasksModal`.
   - No emoji anywhere in the rendered UI.

## File map at a glance

| File | Phase | Read | Pass to `vue-expert` |
|------|-------|------|----------------------|
| `references/design-tokens.md` | 3 | yes | yes |
| `references/app-shell.md` | 4 | yes | yes |
| `references/view-patterns.md` | 5 | yes | yes (per view) |
| `references/verification.md` | 2, 6 | yes | no — orchestrated by the skill |
| `assets/tokens.css` | 3 | no — copied verbatim | no |
| `assets/App.vue.template` | 4 | yes | yes |

## Stop conditions

Stop and report to the user (do not press on) if:
- Any phase produces a Vite or browser console error you cannot resolve in one retry.
- A delegated `vue-expert` task returns saying it cannot preserve the existing composables/translations contract.
- A route 404s after the shell rewrite.
- Working tree is on `main` with uncommitted changes at Phase 1.
