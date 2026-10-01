---
tags: [accessibility, a11y, data-visualization, choropleth, oklab, color-vision-deficiency]
---

# Accessible Choropleth Maps

A **choropleth map** shades regions by a data value, usually through a color scale. It's the most common way public data is shown on a map — and one of the easiest ways to accidentally exclude a large share of users, because the meaning is carried **only by color**. This note documents the concrete problem hit while building the ODIN observatory and the architecture used to fix it.

## The problem

Three patterns, each failing the same criterion for a different reason:

1. **Linear sRGB interpolation.** `interpolateHex` mixed the raw RGB channels (`r + (b - r) * t`). The eye doesn't read lightness linearly in sRGB, so mid-range values bunch together and become indistinguishable — especially under CVD.
2. **Static bicolor indicator scales.** Modules declared pairs like `colorScale: ["#FEF9C3", "#B91C1C"]` (light yellow → red) or `["#ECFDF5", "#065F46"]` (light green → dark green). For protanopia and deuteranopia, yellow↔red and green ramps lose essentially all contrast.
3. **A red/yellow/green "simplified" view.** The classic traffic-light palette (`critical: #ea580c`, `attention: #facc15`, `good: #0f766e`) is precisely the WCAG **1.4.1** anti-pattern — it reports status through color alone.

The unifying cause is **mono-encoding**: the map tells the story with a single channel (hue), so removing or misreading that channel removes the story.

## The architecture

The fix splits into four pieces, all **client-side** — no backend or API changes:

```
┌────────────────────────────────────────────────────────┐
│                   TopBar / Shell                       │
│  [🌓 Theme]  [👁️ Accessibility / color-vision mode ▼]  │
└──────────────────────────┬─────────────────────────────┘
                           │
       ┌───────────────────┴───────────────────┐
       ▼                                       ▼
┌──────────────────────────────┐ ┌──────────────────────────────┐
│  A11yContext / localStorage  │ │  Oklab Interpolator          │
│  - default                   │ │  - replaces interpolateHex   │
│  - protanopia_deuteranopia   │ │  - Cividis / Viridis palettes│
│  - tritanopia                │ │  - guaranteed lightness ramp │
│  - high_contrast             │ └──────────────┬───────────────┘
└──────────────┬───────────────┘                │
               │                                │
               ▼                                ▼
┌──────────────────────────────────────────────────────────────┐
│            Map & Legend                                       │
│  - choropleth applies the active mode's palette               │
│  - legend updates its gradient + band markers (double encode) │
│  - tooltip shows the textual band (not just the color)        │
└──────────────────────────────────────────────────────────────┘
```

## The four building blocks

### 1. Accessible color engine
A **palette registry** holds the scientific palettes (Cividis, Viridis, Okabe–Ito) and the substitution table for the simplified view, while an **Oklab interpolator** replaces linear sRGB mixing. Endpoints are chosen so lightness varies monotonically — the property that keeps bands distinguishable under CVD. See [[Color Accessibility]] for the theory.

### 2. Global state and persistence
A small context exposes `colorVisionMode: "default" | "protanopia_deuteranopia" | "tritanopia" | "high_contrast"` (plus an optional `usePatterns` flag), persisted to `localStorage` so the choice survives reloads without a server round trip.

### 3. Choropleth and legend integration
When a CVD mode is active, the choropleth swaps the indicator's original scale for the adapted palette, and the legend re-renders its gradient in real time. The legend also adds **double-encoded band markers** — e.g. `▼` critical, `◆` attention, `▲` good — so the meaning survives even if a user can't read the colors.

### 4. Accessibility menu
A menu next to the theme toggle offers one entry per mode, each with a **swatch preview** so the choice is self-explanatory:

- 👁️ Default
- 🔵🟡 Red-green (protanopia / deuteranopia) — Cividis-based
- 🔴🔷 Blue-yellow (tritanopia)
- ⚪⚫ High contrast (monochromatic)

## Why this shape

- **100% client-side** — nothing in the backend or the data pipeline changes.
- **No heavy dependencies** — Oklab interpolation and the palettes are small TypeScript files.
- **WCAG 2.1 / 2.2** (SC 1.4.1 and 1.4.3) compliant for the color encoding.
- **Non-invasive** — users who don't need a mode see the original experience; the improved option is one click away.

## See also

- [[Accessibility (a11y)]]
- [[Color Accessibility]]

Write by **Samuel**
