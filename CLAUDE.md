# CLAUDE.md

Notes for agentic sessions working in this repo. See [README.md](./README.md) for the human-facing overview.

## What this repo is

A collection of markdown style guides and development principles. There is no build, no test suite, no runtime — every change lands in a markdown file.

Code _formatting_ (indentation, quotes, semicolons, trailing commas) is deliberately out of scope — delegated to Prettier / Biome / ESLint. Don't add formatting rules here; if a contributor proposes one, redirect them to their linter config.

## Structure

```
README.md                        # repo overview
code-gardening.md                # human-facing manifesto (not part of the LLM reading path)
philosophy.md                    # language-agnostic principles
inspiration.md                   # attribution / further reading
code-style/
  _index.md                      # pattern docs + strength-level definitions + guide status
  typescript.md                  # rules
  typescript.why.md              # rationale
  react.md                       # rules
  react.why.md                   # rationale
  vanilla-extract-css.md         # rules
  vanilla-extract-css.why.md     # rationale
best-practices/
  _index.md                      # guide status
  refactoring.md                 # rules
  refactoring.why.md             # rationale
  testing.md                     # rules
  testing.why.md                 # rationale
```

## The rule / rationale split

Every `<lang>.md` rules file has an optional `<lang>.why.md` sibling for the non-obvious reasoning. This exists because:

- Rules files are the primary artifact an LLM will consume — keeping them terse saves context.
- Rationale is valuable but cheap to skip when not needed. Split it out.

Add a why section **only when the rationale is non-obvious, contested, or worth defending**. Self-explanatory rules (naming, key-from-data, handler prefix) don't need one. Over-documentation is a real failure mode here.

Section names in the why file mirror section names in the rules file so anchor links work (`./typescript.why.md#types-vs-interfaces`).

## Strength levels

Every rule section is tagged with one of:

- `non-negotiable` — hard rule. Violating requires a local justifying comment.
- `strong` — default with rare exceptions. Deviation defensible on review.
- `weak` — consistency preference. Either choice is defensible; pick one.

Canonical definitions: [code-style/\_index.md](./code-style/_index.md). Canonical example: [code-style/typescript.md](./code-style/typescript.md).

## Voice and tone

- Inspired by _Clean Code_ (naming, small units, SRP) and Kent C. Dodds (simplest thing that works; avoid hasty abstractions).
- Terse. Rules as bullets. Examples as code blocks. No marketing prose.
- Show, don't tell — examples first, rules annotating them.
- No trailing summaries, no restating what the diff did.

## When adding a new guide or rule

1. Write the rule as a concise bullet or sentence. Add a strength tag.
2. Add a code example if the rule is structural.
3. Only add a why section if the reasoning is non-obvious.
4. If you add a why section, link to it from the rules file: `[rationale](./<lang>.why.md#section-anchor)`.
5. Update [code-style/\_index.md](./code-style/_index.md) status table if creating a new guide.

## Citation convention

Two patterns for linking to source material:

- **"Adapted from [...]" at the top of a why section** (under the `**Strength:**` line) when the whole rationale is directly derived from a named source. See [best-practices/testing.why.md](./best-practices/testing.why.md) for examples.
- **Inline link within prose** when a specific sentence references a specific source. Use this when the source strengthens one claim rather than anchoring the whole section.

Don't add citations where there's no specific source — community-derived synthesis stands on its own. Forcing "further reading" onto every section clutters the files without adding signal.

[inspiration.md](./inspiration.md) at the repo root remains the lean browsable index of shaping influences (books, foundational essays, practitioners). It complements inline citations; it doesn't duplicate them.

## When not to add rationale

- The rule restates a language-level behavior (e.g., "keys should be unique").
- The reason is "convention" and no reasonable reader would contest it.
- You're explaining _what_ the rule does instead of _why_ it exists.

If the why is already obvious from the rule, skip it.

## Evolving the guide

This repo grows conversationally — the user often adds opinions stream-of-consciousness and may revisit ground that's partially covered. Bring judgment to the rhythm:

- **Push back when something is already covered.** If new feedback restates an existing rule or rationale, say so and point to where. Don't silently add duplicative content; the user has explicitly asked for this pushback.
- **Refactor freely.** Merge, split, rename, or rearrange sections when the shape of the content calls for it — don't treat the existing layout as load-bearing. Call out the restructuring in your response so it can be vetoed if wrong.
- **Don't over-document.** Obvious, self-explanatory, uncontested rules don't need a rationale section or an inline citation. If you catch yourself writing "this is because… well, it's obvious," drop the prose.
- **Ask on design questions; proceed on minor ones.** A genuine decision ("strong or weak?", "where does this section live?") is worth a single clarifying exchange. A wording tweak isn't.

## Agentic defaults

- Prefer editing existing files over creating new ones.
- Never write a trailing "summary" paragraph that restates changes. The diff speaks.
- When a decision is ambiguous (e.g., should this be `strong` or `weak`?), ask rather than guess.
- Match the voice of neighboring content. If in doubt, re-read `code-style/typescript.md` — that's the canonical example.

## Markdown conventions

- 2-space indent, LF line endings, UTF-8, max 100 cols (see `.editorconfig`).
- Headings in sentence case except for proper nouns and code identifiers.
- Fenced code blocks with a language tag (`ts`, `tsx`, `md`).
- Internal links are relative paths: `./typescript.why.md#...`.
