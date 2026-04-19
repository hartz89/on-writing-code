# Refactoring

> Refactoring is rhythm, not an event — it's how you keep the cost of understanding from compounding.

Rationale for the opinionated sections lives in [refactoring.why.md](./refactoring.why.md). Strength levels (`non-negotiable` / `strong` / `weak`) are defined in [../code-style/\_index.md](../code-style/_index.md).

## Guiding Principles

- Refactoring is woven into normal work, not carved out of it.
- Make the change easy, then make the easy change.
- A refactor preserves observable behavior. If behavior changes, it's a feature change, not a refactor.
- Separate refactor commits from behavior commits so reviewers (and `git bisect`) can reason about each.

## Workflows of Refactoring

`strength: strong` · [rationale](./refactoring.why.md#workflows-of-refactoring)

Refactoring happens in distinct rhythms. Each earns its keep at a different moment:

- **TDD refactor** — the third beat of red / green / refactor. Structural cleanup while the tests are fresh.
- **Comprehension refactoring** — when reading unfamiliar code, apply your understanding back to the code: rename a cryptic variable, extract a clearer function. The next reader benefits.
- **Litter-pickup refactoring** — you notice something small and obviously wrong. Fix it if it's cheap; leave a note otherwise.
- **Preparatory refactoring** — before adding a feature, restructure so the feature fits cleanly. _Make the change easy, then make the easy change._
- **Planned refactoring** — scheduled, larger-scope cleanup. Rare; frequent need for planned work is a sign the other rhythms are being neglected.
- **Long-term refactoring** — large architectural shifts done incrementally alongside feature work, over weeks or months.

## Leave the Code Better Than You Found It

`strength: strong` · [rationale](./refactoring.why.md#leave-the-code-better-than-you-found-it)

The campsite rule: when you touch a file, improve it cheaply if you can. Rename a variable, clarify a comment, extract a helper. Small wins compound.

With pragmatic limits for larger codebases:

- "Cheap to improve" must be cheap for the _reviewer_ too, not just the author. A 20-line refactor tucked into a 200-line feature PR costs more than it saves.
- A drive-by rewrite of shared code can conflict with another team's in-flight work. Check before restructuring anything touched frequently.
- Under deadline pressure, file a tracking issue or leave a comment instead of refactoring in place. Visibility is almost as good as action.
- When in doubt, defer. The cost of a missed cleanup is smaller than the cost of a tangled review.

## Separate Refactors from Behavior Changes

`strength: strong`

- A refactor commit should preserve observable behavior. Tests pass before and after, without changing.
- Commit refactors separately from behavior changes. Each commit has one purpose; `git log` and `git bisect` stay useful.
- When a refactor is bigger than a commit, consider a dedicated PR rather than bundling it into a feature PR.
- Many small, well-named refactor commits beat one sprawling one: reviewable in 30 seconds each, revertable individually.
