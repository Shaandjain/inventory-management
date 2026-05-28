# Design Tokens — saas-redesign

The single source of truth for the locked design system. All values live in `assets/tokens.css` as CSS custom properties. This document explains intent and substitution rules.

## Color

| Token | Value | Use |
|-------|-------|-----|
| `--bg` | `#f8fafc` | App background (outside sidebar + main surfaces) |
| `--surface` | `#ffffff` | Cards, sidebar, modals |
| `--surface-2` | `#f1f5f9` | Hover states, table zebra, code chips |
| `--ink` | `#0f172a` | Primary text, headings |
| `--ink-2` | `#334155` | Body text inside cards |
| `--muted` | `#64748b` | Secondary text, eyebrow labels, table headers |
| `--muted-2` | `#94a3b8` | Disabled / placeholder |
| `--line` | `#e2e8f0` | Borders (cards, tables, dividers) |
| `--line-strong` | `#cbd5e1` | Emphasized dividers, focus rings |
| `--accent` | `#2563eb` | Primary buttons, active nav, links |
| `--accent-soft` | `#eff6ff` | Active nav background pill |
| `--accent-strong` | `#1e40af` | Hover state on accent |
| `--success` / `--success-soft` | `#10b981` / `#d1fae5` | Delivered, increasing, healthy |
| `--warn` / `--warn-soft` | `#f59e0b` / `#fed7aa` | Processing, low stock |
| `--danger` / `--danger-soft` | `#ef4444` / `#fecaca` | Backordered, out of stock |
| `--info` / `--info-soft` | `#3b82f6` / `#dbeafe` | Shipped, neutral indicator |

**Substitution rule for Phase 3:** any hex value in an existing `.vue` style block that matches one of these (case-insensitive) should become `var(--<token>)`. If a hex is *close* but not exact (e.g. `#fef0c7` vs `#fed7aa`), prefer the token unless the visual difference matters for that specific status badge — then keep the literal and add a one-line WHY comment.

## Spacing — 8pt grid

| Token | Value | Typical use |
|-------|-------|-------------|
| `--s-1` | `4px` | Icon gaps, badge gutters |
| `--s-2` | `8px` | Button vertical padding, tight gaps |
| `--s-3` | `12px` | Table cell vertical padding, form field gaps |
| `--s-4` | `16px` | Table cell horizontal padding, button horizontal padding |
| `--s-5` | `20px` | Card padding (default) |
| `--s-6` | `24px` | Grid gaps inside views |
| `--s-8` | `32px` | Main content vertical padding, section margin-bottom |
| `--s-10` | `40px` | Main content horizontal padding |
| `--s-12` | `48px` | Page top padding on large screens |

**Substitution rule:** any literal `rem`/`px` value used for padding/margin/gap that maps cleanly to a token (e.g. `1.25rem` → `var(--s-5)`, `0.5rem 0.75rem` → `var(--s-2) var(--s-3)`) gets substituted. Values that do not map cleanly (e.g. odd `0.313rem` badge padding) get rounded to the nearest token unless an explicit design reason exists — and that reason becomes a WHY comment.

## Radii

| Token | Value | Use |
|-------|-------|-----|
| `--r-sm` | `6px` | Buttons, inputs, badges, dropdown items |
| `--r-md` | `10px` | Cards, modals, dropdown menus |
| `--r-lg` | `14px` | Hero panels, feature surfaces |

## Shadows

| Token | Value | Use |
|-------|-------|-----|
| `--shadow-xs` | `0 1px 2px rgba(15,23,42,0.04)` | Cards (resting) |
| `--shadow-sm` | `0 1px 3px rgba(15,23,42,0.06), 0 1px 2px rgba(15,23,42,0.04)` | Cards (hover), sidebar right edge |
| `--shadow-md` | `0 8px 24px rgba(15,23,42,0.08)` | Dropdowns, popovers, profile menu |
| `--shadow-lg` | `0 24px 48px rgba(15,23,42,0.12)` | Modals |

## Typography

Family: `'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif` — already imported in `App.vue`. Do not change.

| Token | Value | Use |
|-------|-------|-----|
| `--fs-eyebrow` | `11px` | Uppercase labels, table headers (letter-spacing `0.08em`) |
| `--fs-caption` | `12px` | Helper text, metadata |
| `--fs-small` | `13px` | Table body, dropdown items |
| `--fs-body` | `14px` | Body copy, form inputs |
| `--fs-md` | `16px` | Card-level headings (h4), buttons |
| `--fs-lg` | `18px` | Card title (h3), modal title |
| `--fs-xl` | `22px` | Section header |
| `--fs-h2` | `28px` | Page subheaders |
| `--fs-h1` | `36px` | Page title (single h2 per view) |

Heading rules:
- 18px+ → letter-spacing `-0.01em`, weight 600.
- 28px+ → letter-spacing `-0.02em`, weight 700.
- Body line-height 1.5, headings 1.25.

## Sidebar specifics

| Property | Value |
|----------|-------|
| Width (desktop ≥1100px) | `248px` |
| Width (collapsed <1100px) | `72px` (icons + tooltip on hover) |
| Background | `var(--surface)` |
| Right border | `1px solid var(--line)` |
| Logo block padding | `var(--s-6) var(--s-5)` |
| Nav item padding | `var(--s-3) var(--s-4)` |
| Nav item radius | `var(--r-sm)` |
| Nav item gap | `var(--s-1)` |
| Active background | `var(--accent-soft)` |
| Active text | `var(--accent)` |
| Active left accent bar | `4px solid var(--accent)`, inset on the radius |
| Hover background | `var(--surface-2)` |
| Footer padding | `var(--s-5)` |
| Footer top border | `1px solid var(--line)` |

## Main content area

| Property | Value |
|----------|-------|
| Padding | `var(--s-8) var(--s-10)` |
| Max-width | `1440px` |
| Margin | `0 auto` |
| Background | `var(--bg)` |
| FilterBar sticky top | `0` (within main column scroll) |
| Section gap | `var(--s-8)` |

## Status badge recipe

Replace existing ad-hoc badge styles with this single pattern:

```css
.badge {
  display: inline-flex;
  align-items: center;
  gap: var(--s-1);
  padding: var(--s-1) var(--s-3);
  border-radius: 999px; /* pill */
  font-size: var(--fs-caption);
  font-weight: 600;
  letter-spacing: 0.02em;
}
.badge.success { background: var(--success-soft); color: var(--success); }
.badge.warn    { background: var(--warn-soft);    color: var(--warn); }
.badge.danger  { background: var(--danger-soft);  color: var(--danger); }
.badge.info    { background: var(--info-soft);    color: var(--info); }
```

## What NOT to do in Phase 3

- Do not change layout (no new grids, no shell rewrite — that is Phase 4).
- Do not delete or rename CSS classes used by other components.
- Do not introduce new colors outside this table.
- Do not touch `<template>` or `<script>` blocks. Token substitution lives in `<style>` only.
