# Inspiration

Few of the opinions in this repo are original. This page collects the source material — books, essays, and practitioners whose thinking shaped these guides. Not exhaustive; a starting library.

## Books

- **_Clean Code_**, Robert C. Martin. Meaningful names, small functions, single responsibility, comments as a failure to express yourself in code.
- **_The Pragmatic Programmer_**, Hunt & Thomas. Tracer bullets, broken windows, DRY, orthogonality — the practitioner's handbook.
- **_On Writing_**, Stephen King. Nothing at all to do with code, inspired the repository name.

## Martin Fowler

- [**Test Coverage**](https://martinfowler.com/bliki/TestCoverage.html) — coverage is a lagging indicator; don't chase a percentage, use it to find untested paths.
- [**Workflows of Refactoring**](https://martinfowler.com/articles/workflowsOfRefactoring/) — the rhythms of applying refactoring: TDD, litter-pickup, comprehension, preparatory, planned, long-term.

## Gary Bernhardt

- [**Boundaries**](https://www.destroyallsoftware.com/talks/boundaries) — the "Functional Core, Imperative Shell" pattern. Push side effects to the edges; keep the core pure.

## Kent C. Dodds

- [**AHA Programming**](https://kentcdodds.com/blog/aha-programming) — _Avoid Hasty Abstractions._ Duplicate before you abstract; the wrong abstraction costs more than duplication.
- [**Colocation**](https://kentcdodds.com/blog/colocation) — things that change together should live together. Organize by feature, not by kind.
- [**The Testing Trophy**](https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications) — static → unit → integration → e2e. Integration tests give the best confidence-per-effort.
- [**Write tests. Not too many. Mostly integration.**](https://kentcdodds.com/blog/write-tests) — the Guillermo Rauch quote, expanded into a testing philosophy.
- [**Write Fewer, Longer Tests**](https://kentcdodds.com/blog/write-fewer-longer-tests) — the "one assertion per test" rule is a unit-test convention that breaks down for integration tests.
- [**Avoid Nesting When You're Testing**](https://kentcdodds.com/blog/avoid-nesting-when-youre-testing) — flat `it` blocks over deeply nested `describe`s; independent, readable tests.
- [**Stop Mocking Fetch**](https://kentcdodds.com/blog/stop-mocking-fetch) — mock at the network layer (MSW), not the function. Closer to what ships.
