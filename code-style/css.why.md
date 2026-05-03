# CSS — Rationale

Expanded rationale for opinionated rules in [css.md](./css.md). Section names mirror the rules file.

---

## Logical Properties

**Rule:** Prefer `inline-*` / `block-*` logical properties over physical `left` / `right` / `top` / `bottom`.

### Physical directions bake in English-language assumptions

`margin-left: 8px` literally means "put 8px of space on the viewport's left." In a left-to-right locale, that's "before the content." In a right-to-left locale (Arabic, Hebrew, Farsi, Urdu), the content flows the other way — now the margin is on the _trailing_ side of the element instead of the leading side. Every icon-next-to-text, every start-aligned badge, every timeline node quietly misaligns.

`margin-inline-start: 8px` means "put 8px before the content, in whichever direction content flows." Under `dir="ltr"` it resolves to `margin-left`; under `dir="rtl"` it resolves to `margin-right`. The browser does the work; the code doesn't change.

### Retrofitting is expensive; starting logical is free

A codebase that ships with physical directions and later needs to support RTL has to audit every `left`, `right`, `margin-left`, `padding-right`, `border-left`, `inset-left`, etc., and replace each one — while distinguishing the cases where physical was deliberate (viewport-anchored overlays) from the cases where it was just the default. That audit is tedious, error-prone, and easy to half-finish.

A codebase that starts with logical properties pays no tax: the syntax is the same length, browser support is universal, and the day someone enables an Arabic locale, the layout flips correctly without a single style change.

### When physical is actually right

- Absolute overlays anchored to the viewport, not to content (`position: fixed; left: 0`).
- Decorative offsets that shouldn't mirror (a signature in the bottom-right corner of a page, intentionally).
- Visual metaphors tied to physical space (a "left" and "right" panel in a DAW, where "left" and "right" are the subject, not a content flow direction).

Use physical deliberately in those cases. Use logical everywhere else.

---

## Relative Units for Type and Space

**Rule:** Use `rem` for type and typographic spacing; `px` for hairlines and pixel-precise details.

### `px` silently overrides user preferences

Every major browser lets the user set a base font size (for accessibility, vision, or personal preference). `font-size: 16px` ignores that setting entirely; `font-size: 1rem` respects it. Spacing tied to typography (padding inside a text container, line-relative margins) should scale the same way — a button with `padding: 12px 16px` gets cramped when the user bumps their base size up, while `padding: 0.75rem 1rem` stays balanced.

### `px` is still the right call for some things

Hairline borders, icon dimensions, and design-token details where sub-pixel rendering matters all benefit from exact pixel values. Scaling a 1px border to 1.125px at 18px base size gets you a fuzzy gray blur. Keep `px` for the places where pixel precision is the point.

### Viewport units earn their keep at the layout level

`100vh` for full-viewport layouts; `svh` / `dvh` / `lvh` when you need to reason about mobile browser chrome. Don't reach for `vw` / `vh` for small details — they don't compose with typography and break at unusual viewport sizes.

---

## Avoid `!important`

**Rule:** Treat `!important` as a signal of broken cascade, not a tool for winning it.

### `!important` is the nuclear option — using it once pressures everything else

Once `!important` exists in a codebase, every future override has to either out-specify the rule normally or add `!important` of its own. Stacking `!important`s doesn't produce better CSS; it produces CSS that nobody can change without adding yet another `!important`. The escape velocity keeps climbing.

### The fix is almost always specificity, not escalation

If you need `.foo { color: red !important }` to beat something, the "something" is probably doing too much — nested selectors, over-qualified rules, or specificity-boosting IDs. Fixing the overreaching rule is usually a five-minute change that removes the problem forever. Adding `!important` is a thirty-second change that creates a problem forever.

### The legitimate uses are narrow

- **Beating inline styles from a third-party:** external widgets that inject `style="..."` attributes can only be overridden with `!important`. Document which widget; consider whether the widget is worth the cost.
- **Low-specificity utility classes by design:** atomic CSS frameworks sometimes ship utilities with `!important` so they reliably win over component styles. This is the framework's design choice, applied consistently.
- **Stateful override classes in legacy systems:** `.is-hidden { display: none !important }` in a codebase with ambient styles that can't be cleaned up today. Use as a compatibility shim, not a pattern.

---

## Respect User Preferences

**Rule:** Honor `prefers-reduced-motion`, `prefers-color-scheme`, and `forced-colors`.

### Accessibility is a feature, not a polish pass

Reduced-motion users include people with vestibular disorders for whom animation is physically unpleasant — not a stylistic preference. Dark-mode users often include people with light sensitivity or migraines. Forced-colors users include people using screen-magnifier and high-contrast setups on Windows. Treating these as "default off" ships a product that makes a real population of users actively uncomfortable.

### Motion as additive, not default

The cleanest pattern is: the static state is the baseline; animation is an enhancement wrapped in `@media (prefers-reduced-motion: no-preference)`. The reverse pattern — animate by default, `prefers-reduced-motion: reduce` to disable — works but inverts the risk: you ship the accessibility violation by default and rely on the media query to clean it up.

### `forced-colors` requires testing, not intention

`forced-colors: active` replaces your palette with a system palette — foreground, background, highlight, link — and the user has tuned that palette to their eyes. Most "dark mode" designs look reasonable in forced-colors because the system colors take over cleanly. The places that break are typically:

- Icons conveyed via `background-image` (invisible — images don't get the forced palette).
- Outlines and borders set to specific hex values (overridden — fine, but your design intent is gone).
- Components that opt out with `forced-color-adjust: none` without verifying the result (usually a regression in disguise).

Test with Windows High Contrast or DevTools emulation before shipping anything visually complex.

---

## Custom Properties for Theming

**Rule:** Define design tokens as CSS custom properties; consume tokens in components.

### Custom properties are the universal theming primitive

Every tool — Vanilla Extract, Tailwind, CSS Modules, styled-components — either emits custom properties directly or composes cleanly with them. Building on custom properties means dark mode, density variants, and white-label themes all reduce to swapping a `:root` block, independent of what the components are authored in.

### Raw values drift; tokens stay consistent

A design system has maybe twenty meaningful colors and a dozen spacing steps. When components use those as tokens (`var(--color-border)`, `var(--space-md)`), a design change updates in one place. When components use raw values (`#e5e5e5`, `12px`), a design change requires a codebase-wide find-and-replace that you won't do completely. Token drift is one of the most common reasons "minor design tweaks" stall into multi-sprint refactors.

---

## Keep Specificity Flat

**Rule:** Target a single class per rule by default. Don't chain selectors for no reason.

### Specificity is a debt that compounds

Every selector has a specificity score; every override has to beat it. A rule written as `.card .title .label` forces every future override of `.label` to either match that chain or exceed it — and the next author writing `.danger-label` suddenly needs `.card .title .label.danger-label` or `!important`. The cost propagates to code that hasn't been written yet.

### Flat selectors leave room for change

A single class per rule gives future overrides room to win with just one more class or one attribute selector. Components stay independently styleable. Variants compose cleanly. There's no specificity ladder for other authors to climb.

### `:where()` is the release valve

When you need to write a base style that's explicitly meant to be overridden — a reset, a theme baseline, a default from a library — `:where()` wraps any selector and contributes zero specificity. `:where(.card) { ... }` matches `.card` but imposes no specificity cost on consumers. Use it where you want defaults that can be beaten trivially.
