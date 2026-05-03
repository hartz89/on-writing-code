# Code Style

Opinionated guides for TypeScript, React, and Vanilla Extract CSS. Framing (audience, why-split, repo conventions) lives in the [root README](../README.md) and [CLAUDE.md](../CLAUDE.md); this file is the index into the guides themselves.

## Strength Levels

Applies to every rule across this repo — style guides and best practices alike.

- **non-negotiable** — hard rule. Violation requires a local comment explaining why.
- **strong** — default with rare exceptions. Deviation should be defensible on review.
- **moderate** — good default, but legitimate alternatives exist. Deviation doesn't require justification, just intent.
- **trivial** — preference for consistency. Either choice is defensible in isolation; pick one and stick with it.

See [typescript.md](./typescript.md) for the canonical example of these tags in use.

## Available Guides

| Guide           | Rules                                              | Rationale                                                  | Status   |
| --------------- | -------------------------------------------------- | ---------------------------------------------------------- | -------- |
| TypeScript      | [typescript.md](./typescript.md)                   | [typescript.why.md](./typescript.why.md)                   | Complete |
| React           | [react.md](./react.md)                             | [react.why.md](./react.why.md)                             | Complete |
| CSS (general)   | [css.md](./css.md)                                 | [css.why.md](./css.why.md)                                 | Complete |
| Vanilla Extract | [vanilla-extract-css.md](./vanilla-extract-css.md) | [vanilla-extract-css.why.md](./vanilla-extract-css.why.md) | Complete |

Contribution mechanics (adding a guide, the rule/rationale split, citation conventions) live in [CLAUDE.md](../CLAUDE.md).
