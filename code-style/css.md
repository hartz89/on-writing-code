# CSS

General styling rules that apply regardless of the authoring tool (vanilla CSS, CSS Modules, Vanilla Extract, styled-components, Tailwind, etc.). Tool-specific rules live alongside the tool — see [vanilla-extract-css.md](./vanilla-extract-css.md).

Rationale for the opinionated sections lives in [css.why.md](./css.why.md). Strength levels (`non-negotiable` / `strong` / `moderate` / `trivial`) are defined in [\_index.md](./_index.md).

## Guiding Principles

- Respect user intent: locale, reduced motion, color scheme, forced colors, font-size preferences.
- Prefer native CSS (custom properties, modern layout, logical properties, container queries) over tool-specific workarounds.
- Keep specificity flat.

## Logical Properties

`strength: strong` · [rationale](./css.why.md#logical-properties)

- Prefer `inline-*` / `block-*` (`margin-inline`, `padding-block`, `inset-inline-start`) over physical `left` / `right` / `top` / `bottom` for any content-flow-relative direction.
- Physical directions are fine when you genuinely mean "viewport left" (fixed overlays, deliberate non-mirroring, real-world metaphors).

```css
/* good — flips automatically under dir="rtl" */
.card {
  padding-inline: 16px;
  padding-block: 12px;
  margin-inline-start: 8px;
  border-inline-start-width: 2px;
}

/* avoid — breaks in RTL locales */
.card {
  padding-left: 16px;
  padding-right: 16px;
  margin-left: 8px;
  border-left-width: 2px;
}
```

## Relative Units for Type and Space

`strength: strong` · [rationale](./css.why.md#relative-units-for-type-and-space)

- `rem` for font sizes and typography-relative spacing.
- `px` for hairlines, pixel-precise iconography, and details where scaling would distort.
- Viewport units (`vw`, `vh`, `dvh`, `svh`) for layout that genuinely tracks the viewport.

```css
/* good — respects the user's base font size */
.card {
  font-size: 1rem;
  padding-block: 0.75rem;
  padding-inline: 1rem;
  border: 1px solid var(--color-border);
}
```

## Avoid `!important`

`strength: strong` · [rationale](./css.why.md#avoid-important)

- Treat as a last resort, not a tool for winning the cascade.
- Acceptable with a justifying comment: beating third-party inline styles, utility-class frameworks that ship with it by design, stateful override classes in legacy systems.
- Don't stack `!important` to beat earlier `!important`. Fix the specificity instead.

## Respect User Preferences

`strength: strong` · [rationale](./css.why.md#respect-user-preferences)

- **`prefers-reduced-motion`** — wrap non-essential animation; motion is additive, not default.
- **`prefers-color-scheme`** — honor the OS preference if the app has a dark mode.
- **`forced-colors`** — don't convey meaning via `background-image`; audit before using `forced-color-adjust: none`.

```css
/* good — motion is additive */
.toast {
  opacity: 1;
}

@media (prefers-reduced-motion: no-preference) {
  .toast {
    transition: opacity 200ms ease-out;
  }
}
```

## Custom Properties for Theming

`strength: strong` · [rationale](./css.why.md#custom-properties-for-theming)

- Define design tokens (colors, spacing, radii, typography) as CSS custom properties at the theme root.
- Consume tokens in components; avoid raw hex values and magic numbers.
- Use a tool's token system (`createTheme`, Tailwind config) when provided; it compiles to custom properties anyway.

## Keep Specificity Flat

`strength: strong` · [rationale](./css.why.md#keep-specificity-flat)

- One class per rule by default. No `.card .title .label` chains.
- Don't combine IDs with classes (`#app .card`) to boost specificity.
- `:where()` to _reduce_ specificity for base styles meant to be overridden.
- Nest only for real structural relationships (state, adjacent selectors, scoped variants) — not for tidiness.

## Semantic Class Names

`strength: moderate`

- Name classes for what the element _is_ or _does_: `card`, `primary-action`, `error-message`. Not `blue-box`, `padded-div`, `mt-4`.
- Utility classes (Tailwind-style) are the deliberate exception.
- Variants name the axis, not the appearance: `tone-danger`, not `red`.
