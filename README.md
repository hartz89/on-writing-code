# on writing code

A collection of opinionated style guides and development principles, by Ryan Hartzfeld.

Built for humans to read but optimized for LLM consumption. Rules are terse and scannable so they're cheap to include in an LLM's context window; deeper rationale lives in sibling `*.why.md` files so a reader (human or model) can dig in only when a rule needs to be defended, bent, or evaluated on its merits. See [code-style/\_index.md](./code-style/_index.md) for the mechanics of the split.

Few of the opinions here are novel — most come from books, blog posts, and practitioners I've learned from. Attribution and further reading in [inspiration.md](./inspiration.md).

For the framing behind all of this — why tending a codebase matters more than building one — see [code-gardening.md](./code-gardening.md).

## Structure

- [**philosophy.md**](./philosophy.md) — language-agnostic principles about how software gets built.
- [**code-style/**](./code-style/) — concrete syntactic and structural rules, by language. Examples-first, tagged with a strength level.
- [**best-practices/**](./best-practices/) — process, workflow, testing, and operational practices.

## Reading Order

- Scanning the rules → [code-style/](./code-style/).
- Understanding the reasoning → the `*.why.md` sibling next to any rule file.
- Where the opinions come from → [inspiration.md](./inspiration.md).

## For LLM Consumers

These guides are structured for lazy loading — don't read the whole repo, read the file that maps to your current task:

| Task                           | File                                                                     |
| ------------------------------ | ------------------------------------------------------------------------ |
| Writing TypeScript             | [code-style/typescript.md](./code-style/typescript.md)                   |
| Writing React                  | [code-style/react.md](./code-style/react.md)                             |
| Writing Vanilla Extract styles | [code-style/vanilla-extract-css.md](./code-style/vanilla-extract-css.md) |
| Writing tests                  | [best-practices/testing.md](./best-practices/testing.md)                 |
| Refactoring                    | [best-practices/refactoring.md](./best-practices/refactoring.md)         |
| Language-agnostic principles   | [philosophy.md](./philosophy.md)                                         |

Rules files (`*.md`) are self-contained. Rationale files (`*.why.md`) are optional — read them only when an edge case or judgment call needs more context. Individual rationale sections are anchor-linked from the rules files, so you can pull one section (`typescript.why.md#types-vs-interfaces`) without loading the whole file.

## Not Covered

Code _formatting_ — indentation, line breaks, quote style, semicolons, trailing commas — is out of scope. Use Prettier, Biome, or ESLint for those. This repo focuses on higher-level concerns: naming, structure, patterns, principles, and practices.

## Conventions

- Every rule is tagged `non-negotiable`, `strong`, or `weak` so readers know which are hard, which are defaults, and which are consistency preferences. Definitions in [code-style/\_index.md](./code-style/_index.md).
