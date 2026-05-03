# Testing — Rationale

Expanded rationale for opinionated rules in [testing.md](./testing.md). Section names mirror the rules file.

---

## Testing Trophy

**Rule:** Lean toward integration tests; unit-test pure logic; use e2e sparingly.

Adapted from [Kent C. Dodds, _The Testing Trophy_](https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications) and [_Write tests. Not too many. Mostly integration._](https://kentcdodds.com/blog/write-tests).

### Integration tests give the best confidence-per-effort

Unit tests verify that a module does what it says on the tin. Integration tests verify that modules work together. Most bugs don't live inside a single function — they live at the seams. Integration tests catch those; unit tests don't.

### The pyramid was built for a different era

The classic pyramid (many units, fewer integrations, few e2e) assumed integration tests meant spinning up a database and a server. With MSW, Testing Library, and modern runners, integration tests are fast enough that the pyramid flattens into a trophy: static base, integration body, thin e2e tip.

### TypeScript is your base layer

Types rule out entire classes of bugs before a test runs — wrong shapes, missing fields, impossible states. Writing unit tests for what the compiler already proves is duplicated effort that rots.

### When to reach for each

- **Static (TS)**: types, `unknown`, discriminated unions. This layer is always on — it's the compiler doing work before any test runs, not a tier you choose to invest in per feature.
- **Unit**: don't over-write these. Reach for them primarily around common utilities and pure-logic modules, where they earn their keep three ways:
  - as executable documentation for the utility's contract,
  - to cover edge cases that are unreasonable to provoke from an integration test,
  - to isolate the point of failure fast when a wave of integration tests goes red pointing at the same shared helper.
  - Otherwise, default to integration — integration tests already exercise most of the same code paths with better confidence-per-effort.
- **Integration**: components rendering with real dependencies, API endpoints end-to-end within the process. The workhorse.
- **E2E**: critical user flows against the deployed system. Slow, flaky; spend the budget where it actually matters.

---

## Coverage

**Rule:** Don't chase a coverage percentage. Use coverage to find untested paths.

Adapted from [Martin Fowler, _Test Coverage_](https://martinfowler.com/bliki/TestCoverage.html).

### Coverage is a lagging indicator

A high coverage number tells you that lines were executed during tests. It doesn't tell you that the tests were good, that the right behaviors were asserted, or that the code is correct. You can reach 100% coverage with a test suite that would miss most real bugs.

### Targeting coverage distorts incentives

Set a coverage target and you get tests that hit coverage — tests that exercise lines without meaningful assertions, tests that assert implementation details to force branches, tests that exist to pass the threshold. The number goes up; the confidence doesn't.

### How to use coverage

- Run it. Look at uncovered code. Decide: does this need a test, or is it unreachable / trivially correct?
- Treat a drop as a signal, not an alert. A drop means new code landed without tests — was that a deliberate call or an oversight?
- Don't gate PRs on a percentage. Gate on "are the important behaviors covered?"

---

## Test Behavior, Not Implementation

**Rule:** Assert what callers see. Don't assert on internal state.

### Implementation tests break on every refactor

A test that reaches into component state, private methods, or specific function calls will break the moment you refactor the internals — even if the behavior is identical. That's not a good test; that's a tripwire that fires on any structural change.

### Behavior tests survive refactors

A test that clicks a button and asserts the rendered result doesn't care whether the component uses Redux, Zustand, or local state. Rewrite the internals, keep the behavior, the test still passes. That's the point of a test suite — a safety net that lets you change the code with confidence.

### The rule of thumb

Ask: "if I refactor this to a different implementation that does the same thing, will this test still pass?" If no, the test is asserting on implementation. Move it up a level.

---

## Query as the User Perceives

**Rule:** Query the DOM the way a user perceives it, following [Testing Library's query priority](https://testing-library.com/docs/queries/about#priority).

Adapted from [Testing Library — About Queries](https://testing-library.com/docs/queries/about) and the [React Testing Library cheatsheet](https://testing-library.com/docs/react-testing-library/cheatsheet).

### Queries double as an accessibility contract

A test that finds a button by role and accessible name is also asserting that the button is reachable by a screen reader, a keyboard user, and an automated a11y audit. The query is doing two jobs at once. `getByTestId` does neither — it just finds an element. Choosing role-first queries gets a free a11y check for every interactive element you test.

### The priority order isn't arbitrary

Testing Library ranks queries by how closely they match real-user perception:

1. **Role + name** — what assistive tech and most users perceive ("the Save button").
2. **Label / placeholder / text** — what a sighted user reads on the page.
3. **Alt / title** — fallbacks for elements without text content.
4. **TestId** — invisible to every user; a developer-only hook.

Walk down the list until something matches. Reaching the bottom is a signal: either the markup lacks affordances real users need, or the test is asserting on plumbing.

### `*ByTestId` is an escape hatch

Some things genuinely have no role, label, or text — a non-interactive container, a chart canvas, a third-party widget you can't restructure. Test ids exist for those. If `data-testid` is sprinkled across normal buttons, headings, and form fields, the markup is the problem; fix it before reaching for the escape hatch.

### `getBy` / `queryBy` / `findBy` aren't interchangeable

Each variant encodes a different assertion intent:

- **`getBy*`** throws when the element is missing. Use it to assert presence — the throw becomes the failure with a useful message.
- **`queryBy*`** returns `null` when missing. Use it (and only it) to assert absence: `expect(queryBy...).not.toBeInTheDocument()`. Reaching for `getBy` here gives a confusing failure — the query throws before the matcher ever runs, so you see "unable to find element" instead of "expected absent."
- **`findBy*`** retries until the element appears or times out. Use it for anything that arrives asynchronously — after a fetch, a transition, an effect.

Picking the wrong variant works _most_ of the time, then fails with a misleading message at the worst moment. Pick by intent.

### `userEvent` over `fireEvent`

`fireEvent.click(button)` dispatches a single synthetic `click`. A real click also fires `pointerdown`, `mousedown`, `focus`, `pointerup`, `mouseup`, `click` — and produces no event on a `disabled` button. `userEvent.click` simulates the full sequence and respects disabled state, focusability, and form semantics. Tests written with `fireEvent` can pass against components real users can't actually use.

`fireEvent` is still the right tool for cases without a user-facing analogue: firing a `change` on a hidden input during a non-standard flow, dispatching a custom event. Reach for it deliberately, not by default.

---

## Mock at the Furthest Reasonable Boundary

**Rule:** Mock as far toward the edge of your code as you reasonably can.

Adapted from [Kent C. Dodds, _Stop Mocking Fetch_](https://kentcdodds.com/blog/stop-mocking-fetch).

### Function-level mocks lie to you

`jest.mock('./api')` replaces your API module with a stub. Every downstream caller sees the stub. You're now testing "does my code call my mock correctly?" — not "does my code work?" The mock can drift from reality without any test noticing.

### Boundary mocks stay honest

Mock HTTP at the network layer (MSW). Your real API module runs. Your real fetch logic runs. Your real parsing runs. The only thing stubbed is the server response — which is the part that actually isn't running in the test. The test exercises everything that's yours.

### "Reasonable" matters

Some third-party SDKs don't expose a clean network boundary — they wrap HTTP internally, or use transports (gRPC, WebSockets) that don't fit easily with `fetch` interception. In those cases, stubbing at the SDK client is the furthest you can reasonably go. Don't force the test through a technically-correct layer that nothing else touches; stop at the last layer that maps to something real.

### Where the boundary actually is

- **Network (HTTP)**: MSW or equivalent. Intercepts `fetch` / XHR.
- **Database**: testcontainers, in-memory instance, or a dedicated test schema. Not a mock of your ORM.
- **Third-party SDK**: stub the SDK client at construction; let your adapter code run. Go further (intercept its HTTP) if it uses plain `fetch` underneath.
- **Filesystem / clock / randomness**: inject as dependencies; don't mock globally.

---

## Avoid Nesting

**Rule:** Prefer flat `test` blocks. Minimize `describe` depth.

Adapted from [Kent C. Dodds, _Avoid Nesting When You're Testing_](https://kentcdodds.com/blog/avoid-nesting-when-youre-testing).

### Nested `describe`s hide state

A `beforeEach` five levels up sets up state the current test depends on. To understand what's being tested, you scroll up four blocks, track which setup applies at which level, then scroll back down. The test becomes a puzzle.

### Flat tests are readable in isolation

Each `test` contains (or explicitly calls) its own setup. Drop into any test and you can understand it without context. The duplication is real — three tests might repeat a line or two — and it's worth it. Readability compounds.

---

## Test Structure

**Rule:** AAA (arrange, act, assert) is the default shape. Prefer fewer, longer tests over many tiny ones.

Adapted from [Kent C. Dodds, _Write Fewer, Longer Tests_](https://kentcdodds.com/blog/write-fewer-longer-tests).

### One assertion per test is a unit-test convention that doesn't translate

The "one assert per test" rule comes from unit testing, where each test verifies one pure-function invariant. It doesn't fit integration tests that simulate user flows. An integration test that clicks through three steps and checks state at each step isn't three tests — it's one test exercising one journey.

### Smaller tests cost more to maintain

Splitting a flow into many tests means duplicating setup (or maintaining brittle shared setup), repeating assertions about intermediate state, and writing more test names. The multiplier hurts readability and runtime without a proportional gain in diagnosis quality.

### Split on unrelated behaviors, not on assertion count

If two branches of the same test are exercising different features, split them. If they're exercising different beats of the same feature, keep them together.

---

## Setup Functions

**Rule:** Start inline; extract a setup function when inlining gets repetitive. Reserve `beforeEach` for fixture isolation.

### `beforeEach` hides dependencies

A `beforeEach` block runs before every test in a `describe`, but you can't see it from the test itself. To understand what a test depends on, you have to scroll up — often across multiple levels of nesting — and trace every hook that applies. A setup function called explicitly makes the dependency visible: read the test top-down and see exactly what it sets up.

### Setup functions are composable

`makeFlower({ phase: 'open' })` accepts arguments. You can build variants inline, test-by-test, without a cascade of `describe` blocks each specializing setup. Composition scales; hook-layering doesn't.

### Don't extract prematurely, don't resist at scale

A two-test file probably doesn't need a setup helper — inline construction is clearer. But as a suite grows, the same construction repeats and starts drowning the intent of each test. That's the moment to extract. The goal isn't "setup functions everywhere" or "always inline"; it's keeping the test body about the behavior under test, with construction out of the way.

"Setup function" is the term to reach for — "factory" fits when you're producing domain objects (`makeFlower`), but plenty of useful setup isn't object-construction (mounting with providers, seeding a test DB, building a request context). Call it what it is.

### When `beforeEach` is appropriate

For _teardown_ and _fixture reset_: clearing mocks, closing connections, resetting fake timers. These are isolation concerns every test needs, and they don't carry meaningful state that should be visible to the test body.
