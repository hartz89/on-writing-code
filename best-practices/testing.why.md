# Testing — Rationale

Expanded rationale for opinionated rules in [testing.md](./testing.md). Section names mirror the rules file.

---

## Testing Trophy

**Rule:** Lean toward integration tests; unit-test pure logic; use e2e sparingly.

**Strength:** strong

Adapted from [Kent C. Dodds, _The Testing Trophy_](https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications) and [_Write tests. Not too many. Mostly integration._](https://kentcdodds.com/blog/write-tests).

### Integration tests give the best confidence-per-effort

Unit tests verify that a module does what it says on the tin. Integration tests verify that modules work together. Most bugs don't live inside a single function — they live at the seams. Integration tests catch those; unit tests don't.

### The pyramid was built for a different era

The classic pyramid (many units, fewer integrations, few e2e) assumed integration tests meant spinning up a database and a server. With MSW, Testing Library, and modern runners, integration tests are fast enough that the pyramid flattens into a trophy: static base, integration body, thin e2e tip.

### TypeScript is your base layer

Types rule out entire classes of bugs before a test runs — wrong shapes, missing fields, impossible states. Writing unit tests for what the compiler already proves is duplicated effort that rots.

### When to reach for each

- **Static (TS)**: types, `unknown`, discriminated unions. Always.
- **Unit**: pure functions, reducers, parsers, algorithms. Any time logic is isolable from I/O.
- **Integration**: components rendering with real dependencies, API endpoints end-to-end within the process. The workhorse.
- **E2E**: critical user flows against the deployed system. Slow, flaky; spend the budget where it actually matters.

---

## Coverage

**Rule:** Don't chase a coverage percentage. Use coverage to find untested paths.

**Strength:** strong

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

**Strength:** strong

### Implementation tests break on every refactor

A test that reaches into component state, private methods, or specific function calls will break the moment you refactor the internals — even if the behavior is identical. That's not a good test; that's a tripwire that fires on any structural change.

### Behavior tests survive refactors

A test that clicks a button and asserts the rendered result doesn't care whether the component uses Redux, Zustand, or local state. Rewrite the internals, keep the behavior, the test still passes. That's the point of a test suite — a safety net that lets you change the code with confidence.

### The rule of thumb

Ask: "if I refactor this to a different implementation that does the same thing, will this test still pass?" If no, the test is asserting on implementation. Move it up a level.

---

## Mock at the Furthest Reasonable Boundary

**Rule:** Mock as far toward the edge of your code as you reasonably can.

**Strength:** strong

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

**Strength:** strong

Adapted from [Kent C. Dodds, _Avoid Nesting When You're Testing_](https://kentcdodds.com/blog/avoid-nesting-when-youre-testing).

### Nested `describe`s hide state

A `beforeEach` five levels up sets up state the current test depends on. To understand what's being tested, you scroll up four blocks, track which setup applies at which level, then scroll back down. The test becomes a puzzle.

### Flat tests are readable in isolation

Each `test` contains (or explicitly calls) its own setup. Drop into any test and you can understand it without context. The duplication is real — three tests might repeat a line or two — and it's worth it. Readability compounds.

---

## Prefer Setup Functions

**Rule:** Use factory functions for shared setup; reserve `beforeEach` for fixture isolation.

**Strength:** strong

### `beforeEach` hides dependencies

A `beforeEach` block runs before every test in a `describe`, but you can't see it from the test itself. To understand what a test depends on, you have to scroll up — often across multiple levels of nesting — and trace every hook that applies. Factory functions make the dependency explicit: read the test top-down and see exactly what it sets up.

### Factories are composable

`makeFlower({ phase: 'open' })` accepts arguments. You can build variants inline, test-by-test, without a cascade of `describe` blocks each specializing setup. Composition scales; hook-layering doesn't.

### When `beforeEach` is appropriate

For _teardown_ and _fixture reset_: clearing mocks, closing connections, resetting fake timers. These are isolation concerns every test needs, and they don't carry meaningful state that should be visible to the test body.

---

## Test Structure

**Rule:** AAA (arrange, act, assert) is the default shape. Prefer fewer, longer tests over many tiny ones.

**Strength:** strong

Adapted from [Kent C. Dodds, _Write Fewer, Longer Tests_](https://kentcdodds.com/blog/write-fewer-longer-tests).

### One assertion per test is a unit-test convention that doesn't translate

The "one assert per test" rule comes from unit testing, where each test verifies one pure-function invariant. It doesn't fit integration tests that simulate user flows. An integration test that clicks through three steps and checks state at each step isn't three tests — it's one test exercising one journey.

### Smaller tests cost more to maintain

Splitting a flow into many tests means duplicating setup (or maintaining brittle shared setup), repeating assertions about intermediate state, and writing more test names. The multiplier hurts readability and runtime without a proportional gain in diagnosis quality.

### Split on unrelated behaviors, not on assertion count

If two branches of the same test are exercising different features, split them. If they're exercising different beats of the same feature, keep them together.
