# Refactoring

Rationale: [refactoring.why.md](./refactoring.why.md). Strength levels: [../code-style/\_index.md](../code-style/_index.md).

## Workflows of Refactoring

`strength: strong` · [rationale](./refactoring.why.md#workflows-of-refactoring)

Distinct rhythms, each earning its keep at a different moment:

- **TDD refactor** — the third beat of red / green / refactor. Structural cleanup while the tests are fresh.
- **Comprehension refactoring** — when reading unfamiliar code, apply your understanding back to the code: rename a cryptic variable, extract a clearer function. The next reader benefits.
- **Litter-pickup refactoring** — you notice something small and obviously wrong. Fix it if it's cheap; leave a note otherwise.
- **Preparatory refactoring** — before adding a feature, restructure so the feature fits cleanly. _Make the change easy, then make the easy change._
- **Planned refactoring** — scheduled, larger-scope cleanup. Rare; frequent need for planned work is a sign the other rhythms are being neglected.
- **Long-term refactoring** — large architectural shifts done incrementally alongside feature work, over weeks or months.

## Leave the Code Better Than You Found It

`strength: strong` · [rationale](./refactoring.why.md#leave-the-code-better-than-you-found-it)

- Campsite rule: when you touch a file, improve it cheaply if you can (rename, clarify, extract).
- Keep the cleanup cheap for the _reviewer_ too — extract larger refactors into their own PR.
- Don't drive-by-rewrite shared, high-churn code; check for in-flight work first.
- Can't do it now? File a tracking issue or leave a comment. Visibility is almost as good as action.

## Separate Refactors from Behavior Changes

`strength: strong`

- A refactor commit should preserve observable behavior. Tests pass before and after, without changing.
- Commit refactors separately from behavior changes. Each commit has one purpose; `git log` and `git bisect` stay useful.
- When a refactor is bigger than a commit, consider a dedicated PR rather than bundling it into a feature PR.
- Many small, well-named refactor commits beat one sprawling one: reviewable in 30 seconds each, revertable individually.
