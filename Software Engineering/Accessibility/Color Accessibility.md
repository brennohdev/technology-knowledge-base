---
tags: [accessibility, a11y, color, color-vision-deficiency, wcag, data-visualization, oklab]
---

# Color Accessibility

Color is the cheapest channel to encode meaning — one attribute, one value, read at a glance. It's also the first channel to break: it disappears on monochrome output, fails for users who can't separate the hues, and vanishes entirely for screen-reader users. This note is about making color an *addition* to meaning, never its *only* carrier.

## Use of Color (WCAG SC 1.4.1)

The relevant criterion is **1.4.1 Use of Color** (Level A): *color must not be the only visual means of conveying information, indicating an action, prompting a response, or distinguishing a visual element.*

The canonical failure is a chart that uses red for "bad", yellow for "attention", green for "good" and nothing else — anyone who can't tell red from green, or who prints in grayscale, loses the entire meaning. Status dots, required-field markers, error states and thematic maps are the usual offenders.

## Color vision deficiency (CVD)

Roughly **1 in 12 men and 1 in 200 women** have some form of CVD. The distinctions that matter for design:

- **Protanopia / protanomaly** — reduced or missing red-cone sensitivity. Reds darken and shift; red↔green contrast collapses.
- **Deuteranopia / deuteranomaly** — reduced or missing green-cone sensitivity. Together with protanopia this accounts for the **~99% of all CVD cases**, and both are **red-green** conditions.
- **Tritanopia / tritanomaly** — blue-cone deficiency (rare). Yellow↔blue contrast collapses instead.
- **Achromatopsia** — no color perception at all; only luminance remains.

The practical consequence for data: any **red↔green** or **yellow↔green** ramp degrades into a single muddy band of brown/gray for the overwhelming majority of colorblind users. A "yellow → red" severity scale is exactly the wrong choice, even though it feels intuitive.

## Perceptual color spaces

Even for users with full color vision, the *way* you interpolate between two colors matters. Linear interpolation in **sRGB** is the naive approach:

```ts
// Linear sRGB interpolation: interpolate the raw channels
function interpolateHex(from: string, to: string, t: number): string {
  const a = hexToRgb(from);
  const b = hexToRgb(to);
  const r = Math.round(a.r + (b.r - a.r) * t);
  const g = Math.round(a.g + (b.g - a.g) * t);
  const bl = Math.round(a.b + (b.b - a.b) * t);
  return rgbToHex({ r, g, bl });
}
```

It produces gradients with **dead zones**: perceived lightness doesn't move uniformly from one end to the other, so mid-range values bunch together and become hard to tell apart — and the problem is far worse under CVD.

Interpolating in a **perceptual color space** fixes this. **Oklab** (Björn Ottosson, 2020) is a modern, lightweight perceptual space: distances in it approximate how different two colors *look*, and its **L** axis tracks perceived lightness. Moving `L` at a steady rate yields a ramp that reads uniformly from end to end — for trichromats, and (with well-chosen endpoints) for protanopes and deuteranopes too.

```ts
// The shape of the fix: convert both endpoints to Oklab, interpolate,
// and convert back. Because L varies monotonically, no band of
// intermediate values collapses into its neighbours.
function interpolateOklab(from: string, to: string, t: number): string {
  const a = srgbToOklab(hexToRgb(from));
  const b = srgbToOklab(hexToRgb(to));
  const mixed = {
    l: a.l + (b.l - a.l) * t,
    a: a.a + (b.a - a.a) * t,
    b: a.b + (b.b - a.b) * t,
  };
  return rgbToHex(oklabToSrgb(mixed));
}
```

The implementation is pure vector math — a few matrix multiplications between linear sRGB and the Oklab `L/a/b` axes — so no dependency is needed, which is what makes it practical to ship client-side.

## Colorblind-safe palettes

The state of the art is a set of palettes designed with perceptual uniformity and CVD in mind:

| Palette | Type | Why it's used |
|---------|------|---------------|
| **Viridis** | continuous | Perceptually uniform and colorblind-friendly; the modern default for continuous data (Matplotlib / R). |
| **Cividis** | continuous | Optimized specifically for **CVD** (Pacific Northwest National Laboratory, 2018); luminance-ordered for trichromats and for red-green deficiencies alike. |
| **Okabe–Ito** | categorical | An 8-color qualitative set designed for color blindness. |
| **Paul Tol** | categorical / sequential | Bright, muted and high-contrast variants that stay safe under CVD and in print. |

Bicolor scales are still common; the trick is to pick endpoints that differ in **lightness** and, under red-green CVD, in **blue↔yellow**. A substitution table that replaces a failing red/yellow/green scheme:

| Semantic | Default (fails CVD) | CVD-safe (Paul Tol Bright) |
|----------|----------------------|-----------------------------|
| critical | red | deep blue `#004488` |
| attention | yellow | gold `#ddaa33` |
| good | green | light blue `#99ddff` |

## Double encoding

No palette alone satisfies WCAG 1.4.1, because color is still the only channel. **Double encoding** carries the same information on a second, non-color channel:

- **Shape** — a different marker per category (circle / square / triangle).
- **Pattern** — hatching or dots inside polygons.
- **Label / symbol** — a letter or icon next to each band (e.g. `▼` critical, `◆` attention, `▲` good).
- **Luminance** — endpoints far apart in lightness, so the ramp survives grayscale.

This is what turns a "colorblind-friendly palette" into genuinely accessible encoding.

## Contrast criteria

Accessibility also constrains text and UI contrast, independently of hue:

- **SC 1.4.3 Contrast (Minimum)** (AA) — normal text needs a ratio of at least **4.5:1** against its background; large text (≥ 18.66px bold or ≥ 24px) needs **3:1**.
- **SC 1.4.11 Non-text Contrast** (AA) — UI components, icons and graphical objects that carry meaning need **3:1** against adjacent colors.
- AAA raises text contrast to **7:1**.

## See also

- [[Accessibility (a11y)]]
- [[Accessible Choropleth Maps]]

## Reference

- NUÑEZ, J. R.; ANDERTON, C. R.; RENSLOW, R. S. *Optimizing colormaps with consideration for color vision deficiency to enable accurate interpretation of scientific data*. PLoS ONE, v. 13, n. 7, e0199239, 2018. (Cividis)
- OTTOSSON, Björn. *A perceptual color space for image processing*. 2020. (Oklab)
- W3C — *Web Content Accessibility Guidelines (WCAG) 2.1* (SC 1.4.1, 1.4.3, 1.4.11).

Write by **Samuel**
