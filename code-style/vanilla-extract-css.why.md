# Vanilla Extract — Rationale

Expanded rationale for opinionated rules in [vanilla-extract-css.md](./vanilla-extract-css.md). Section names mirror the rules file.

---

## Style Array Ordering

**Rule:** In `style([...])`, order entries by increasing specificity: atomic utilities, then component base, then overrides and selectors.

### `style([])` concatenates classes; the cascade decides the winner

When you pass an array to `style`, VE emits each entry as a separate class and joins them with a space. The browser's cascade resolves conflicts: at equal specificity, the class that appears later in the stylesheet wins. VE emits classes in the order they're declared, so an override at the end of the array reliably overrides an atomic utility at the start.

Putting overrides first doesn't "just work" — the atomic utilities declared later will overwrite them, and you'll find yourself reaching for `!important` or restructuring. Ordering by increasing specificity is the path that stays out of your way.

### Why this specific order

- **Atomic utilities (sprinkles) first** — they're the most generic; they set defaults across the system.
- **Component base next** — it's the component's identity, the styles that define what this thing looks like.
- **Overrides and selectors last** — `:hover`, media queries, prop-driven tweaks. They're the most specific and should win.

---

## `styleVariants` vs `recipe`

**Rule:** Reach for `recipe` only when you have multiple independent axes or compound variants.

### `styleVariants` is cheaper to read

For a single axis (phase, tone, size), `styleVariants` produces a keyed object: `flowerPhase.bud`. It reads like an enum. There's no type gymnastics, no runtime wrapper, no defaults machinery to think about.

### `recipe` earns its complexity when axes multiply

Once you have multiple axes (size × tone), the combinatorial space outgrows `styleVariants`. `recipe` is built for this — typed variants, compound variants, and defaults in one place. But: every `recipe` is one more API surface to learn when reading the file.

Rule of thumb: if a prop maps 1:1 to a key, use `styleVariants`. If you find yourself concatenating variant classes by hand or writing lookup tables, escalate to `recipe`.
