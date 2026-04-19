# Code Style

Opinionated guides for TypeScript, React, and Vanilla Extract CSS. Each guide is designed for fast scanning by humans and context-efficient consumption by LLMs.

## The ".why.md" Pattern

Some guides split into two files: a rules file and a rationale file.

- **`<lang>.md`** — rules and examples. Scannable format: strength tags (`non-negotiable` / `strong` / `weak`), concise rules, code samples. Rules link to rationale when the reasoning isn't obvious.
- **`<lang>.why.md`** — expanded rationale and context. Section names match the rules file so anchor links work. Read this when you need to understand the _why_, defend an edge case, or evaluate a rule for your context.

### Why split?

**Scanning:** An LLM applying a guide only needs the rules and examples. Rationale prose (history, tradeoffs, exceptions) adds context cost without changing behavior — move it to a sibling file so the main file stays terse.

**Reasoning:** A human (or an LLM making a judgment call) can read the why file to understand the full picture. Keeping rationale separate from rules means we can be opinionated in the rules without needing to hedge or explain every corner case inline.

## Strength Levels

Each rule in a guide is tagged with one of:

- **non-negotiable** — hard rule. Violation requires a local comment explaining why.
- **strong** — default with rare exceptions. Deviation should be defensible on review.
- **weak** — preference for consistency. Either choice is defensible in isolation; pick one and stick with it.

See [typescript.md](./typescript.md) for the canonical example.

## Available Guides

| Guide           | Rules                                              | Rationale                                                  | Status   |
| --------------- | -------------------------------------------------- | ---------------------------------------------------------- | -------- |
| TypeScript      | [typescript.md](./typescript.md)                   | [typescript.why.md](./typescript.why.md)                   | Complete |
| React           | [react.md](./react.md)                             | [react.why.md](./react.why.md)                             | Complete |
| Vanilla Extract | [vanilla-extract-css.md](./vanilla-extract-css.md) | [vanilla-extract-css.why.md](./vanilla-extract-css.why.md) | Complete |

## Adding a New Guide or Rule

1. Create a new rules file (`<lang>.md`) or update an existing one.
2. Tag each section with a strength level (see above).
3. Add rationale only when it's non-obvious or worth defending. Keep rules terse; prose goes in the why file.
4. If adding rationale, create a sibling `<lang>.why.md` file with section names that match the rules file exactly.
5. In the rules file, add anchor links: `[rationale](./typescript.why.md#section-anchor)` so readers can jump to context.

Naming convention: Section anchors use lowercase and hyphens (e.g., `#discriminated-unions`).
