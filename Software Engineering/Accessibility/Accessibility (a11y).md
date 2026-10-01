---
tags: [accessibility, a11y, wcag, frontend]
---

# Accessibility (a11y)

**Accessibility**, abbreviated **a11y** — the numeronym `a` + the **11** letters in between + `y` — is the practice of building software that people with disabilities can perceive, operate, understand, and reliably use. It's not a feature bolted on at the end or a checklist ticked once; it's a property of the whole product, the same way performance or security are.

The framing that makes it click: accessibility is about **removing barriers between the interface and the user**, not about building a "special" version of the product for "special" users. A screen-reader user, a person with low vision, someone navigating only with a keyboard, or someone with a temporary injury all use the *same* product — the question is only whether the product lets them.

## Why it matters

Impairment shows up in three shapes, often in the same person at different times:

- **Permanent** — blindness, deafness, a missing limb, a chronic motor or cognitive condition.
- **Temporary** — a broken wrist, an eye infection, post-surgery recovery, a loud environment.
- **Situational** — holding a phone one-handed, driving, bright sunlight on a screen, a poor connection.

Because the boundary keeps moving, building for accessibility is really building for the **full range of human ability**. That's why the benefits are usually described by the **curb-cut effect**: captions written for deaf users help anyone in a noisy room, keyboard operability helps power users, high contrast helps everyone in the sun. Fixing the extreme case usually improves the average one too.

There is a hard reason as well: for public and institutional products (government portals, university systems, anything publicly funded), accessibility is frequently a **legal requirement**, and **WCAG level AA** is the bar most often written into policy.

## The POUR principles

The W3C organizes every WCAG success criterion under four principles:

- **Perceivable** — information must be presentable in ways the user can perceive. Content locked inside an image with no text alternative, or distinguished only by color, fails here.
- **Operable** — the interface must be usable through different inputs. A control that can only be clicked with a mouse and can't be reached by keyboard fails here.
- **Understandable** — content and behavior must be predictable and readable. Unlabeled form fields, inconsistent navigation and unexplained errors fail here.
- **Robust** — it must work with current and future user agents, including assistive technology. Valid, semantic markup that exposes the right roles and names to the accessibility tree is what makes this true.

## WCAG: versions and conformance levels

**WCAG** (Web Content Accessibility Guidelines) is the reference standard, maintained by the W3C's Web Accessibility Initiative. Two things to hold onto:

- **Versions.** WCAG 2.1 (2018) added criteria for mobile, low vision and cognitive needs; **WCAG 2.2** (2023) added a few more (focus appearance, dragging movements, consistent help, redundant entry). Criteria are backward-compatible — conforming to 2.2 means conforming to 2.1 and 2.0 as well.
- **Conformance levels.** Every criterion sits at one of three levels:
  - **A** — the minimum; without it some users simply cannot use the product.
  - **AA** — the realistic target, and the common legal bar.
  - **AAA** — the aspirational extreme; not expected site-wide, only where feasible.

"Conforming" is not the same as "perfect": it means meeting every A and AA criterion, or documenting an equally effective alternative.

## How it works: the techniques that carry most of the weight

1. **Semantic HTML first.** A `<button>` is exposed to assistive technology as a button, is keyboard-focusable, and announces its own state — a `<div onclick>` is not. Most accessibility work is choosing the right element, not sprinkling attributes.
2. **ARIA only when HTML can't express it.** ARIA (`role`, `aria-*`) patches the accessibility tree, but *no ARIA is better than bad ARIA* — an incorrect role is worse than none, because it lies to the screen reader. Reach for it for custom widgets, not to recreate what a native element already provides.
3. **Keyboard operability and focus management.** Everything must be reachable and actionable with the keyboard alone, in a sensible order, with a visible focus indicator. Modals and menus must trap and restore focus correctly.
4. **Text alternatives and media.** Images need `alt` (empty `alt=""` when purely decorative), videos need captions and transcripts, controls need accessible names.
5. **Color and contrast** — the most common failure of all, and rich enough to deserve its own note: see [[Color Accessibility]].

```html
<!-- Bad: not keyboard reachable, no role, no accessible name -->
<div class="btn" onclick="save()">Save</div>

<!-- Good: native semantics; keyboard, name and state come for free -->
<button type="button" onclick="save()">Save</button>

<!-- Decorative image: empty alt so it is skipped, not announced -->
<img src="divider.svg" alt="" />
```

## How it connects to the rest of the vault

Accessibility is not a layer you add at the end — it cuts across the stack. Contrast and color live in the design/theme layer, semantic markup lives in the component layer, and a persisted user preference (a color-vision mode, a high-contrast mode) is a small piece of global state. [[Accessible Choropleth Maps]] is a concrete case where all three met at once.

## See also

- [[Color Accessibility]]
- [[Accessible Choropleth Maps]]

## Reference

W3C — *Web Content Accessibility Guidelines (WCAG) 2.1* and *WCAG 2.2*. World Wide Web Consortium, Web Accessibility Initiative.

Write by **Samuel**
