# Philosophy

Language-agnostic principles that shape the style guides in this repo. Not rules with strength tags — defaults that good code tends to arrive at. When a code-style guide points to "why," it usually lands here.

## YAGNI — You Aren't Gonna Need It

Don't build what you don't need. Don't add configuration for a case that doesn't exist. Don't carve out an abstraction for a second (or third) caller that hasn't shown up. Every line is a liability — it has to be read, maintained, and eventually changed or deleted.

Write for what's real, not what might be. If the second case materializes, you'll be in a better position to design for it than you are today — you'll have two concrete examples to compare.

## AHA — Avoid Hasty Abstractions

Duplicate before you abstract. Two copies of similar code is cheaper than one wrong abstraction. Three is where a pattern starts to emerge, and even then only if the three cases share true structure — not surface similarity.

The failure mode is the abstraction that fits two cases perfectly and fights the third. You end up with conditional flags, optional parameters, and "just for this case" branches — the abstraction costs more than the duplication it replaced.

## Don't Solve Problems You Don't Have

A specific target within YAGNI: invented problems. _"What if we need to swap databases?" "What if we scale 10x?" "What if we go multi-region?"_ If none of those are on the horizon, the abstraction you build for them is speculation — and most speculation is wrong.

Solve the problem in front of you. Leave the structure clean enough to change later. "Later" is the right time to design for scale you don't yet have.

## Prefer Readability over Cleverness

Code is read far more often than it's written. A clever one-liner that takes two minutes to parse costs everyone who reads it. A straightforward ten-liner that takes fifteen seconds costs no one.

Clever is a trap. It rewards the author and punishes the reader. Write the version the next person can skim — which, half the time, is you in six months.

## Don't Pre-Optimize

Performance work is expensive and specific. It trades readability, flexibility, or both for a speedup that may or may not matter. Without measurement, you can't tell whether you've won or lost.

Default: write the clear version. Measure. If something is actually slow, _then_ optimize — and only the part that's actually slow. "Fast by default" usually means "unreadable by default."

## Optimize for Change

Code is edited far more often than it's written. The cost of a line isn't its length — it's the cost of the next person who has to modify it. Between two forms that cost the same today, pick the one that costs less to change tomorrow.

In practice: curly braces around function bodies even when a one-liner works (the next change usually adds a second line). Named constants over inline values (the second caller is inevitable). Additive changes — a new case in a union, a new field on a record — over structural rewrites that force a cascade of edits.

This isn't the same as designing for every possible future — that's speculation, and YAGNI rules. It's choosing the form that costs nothing now and saves effort later, every time you can.

## Colocate Things That Change Together

Adapted from [Kent C. Dodds, _Colocation_](https://kentcdodds.com/blog/colocation).

Put code near the other code that changes with it. A component's styles, tests, and types belong next to the component — not in parallel directory trees. When you're editing, everything relevant is one file-tree expansion away.

Organizing by "kind" (all styles in `styles/`, all tests in `tests/`) optimizes for "show me every style" at the cost of "show me everything about this feature." The second query is the common one. Organize for it.

Colocation scales down too: a helper used only by one module belongs in that module, not in a shared `utils/`. Promote to a shared location only when it has a real second caller.

When you do promote, put the shared code in a `common/` (or `shared/`) directory at the closest scope that makes sense. A helper used by three files inside `features/checkout/` belongs in `features/checkout/common/`, not a top-level `utils/`. Reserve top-level shared directories for code genuinely used across features — not code that's merely reusable in principle.

## Make Illegal States Unrepresentable

Model your data so invalid combinations can't be constructed. A discriminated union that splits variants, a state machine that can't enter a bad transition, a constructor that validates on creation — each one removes a whole class of bugs from the "possible" set.

Validation at boundaries is necessary. Validation inside your code is a sign you're carrying impossible states around and paying for their possibility everywhere you touch them.

## Contain Side Effects

Adapted from [Gary Bernhardt, _Boundaries_](https://www.destroyallsoftware.com/talks/boundaries) (the "Functional Core, Imperative Shell" pattern).

Prefer pure functions — same inputs, same outputs, no hidden state, nothing reaching outside. Pure code is easier to read, test, and change: you only have to understand the function, not the world it interacts with.

Side effects are unavoidable at some point — I/O, network, `useEffect`, writes to disk, logging. The goal isn't to eliminate them but to _contain_ them. Push impurity to the edges ("imperative shell"); keep the middle ("functional core") pure. A thin effectful wrapper around a fat core of pure logic is far easier to test and change than effects scattered throughout.

When an effect does need to live in the core — a `useEffect`, a cache write, a log call — name it clearly and isolate its dependencies. Don't hide side effects inside functions that look pure.

## Small Units, Single Responsibility

Small functions are easier to read, test, name, and reuse. A function that does one thing has an obvious name; a function that does three is a runtime collection of branches.

"Small" isn't about line count — it's about a single purpose. A 20-line function with one responsibility is smaller than a 5-line function that orchestrates three. Extract when the name stops describing the whole thing.

## Naming Things

Names are documentation that ships with the code.

**Prefer verbose over brief.** `fetchActiveUsersForTenant` beats `fetch`. Production code is minified before it ships, and autocomplete eliminates the typing cost. The clarity earned from every future reader compounds; the extra characters cost nothing.

**Make filenames unique across the project.** "Find by filename" is one of the most-used navigation primitives in any editor, and it only works when names don't collide. Prefer `FlowerBed.tsx` over `index.tsx`; `userService.ts` over `service.ts`. Generic names force a second step (path context, tab titles, "which one?") every time.

**Directory names can be shorter and reused.** Directories are browsed in context — the enclosing path already disambiguates. `utils/`, `hooks/`, `components/` can recur across the tree without confusion. Save uniqueness for leaf names where it earns its keep.

## Code Says What; Comments Say Why

Well-named identifiers tell the reader what the code does. The code is the source of truth for behavior. Comments that restate the code add noise — and they rot, because code changes and comments don't.

The comments that earn their keep explain _why_: a non-obvious constraint, a workaround for a specific bug, a decision that surprised you when you made it. If removing the comment wouldn't confuse a future reader, don't write it.
