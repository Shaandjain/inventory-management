# View Patterns — saas-redesign

Per-view polish rules. Apply to every file in `client/src/views/*.vue`. No logic changes. No new components. Only `<style>` normalization plus minor `<template>` class additions (e.g., wrapping a header in `.page-header`) where required for the pattern to apply.

Hand this file to `vue-expert` once per view. Process order recommendation: Dashboard → Inventory → Orders → Spending → Demand → Reports → Backlog (start with the most-used).

## Universal page structure

Each view should follow this skeleton:

```vue
<template>
  <section class="page">
    <header class="page-header">
      <h2>{{ t('<view>.title') }}</h2>
      <p>{{ t('<view>.subtitle') }}</p>
    </header>

    <!-- existing content (cards, tables, charts) -->
  </section>
</template>
```

If a view does not currently have a header, add one using existing translation keys. If a key is missing, **do not invent a new translation key** — instead leave a `[REVIEW: missing nav/<view>.subtitle key]` comment for the user and skip that line for now.

## Token substitutions (style block normalization)

| Old (representative ad-hoc value) | New | Notes |
|------------------------------------|-----|-------|
| `padding: 1.25rem` (card) | `padding: var(--s-5)` | Default card padding |
| `padding: 0.5rem 0.75rem` (table cell) | `padding: var(--s-3) var(--s-4)` | Default table cell |
| `padding: 0.625rem 1.25rem` (button) | `padding: var(--s-2) var(--s-4)` | |
| `padding: 0.313rem 0.75rem` (badge) | `padding: var(--s-1) var(--s-3)` | And switch to badge recipe in `design-tokens.md` |
| `gap: 1rem` / `gap: 1.25rem` (grid) | `gap: var(--s-5)` or `var(--s-6)` | Use 24 if grid items are large cards |
| `margin-bottom: 1.5rem` (section) | `margin-bottom: var(--s-8)` | Page sections |
| `border-radius: 10px` (card) | `border-radius: var(--r-md)` | |
| `border-radius: 6px` (button) | `border-radius: var(--r-sm)` | |
| `background: #ffffff` / `#fff` | `background: var(--surface)` | |
| `background: #f1f5f9` | `background: var(--surface-2)` | |
| `border: 1px solid #e2e8f0` | `border: 1px solid var(--line)` | |
| `color: #0f172a` | `color: var(--ink)` | |
| `color: #64748b` | `color: var(--muted)` | |
| `color: #2563eb` | `color: var(--accent)` | |
| `box-shadow: 0 1px 3px 0 rgba(0,0,0,0.05)` | `box-shadow: var(--shadow-xs)` | |
| `font-family: 'Inter', …` | leave as-is OR remove (inherits from body) | Don't duplicate the family declaration in every component |

## Component-level patterns

### `.page-header`

```css
.page-header {
  margin-bottom: var(--s-8);
}
.page-header h2 {
  font-size: var(--fs-h1);
  font-weight: 700;
  letter-spacing: -0.02em;
  color: var(--ink);
  margin: 0 0 var(--s-2);
}
.page-header p {
  font-size: var(--fs-body);
  color: var(--muted);
  margin: 0;
}
```

### `.card`

```css
.card {
  background: var(--surface);
  border: 1px solid var(--line);
  border-radius: var(--r-md);
  padding: var(--s-5);
  box-shadow: var(--shadow-xs);
}
.card-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: var(--s-4);
  padding-bottom: var(--s-3);
  border-bottom: 1px solid var(--line);
}
.card-header h3 {
  font-size: var(--fs-lg);
  font-weight: 600;
  letter-spacing: -0.01em;
  color: var(--ink);
  margin: 0;
}
```

### `.stats-grid` and stat cards

```css
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: var(--s-5);
  margin-bottom: var(--s-8);
}
.stat-card {
  background: var(--surface);
  border: 1px solid var(--line);
  border-radius: var(--r-md);
  padding: var(--s-5);
}
.stat-label {
  font-size: var(--fs-eyebrow);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: var(--muted);
  margin-bottom: var(--s-2);
}
.stat-value {
  font-size: var(--fs-h2);
  font-weight: 700;
  letter-spacing: -0.02em;
  color: var(--ink);
}
.stat-delta {
  margin-top: var(--s-2);
  font-size: var(--fs-caption);
  color: var(--muted);
}
```

### Tables

```css
.data-table {
  width: 100%;
  border-collapse: collapse;
}
.data-table th {
  text-align: left;
  font-size: var(--fs-eyebrow);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  color: var(--muted);
  padding: var(--s-3) var(--s-4);
  border-bottom: 1px solid var(--line);
  background: var(--surface-2);
}
.data-table td {
  padding: var(--s-3) var(--s-4);
  font-size: var(--fs-small);
  color: var(--ink-2);
  border-bottom: 1px solid var(--line);
}
.data-table tbody tr:hover { background: var(--surface-2); }
.data-table tbody tr:last-child td { border-bottom: 0; }
```

### Buttons

```css
.btn {
  display: inline-flex;
  align-items: center;
  gap: var(--s-2);
  padding: var(--s-2) var(--s-4);
  border-radius: var(--r-sm);
  font-size: var(--fs-body);
  font-weight: 500;
  cursor: pointer;
  border: 1px solid transparent;
  transition: background-color .15s ease, border-color .15s ease, color .15s ease;
}
.btn-primary   { background: var(--accent); color: #fff; }
.btn-primary:hover { background: var(--accent-strong); }
.btn-secondary { background: var(--surface); color: var(--ink); border-color: var(--line); }
.btn-secondary:hover { background: var(--surface-2); }
.btn-ghost     { background: transparent; color: var(--muted); }
.btn-ghost:hover { color: var(--ink); background: var(--surface-2); }
```

### Charts

The existing custom SVG charts are kept. Replace any hard-coded grid/axis colors with `var(--line)` and axis labels with `var(--muted)`. Series colors: `var(--accent)` primary, `var(--success)` positive, `var(--danger)` negative, `var(--warn)` warning. (WHY: aligns chart colors with status badges so a green line = healthy across both.)

## Acceptance per view

- Page header present and styled per `.page-header`.
- All cards visually match the `.card` recipe.
- All tables follow `.data-table` (or a clearly-named variant for non-tabular grids).
- No literal hex values remain in `<style>` except for token definitions in `tokens.css`. (Exception: status colors used in custom SVG fills are fine but should reference `var(--success)` etc.)
- No emojis added anywhere.
- `v-for` keys remain unique IDs.
