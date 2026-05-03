# CLAUDE.md

Notes for agentic sessions working in this repo. For the task-to-file index, see [\_index.md](./_index.md). For the human-facing overview, see [README.md](./README.md).

## What this repo is

A collection of markdown style guides and development principles. There is no build, no test suite, no runtime — every change lands in a markdown file.

Code _formatting_ (indentation, quotes, semicolons, trailing commas) is deliberately out of scope — delegated to Prettier / Biome / ESLint. Don't add formatting rules here; if a contributor proposes one, redirect them to their linter config.

## How the user works

- Feedback may arrive stream-of-consciousness, not in a pre-organized outline.
- **Pushback is welcomed, within reason.** If feedback duplicates existing content or conflicts with a rule in the repo, say so and point to where. Silent agreement is the wrong move.

## Structure

```
README.md                        # human-facing repo overview
_index.md                        # LLM entrypoint: task-to-file index
code-gardening.md                # human-facing manifesto (not part of the LLM reading path)
philosophy.md                    # language-agnostic principles
inspiration.md                   # attribution / further reading
code-style/
  _index.md                      # strength-level definitions + guide status
  typescript.md                  # rules
  typescript.why.md              # rationale
  react.md                       # rules
  react.why.md                   # rationale
  css.md                         # general CSS rules (tool-agnostic)
  css.why.md                     # rationale
  vanilla-extract-css.md         # rules
  vanilla-extract-css.why.md     # rationale
best-practices/
  _index.md                      # guide status
  architecting-features.md       # rules
  architecting-features.why.md   # rationale
  refactoring.md                 # rules
  refactoring.why.md             # rationale
  testing.md                     # rules
  testing.why.md                 # rationale
```

## Strength levels

Every rule section is tagged `non-negotiable` / `strong` / `moderate` / `trivial`. Canonical definitions: [code-style/\_index.md](./code-style/_index.md). Canonical example: [code-style/typescript.md](./code-style/typescript.md).

## The rule / rationale split

Rules files are the hot path — pulled into context when an LLM writes code. `.why` files are cold — read only when an edge case, pushback, or design decision needs the reasoning. Keep the hot path thin: prose in a rules file is context every consumer pays for, forever; prose in a `.why` file is opt-in.

### What belongs in each file

**Rules file (`<lang>.md`):**

- A strength tag.
- The rule, as a bullet or a short sentence. No paragraphs.
- A code example when the rule is structural.
- An anchor link to the `.why` file when rationale exists.

That's it. If you find yourself writing "because…", "the argument is…", "this matters when…", "consider that…" — stop. That sentence belongs in the `.why` file.

**`.why` file (`<lang>.why.md`):**

- The rule restated in one sentence.
- The reasoning — history, tradeoffs, counter-arguments, examples of what goes wrong without the rule. Paragraphs are fine here.

### When rationale is worth writing

Add a rationale section only when the reasoning is non-obvious, contested, or worth defending. Skip it when:

- The rule restates a language-level behavior (e.g., "keys should be unique").
- The reason is "convention" and no reasonable reader would contest it.
- You're explaining _what_ the rule does instead of _why_ it exists.

Over-documentation is a real failure mode here.

### Drift signals

A rules file that's grown prose paragraphs, multi-sentence bullets, or parenthetical justifications has drifted. When you notice it — in a diff you're writing, or a file you're editing — pull the prose into the `.why` file and leave the rules file with bullets and examples.

### Structural parity

Treat the rules file and its `.why.md` sibling as two views of one document. Section names and ordering must mirror — anchor links depend on it.

When you rename, add, or remove a section:

1. Do it in both files in the same commit.
2. Update every anchor link pointing at the old section. Search the repo first — links can come from anywhere.
3. If removing, remove the rule _and_ the rationale, not just one.

## Evolving the guide

### Adding a rule within an existing guide

1. Write the rule as a concise bullet; add a strength tag.
2. Add a code example if the rule is structural.
3. Decide on rationale per ["When rationale is worth writing"](#when-rationale-is-worth-writing); if yes, add the mirroring section in the `.why` file and link to it.

### Adding, renaming, or removing a whole guide

A stale index sends readers to files that don't exist. Update all of these in the same commit:

- `code-style/_index.md` or `best-practices/_index.md` — status table.
- [README.md](./README.md) — "For LLM Consumers" and Structure sections.
- The Structure tree in this file.
- Any cross-guide pointers (e.g., `vanilla-extract-css.md` links to `css.md`).

### Restructuring an existing guide

Refactor freely — don't treat the existing layout as load-bearing. Call out the restructuring so it can be vetoed. Apply structural parity above: restructuring a rules file means restructuring the `.why` file too.

## Citation convention

- **"Adapted from [...]" at the top of a rationale section** (under `**Strength:**`) when the whole rationale derives from one named source. See [best-practices/testing.why.md](./best-practices/testing.why.md).
- **Inline link** when a specific sentence references a specific source.

Don't force citations where there's no specific source — community-derived synthesis stands on its own. [inspiration.md](./inspiration.md) is the browsable index of shaping influences; it complements inline citations, doesn't duplicate them.

## Writing style

- Terse. Rules as bullets, examples as code blocks, no marketing prose.
- Show, don't tell — examples first, rules annotating them.
- Match neighboring voice. Canonical example: `code-style/typescript.md`.
- Prefer editing existing files over creating new ones. No trailing "summary" paragraph — the diff speaks.
- Sentence-case headings (except proper nouns and code identifiers). Fenced code blocks with a language tag. Relative internal links (`./typescript.why.md#...`). Other formatting: see `.editorconfig`.
