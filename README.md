# on writing code

A collection of opinionated style guides and development principles, compiled by Ryan Hartzfeld.

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

See [\_index.md](./_index.md) for the task-to-file index.

## Not Covered

Code _formatting_ — indentation, line breaks, quote style, semicolons, trailing commas — is out of scope. Use Prettier, Biome, or ESLint for those. This repo focuses on higher-level concerns: naming, structure, patterns, principles, and practices.

## Conventions

- Every rule is tagged `non-negotiable`, `strong`, `moderate`, or `trivial` so readers know which are hard, which are defaults, which are good-but-flexible, and which are consistency preferences. Definitions in [code-style/\_index.md](./code-style/_index.md).
