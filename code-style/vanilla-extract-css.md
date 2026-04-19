# Vanilla Extract

Zero-runtime CSS-in-TS. Styles live in `.css.ts` files colocated with their component.

Rules here cover Vanilla-Extract-specific concerns (`style` arrays, variants vs. recipes, themes, sprinkles). General CSS rules — logical properties, units, specificity, user preferences — live in [css.md](./css.md) and apply here too.

Rationale for the opinionated sections lives in [vanilla-extract-css.why.md](./vanilla-extract-css.why.md). Strength levels (`non-negotiable` / `strong` / `weak`) are defined in [\_index.md](./_index.md).

## File Structure

`strength: strong`

```
Flower/
├── Flower.tsx
├── Flower.css.ts
└── index.ts
```

- One `.css.ts` per component, colocated.
- Export named consts; import them by name from the component.

```ts
// filename: Flower.css.ts
import { style } from '@vanilla-extract/css';

export const petal = style({
  backgroundColor: 'pink',
  borderRadius: 999,
  padding: 8,
});
```

## Style Arrays

`strength: strong` · [rationale](./vanilla-extract-css.why.md#style-array-ordering)

Order entries by increasing specificity: atomic utilities first, component base next, overrides and selectors last. Later entries win.

```ts
import { style } from '@vanilla-extract/css';
import { sprinkles } from '~/styles/sprinkles.css';
import { petal } from './Petal.css';

export const rosePetal = style([
  sprinkles({ display: 'flex', padding: 'md' }), // 1. atomic utilities
  petal, // 2. component base
  {
    // 3. overrides + selectors
    border: '1px solid crimson',
    selectors: {
      '&:hover': { transform: 'scale(1.05)' },
    },
  },
]);
```

## Variants

`strength: strong` · [rationale](./vanilla-extract-css.why.md#stylevariants-vs-recipe)

- `styleVariants` for a single axis of variation.
- `recipe` only when you have multiple independent axes or need compound variants.

```ts
// styleVariants — single axis
import { styleVariants } from '@vanilla-extract/css';

export const flowerPhase = styleVariants({
  bud: { transform: 'scale(0.5)', opacity: 0.6 },
  open: { transform: 'scale(1)', opacity: 1 },
  seed: { transform: 'scale(0.8)', opacity: 0.4 },
});
```

```ts
// recipe — multiple axes + compound variants
import { recipe } from '@vanilla-extract/recipes';

export const button = recipe({
  base: { borderRadius: 4, padding: 8 },
  variants: {
    size: { sm: { fontSize: 12 }, md: { fontSize: 14 }, lg: { fontSize: 16 } },
    tone: { neutral: { backgroundColor: 'gray' }, primary: { backgroundColor: 'crimson' } },
  },
  compoundVariants: [{ variants: { size: 'lg', tone: 'primary' }, style: { fontWeight: 'bold' } }],
  defaultVariants: { size: 'md', tone: 'neutral' },
});
```

## Theme and Vars

`strength: strong`

- `createTheme` for themeable token sets.
- `createVar` for one-off runtime-swappable values.

```ts
// filename: theme.css.ts
import { createTheme } from '@vanilla-extract/css';

export const [themeClass, vars] = createTheme({
  color: { petal: 'pink', stem: 'green' },
  space: { sm: '4px', md: '8px', lg: '16px' },
});
```

## Sprinkles

`strength: strong`

- Sprinkles for high-frequency utility classes: spacing, layout, color tokens.
- Don't use them for component-unique styling — reach for `style` instead.

## Global Styles

`strength: strong`

- Avoid `globalStyle`.
- When unavoidable (resets, third-party overrides, legacy integrations), colocate globals in a single `globals.css.ts` at the app root and scope selectors tightly.

## Naming

`strength: weak`

- Name exports after their role (`petal`, `stemActive`), not their properties (`pinkRounded`, `flexMd`).
- Match variant keys to the prop values that select them: `flowerPhase.bud` ↔ `phase: 'bud'`.
- Filename matches the component: `Flower.tsx` + `Flower.css.ts`.
